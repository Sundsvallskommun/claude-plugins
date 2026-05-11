# Scaffold New Entity

Create a JPA entity with repository and Flyway migration following dept44 patterns. Use existing entities in this repo as reference.

## Arguments
- `$ARGUMENTS` — the entity name and fields (e.g. "Reminder with message:String, remindAt:OffsetDateTime, errandId:String")

## What to create

Examine existing `integration/db/model/` to match the exact style.

### 1. Entity (`integration/db/model/`)
- `{Name}Entity.java`
- `@Entity`, `@Table(name = "snake_case")`
- `@Id` + `@UuidGenerator` — `String id` (import `org.hibernate.annotations.UuidGenerator`)
- `@Column(name = "snake_case")` with `nullable`, `length` attributes
- `@TimeZoneStorage(NORMALIZE)` on any `OffsetDateTime` fields
- `create()` factory method, `with*()` fluent setters, standard getters/setters
- Manual `equals()`, `hashCode()`, `toString()` — use concatenation style for toString
- Flyway: UUID columns use `VARCHAR(255) NOT NULL PRIMARY KEY` (no auto-increment)
- `@PrePersist` / `@PreUpdate` for auto-setting `created`/`modified` timestamps if applicable
- Relationships (`@ManyToOne`, `@OneToMany`) as specified

### 2. Repository (`integration/db/`)
- `{Name}Repository.java` — interface extending `JpaRepository<{Name}Entity, {IdType}>`
- **`@CircuitBreaker(name = "{name}Repository")` is mandatory**
- Add `JpaSpecificationExecutor` if filtering/search will be needed
- Add derived query methods for common lookups (e.g. `findByIdAndNamespaceAndMunicipalityId`)

### 3. Flyway migration (`src/main/resources/db/migration/`)
- Determine next version number from existing migrations
- `V{version}__{description}.sql` (e.g. `V2_3__create_reminder_table.sql`)
- Use `snake_case` for table and column names
- Include proper types, NOT NULL constraints, foreign keys
- MariaDB syntax

### 4. Tests
- `{Name}EntityTest.java` — three standard bean tests:
  - `testBean()` — BeanMatchers
  - `testBuilderMethods()` — all `with*()` methods, `hasNoNullFieldsOrProperties`
  - `testNoDirtOnCreatedBean()` — both `create()` and `new {Name}Entity()`, `hasAllNullFieldsOrProperties()`

### 5. Test data
- If integration tests exist, add seed data to `src/integration-test/resources/db/scripts/testdata-it.sql`
- Add truncation to `truncate.sql`

### 6. Dependencies (`pom.xml`)
- Ensure `dept44-starter-jpa` is present as a dependency
- Ensure `dept44-starter-jpa-test` is present with `<scope>test</scope>`

## Important
- Look at existing entities in this repo for exact patterns, column types, and relationship styles
- Use `final` on all parameters
- No Lombok, no wildcard imports
- Ensure Flyway is enabled in config if this is the first migration
