# Entity & Repository Pattern Reference

Reference for dept44 JPA entities and Spring Data repositories. Look at existing code in `integration/db/` for exact style.

## Entity Class

- Package: `integration/db/model/`
- Class name: `{Name}Entity`
- Same `create()`/`with*()`/getter/setter pattern as API POJOs, plus JPA annotations
- `@Entity`, `@Table(name = "snake_case_table_name")`
- `@Id` with `@UuidGenerator` — `String id` (import `org.hibernate.annotations.UuidGenerator`)
- `@Column(name = "snake_case")` with `nullable` and `length` attributes
- `@TimeZoneStorage(NORMALIZE)` on `OffsetDateTime` fields
- `@PrePersist` / `@PreUpdate` for auto-setting timestamps
- Relationships: `@ManyToOne`, `@OneToMany(cascade = ALL, orphanRemoval = true, fetch = EAGER)`
- Manual `equals()`, `hashCode()`, `toString()` — use concatenation style for `toString()`:
  ```java
  return "EntityName{" +
      "id='" + id + '\'' +
      ", field=" + field +
      '}';
  ```
- Flyway migration: UUID columns use `VARCHAR(255) NOT NULL PRIMARY KEY` (no auto-increment)

## Entity Test Pattern

Same three tests as POJO, plus checks for `new Entity()`:

```java
@Test
void testNoDirtOnCreatedBean() {
    assertThat(ReminderEntity.create()).hasAllNullFieldsOrProperties();
    assertThat(new ReminderEntity()).hasAllNullFieldsOrProperties();
}
```

Since `@UuidGenerator` uses `String id` (default `null`), no fields need to be excluded.

## Repository Interface

- Package: `integration/db/`
- Extends `JpaRepository<{Entity}, {IdType}>` (and `JpaSpecificationExecutor` if filtering needed)
- **`@CircuitBreaker(name = "{repositoryName}")` is mandatory**
- Derived query methods: `findByIdAndNamespaceAndMunicipalityId`, `existsBy...`
- `@Lock(LockModeType.PESSIMISTIC_WRITE)` for row-level locking — method uses `WithLocking` convention
- No `@Repository` annotation needed
- Complex queries: `@Query` with JPQL

```java
@CircuitBreaker(name = "errandsRepository")
public interface ErrandsRepository extends JpaRepository<ErrandEntity, String>,
    JpaSpecificationExecutor<ErrandEntity> {

    boolean existsByIdAndNamespaceAndMunicipalityId(String id, String namespace, String municipalityId);

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    boolean existsWithLockingByIdAndNamespaceAndMunicipalityId(String id, String namespace, String municipalityId);

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    Optional<ErrandEntity> findWithLockingById(String id);

    Optional<ErrandEntity> findByIdAndNamespaceAndMunicipalityId(String id, String namespace, String municipalityId);
}
```
