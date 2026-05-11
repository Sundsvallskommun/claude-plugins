# Scaffold New Endpoint (Integration Proxy)

Create a new endpoint that proxies requests to an external service via a Feign integration. No local database. Use existing code in this repo as reference.

## Arguments
- `$ARGUMENTS` — the domain name and target service (e.g. "Message via messaging" or "CitizenInfo proxying to citizen service")

## What to create

Examine the existing package structure and integrations to determine the base package, service name, and whether the target integration already exists.

### 1. Check if the integration already exists
- Look in `integration/` for an existing Feign client to the target service
- If it exists, reuse it. If not, create it (see below).

### 2. API Model (`api/model/`) — if the API model differs from the generated model
- `{Name}.java` or `{Name}Request.java` / `{Name}Response.java`
- Only create if the external API's model needs to be adapted for this service's API
- If the generated model (`generated.se.sundsvall.*`) can be used directly via mapping, prefer that

### 3. Mapper (`service/mapper/`)
- `{Name}Mapper.java` — private constructor, all static methods
- Maps between this service's API models and the generated integration models
- `to{ExternalModel}(localModel)` — outbound mapping
- `to{Name}(externalModel)` — inbound mapping
- Null-safe with `ofNullable()`

### 4. Service (`service/`)
- `{Name}Service.java` — `@Service`, constructor injection
- Injects the `{Service}Client` or `{Service}Integration` (not a repository)
- Orchestrates: validate → map → call integration → map response
- `Problem.valueOf(...)` for error cases
- No `@Transactional` needed (no local DB writes)

### 5. Resource (`api/`)
- `{Name}Resource.java` — package-private, `@RestController`, `@Validated`
- `@RequestMapping("/{municipalityId}/...")` with `@ValidMunicipalityId`
- Endpoints matching what the external service offers (GET, POST, etc.)
- Full `@Operation` + `@ApiResponse`

### 6. Integration (only if it doesn't already exist)
If the target integration needs to be created:

- `{Service}Client.java` — `@FeignClient` interface with `@CircuitBreaker`
- `{Service}Configuration.java` — `@Import(FeignConfiguration.class)`, `CLIENT_ID`, `FeignBuilderCustomizer` bean
- `{Service}Properties.java` — `@ConfigurationProperties` record
- OpenAPI spec in `src/main/resources/integrations/api-{servicename}.yaml`
- `openapi-generator-maven-plugin` execution in `pom.xml`
- OAuth2 + integration config in `application.yml` and `application-junit.yml`

### 7. Tests
Create all test classes following CLAUDE.md patterns:
- `{Name}Test.java` — POJO tests (if API models were created)
- `{Name}MapperTest.java` — `@ParameterizedTest` with `@MethodSource`
- `{Name}ServiceTest.java` — `@ExtendWith(MockitoExtension.class)`, `@Mock` client/integration, `@InjectMocks` service
- `{Name}ResourceTest.java` — happy path, `@SpringBootTest`, `@AutoConfigureWebTestClient`, `WebTestClient`, `@MockitoBean`
- `{Name}ResourceFailureTest.java` — validation failures, `@AutoConfigureWebTestClient`
- Integration tests (if integration was created):
  - `{Service}IntegrationTest.java`
  - `{Service}ConfigurationTest.java`
  - `{Service}PropertiesTest.java`

### 8. Dependencies (`pom.xml`)
- Ensure `dept44-starter-feign` is present as a dependency
- Ensure `dept44-common-validators` is present if using `@ValidMunicipalityId`, `@ValidUuid`, etc.

### 9. Application class
- `Application.java` must have `@ExcludeFromJacocoGeneratedCoverageReport` annotation

## Important
- Look at existing code in the repo for exact patterns before generating
- Reuse existing integrations — don't duplicate Feign clients
- Use `final` on all variables and parameters
- Static imports for enums/constants
- No Lombok, no wildcard imports
- `@PathVariable` without redundant name — use `@PathVariable`, not `@PathVariable("name")`
- Problem imports use `se.sundsvall.dept44.problem` (not `org.zalando.problem`)
- Status codes use `org.springframework.http.HttpStatus` (e.g. `NOT_FOUND`, `BAD_REQUEST`)
