# Scaffold New CRUD Endpoint (Database-backed)

Create a complete CRUD endpoint backed by a local database following dept44 patterns. Use existing code in this repo as reference.

## Arguments
- `$ARGUMENTS` — the entity/domain name and optional fields (e.g. "Reminder" or "Reminder with message:String, remindAt:OffsetDateTime, active:boolean")

## What to create

Examine the existing package structure to determine the base package and service name. Look at existing code for exact import styles, formatting, and naming.

### 1. API Model (`api/model/`)
- `{Name}.java` — POJO with `create()`, `with*()`, getters/setters, `equals`/`hashCode`/`toString`
- `@Schema` annotations on class and fields
- Jakarta validation annotations (`@NotBlank`, `@Null(groups = OnCreate.class)`, etc.)
- No Lombok

### 2. Entity (`integration/db/model/`)
- `{Name}Entity.java` — `@Entity`, `@Table(name = "snake_case")`
- `@Id` + `@UuidGenerator` — `String id`
- `@Column(name = "snake_case")` with `nullable`, `length`
- `@TimeZoneStorage(NORMALIZE)` on `OffsetDateTime` fields
- Same `create()`/`with*()` pattern

### 3. Repository (`integration/db/`)
- `{Name}Repository.java` — extends `JpaRepository<{Name}Entity, {IdType}>`
- **`@CircuitBreaker(name = "{name}Repository")` is mandatory**
- Derived query methods: `findByIdAndNamespaceAndMunicipalityId`, `existsBy...`, etc.

### 4. Mapper (`service/mapper/`)
- `{Name}Mapper.java` — private constructor, all static methods
- `to{Name}(entity)` — entity → API model
- `to{Name}Entity(model)` — API model → entity
- `to{Name}List(entities)` — list mapping
- `updateEntity(entity, model)` — partial update
- Null-safe with `ofNullable()`

### 5. Service (`service/`)
- `{Name}Service.java` — `@Service`, constructor injection
- CRUD methods: `get`, `getAll`/`find`, `create`, `update`, `delete`
- Injects `{Name}Repository` directly (no Integration wrapper needed for local DB)
- `Problem.valueOf(NOT_FOUND, ...)` when entity not found
- Delegates all mapping to `{Name}Mapper`
- `@Transactional` on multi-write operations

### 6. Resource (`api/`)
- `{Name}Resource.java` — package-private, `@RestController`, `@Validated`
- `@RequestMapping("/{municipalityId}/...")` with `@ValidMunicipalityId`
- CRUD endpoints:
  - `POST` → `created(location).build()` with Location header
  - `GET /{id}` → `ok(result)`
  - `GET` (list) → `ok(results)`
  - `PUT /{id}` → `ok(result)` or `noContent().build()`
  - `DELETE /{id}` → `noContent().build()`
- Full `@Operation` + `@ApiResponse` for each endpoint

### 7. Flyway migration (`src/main/resources/db/migration/`)
- Next version number based on existing migrations
- `V{version}__{description}.sql`
- MariaDB syntax, `snake_case` names

### 8. Tests
Create all test classes following CLAUDE.md patterns:
- `{Name}Test.java` — POJO: testBean, testBuilderMethods, testNoDirtOnCreatedBean
- `{Name}EntityTest.java` — Entity: same three tests, with `hasAllNullFieldsOrPropertiesExcept("id")`
- `{Name}MapperTest.java` — `@ParameterizedTest` with `@MethodSource`, null-safety tests
- `{Name}ServiceTest.java` — `@ExtendWith(MockitoExtension.class)`, `@Mock` repo, `@InjectMocks` service
- `{Name}ResourceTest.java` — happy path, `@SpringBootTest`, `@AutoConfigureWebTestClient`, `WebTestClient`, `@MockitoBean`
- `{Name}ResourceFailureTest.java` — validation failures, `@AutoConfigureWebTestClient`, use URL-safe invalid values (e.g. `"bad-municipality-id"`, `"not-a-valid-uuid"`) — do NOT use `#` in constants since it is a URL fragment delimiter
- If the new service injects existing classes, verify those classes have sufficient test coverage (85% line + branch) — add tests for them if missing

### 9. OpenAPI spec (`src/integration-test/resources/api/openapi.yaml`)
- Add new paths matching all Resource endpoints
- Add new schema components for the API model
- Note: `@NotBlank` generates `minLength: 1` in the generated spec — include it
- Run `OpenApiSpecificationIT` via `mvn verify -Dit.test=OpenApiSpecificationIT` to verify the spec matches the runtime-generated one

### 10. Schema verification
- Check if the repo has a `SchemaVerificationTest` or `schema.sql` in `src/test/resources/db/schema/`
- If present, ensure the new table DDL is included so the schema verification test passes

### 11. Dependencies (`pom.xml`)
- Ensure `dept44-starter-jpa` is present as a dependency
- Ensure `dept44-starter-jpa-test` is present with `<scope>test</scope>`
- Ensure `dept44-common-validators` is present if using `@ValidMunicipalityId`, `@ValidUuid`, etc.

### 12. Application class
- `Application.java` must have `@ExcludeFromJacocoGeneratedCoverageReport` annotation

### 13. Test profile config (`application-junit.yml`)
- Ensure datasource is configured for tests:
  ```yaml
  spring:
    flyway:
      enabled: true
    datasource:
      driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
      url: jdbc:tc:mariadb:10.6:///junit
      hikari:
        maximum-pool-size: 2
  ```

## Important
- Look at existing code in the repo for exact patterns before generating
- Use `final` on all variables and parameters
- Static imports for enums/constants
- No Lombok, no wildcard imports
- `@PathVariable` without redundant name — use `@PathVariable`, not `@PathVariable("name")`
- Problem imports use `se.sundsvall.dept44.problem` (not `org.zalando.problem`)
- Status codes use `org.springframework.http.HttpStatus` (e.g. `NOT_FOUND`, `BAD_REQUEST`)
- After creating all files, run `mvn test` to verify unit tests pass, then `mvn verify` to check integration tests and coverage
