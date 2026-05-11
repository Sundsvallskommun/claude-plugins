# Scaffold Integration Tests (AppTests) & OpenAPI Contract Test

Create integration tests and an OpenAPI contract test for a dept44 microservice. Use existing code in this repo and sibling repos as reference.

## Arguments
- `$ARGUMENTS` — (optional) the entity/resource name or notes about what to test

## Prerequisites
- The service must already have Resource and Service classes
- Examine the project to determine if it uses a database (has Entity/Repository/Flyway) or is a pure proxy (Feign integrations only, no DB)

## What to create

Examine the existing Resource class to identify all endpoints, path patterns, request/response models, and HTTP methods. Look at the Service class to understand error messages for NOT_FOUND/CONFLICT responses.

### 1. Integration test profile (`src/main/resources/application-it.yml`)

Merge with any existing content (keep existing logging config etc.).

**If the service uses a database:**
```yaml
spring:
  datasource:
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
    url: jdbc:tc:mariadb:10.6:///
  flyway:
    enabled: true
  jpa:
    properties:
      jakarta:
        persistence:
          schema-generation:
            database:
              action: validate
```

**If the service is a pure proxy (no database):**
- No datasource/flyway/jpa config needed
- Only logging and any integration-specific config (e.g. WireMock base URLs)

### 2. SQL scripts (only if service uses a database)

Create `src/integration-test/resources/db/scripts/`:

**`truncate.sql`**
- `TRUNCATE TABLE {table_name};` for each table
- If foreign keys exist, wrap with `SET FOREIGN_KEY_CHECKS = 0;` / `SET FOREIGN_KEY_CHECKS = 1;`

**`testdata-it.sql`**
- INSERT statements with realistic test data
- Use explicit IDs (not auto-generated) so test assertions can reference them
- Include data for multiple test scenarios (happy path read, update, delete, not-found)
- Include data for multiple municipality IDs to test isolation if needed
- Use the same municipality ID `2281` as the main test constant

### 3. AppTest class (`src/integration-test/java/.../apptest/{Entity}IT.java`)

```java
@WireMockAppTestSuite(files = "classpath:/{Entity}IT/", classes = Application.class)
class {Entity}IT extends AbstractAppTest {
```

**If the service uses a database**, add `@Sql` to load test data:
```java
@Sql(scripts = {
    "/db/scripts/truncate.sql",
    "/db/scripts/testdata-it.sql"
})
```

**If the service is a pure proxy**, add WireMock stub mappings instead (no `@Sql`):
- Create `{Entity}IT/__files/{testMethodName}/mappings/` directories with WireMock JSON stubs for each external service call
- Include an OAuth token stub in `{Entity}IT/__files/common/responses/api-gateway-token-response.json` if the service uses OAuth2

**Test methods — match the actual endpoints in the Resource class:**

For a **database-backed CRUD** service:

| Test method | HTTP | Expected status | Verifies |
|---|---|---|---|
| `test01_create{Entity}` | POST | 201 CREATED | Location header, entity in DB after |
| `test02_read{Entity}` | GET (single) | 200 OK | Response body matches expected |
| `test03_read{Entity}s` / `find` | GET (list) | 200 OK | Response body matches expected list |
| `test04_update{Entity}` | PUT/PATCH | 200 OK | Response body, DB state after |
| `test05_delete{Entity}` | DELETE | 204 NO_CONTENT | Entity removed from DB after |
| `test06_read{Entity}NotFound` | GET | 404 NOT_FOUND | Error response body |
| `test07_delete{Entity}NotFound` | DELETE | 404 NOT_FOUND | Error response body |

For a **proxy** service, test each endpoint with matching WireMock stubs for the downstream calls.

**Patterns:**
- Use `setupCall()` builder: `.withServicePath()`, `.withHttpMethod()`, `.withRequest()`, `.withExpectedResponseStatus()`, `.withExpectedResponse()`, `.sendRequestAndVerifyResponse()`
- DB services: `@Autowired` the repository to assert DB state before/after operations
- Proxy services: WireMock verifies downstream calls were made correctly
- POST: verify Location header with regex, verify `withExpectedResponseBodyIsNull()` if void
- DELETE: verify `withExpectedResponseBodyIsNull()`, assert entity removed from DB
- Error responses: verify `response.json` with `title`, `status`, `detail` matching service error messages

### 4. JSON test fixtures (`src/integration-test/resources/{Entity}IT/__files/`)

**For DB-backed services** (no WireMock needed):
```
{Entity}IT/
└── __files/
    ├── test01_create{Entity}/
    │   └── request.json
    ├── test02_read{Entity}/
    │   └── response.json
    └── ...
```

**For proxy services** (with WireMock stubs):
```
{Entity}IT/
└── __files/
    ├── common/
    │   └── responses/
    │       └── api-gateway-token-response.json
    ├── test01_{methodName}/
    │   ├── mappings/
    │   │   ├── api-gateway-token.json
    │   │   └── api-{downstream-service}-{operation}.json
    │   ├── request.json          (if POST/PUT/PATCH)
    │   └── response.json
    └── ...
```

**JSON conventions:**
- Request JSON: only fields the client sends (match `@RequestBody` model)
- Response JSON: match exact API model output (field names from getter methods)
- Error responses: `{"title": "Not Found", "status": 404, "detail": "..."}`
- Use `${json-unit.any-string}` for generated fields (timestamps, UUIDs)
- Use `${json-unit.regex}^pattern$` for pattern-matched fields
- For DB services: response data must match the test data from `testdata-it.sql`

### 5. OpenAPI contract test (`src/integration-test/java/.../apptest/OpenApiSpecificationIT.java`)

```java
@ActiveProfiles("it")
@AutoConfigureTestRestTemplate
@SpringBootTest(
    webEnvironment = WebEnvironment.RANDOM_PORT,
    classes = Application.class,
    properties = {
        "spring.main.banner-mode=off",
        "logging.level.se.sundsvall.dept44.payload=OFF",
        "wiremock.server.port=10101"
    })
class OpenApiSpecificationIT {

    private static final YAMLMapper YAML_MAPPER = new YAMLMapper();

    @Value("${openapi.name}")
    private String openApiName;

    @Value("${openapi.version}")
    private String openApiVersion;

    @Value("classpath:/api/openapi.yaml")
    private Resource openApiResource;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void compareOpenApiSpecifications() throws IOException {
        final String currentOpenApiSpecification = getCurrentOpenApiSpecification();

        writeString(Path.of("target/api.yaml"), currentOpenApiSpecification);

        final String existingOpenApiSpecification = ResourceUtils.asString(openApiResource);
        assertThatJson(toJson(currentOpenApiSpecification))
            .withOptions(List.of(IGNORING_ARRAY_ORDER))
            .whenIgnoringPaths("servers")
            .isEqualTo(toJson(existingOpenApiSpecification));
    }
}
```

**Key imports:**
- `tools.jackson.dataformat.yaml.YAMLMapper`
- `org.springframework.boot.resttestclient.TestRestTemplate`
- `org.springframework.boot.resttestclient.autoconfigure.AutoConfigureTestRestTemplate`
- `se.sundsvall.dept44.util.ResourceUtils`
- `net.javacrumbs.jsonunit.assertj.JsonAssertions.assertThatJson`

**IMPORTANT:** Write `target/api.yaml` BEFORE loading the baseline — this ensures the generated spec is always captured even if the baseline is missing (bootstrap scenario).

### 6. OpenAPI baseline spec (`src/integration-test/resources/api/openapi.yaml`)

The canonical location is `src/integration-test/resources/api/openapi.yaml`, loaded by `OpenApiSpecificationIT` as `classpath:/api/openapi.yaml`. If the service already has a spec somewhere else (commonly `src/test/resources/api/openapi.yml` or a root-level `openapi.yml`), **move it** to the canonical path, fix the `@Value("classpath:/api/openapi.yaml")` in the test, and delete the old copy — don't leave two.

**Bootstrap process (no baseline yet):**
1. Create all IT test files (steps 1–5 above)
2. Run `mvn verify -Dsurefire.skip=true` — the OpenAPI test will fail because the baseline doesn't exist, but `target/api.yaml` will be written
3. Copy: `cp target/api.yaml src/integration-test/resources/api/openapi.yaml`
4. Run `mvn verify` again — all tests should pass

**Maintaining the spec:**
- When API changes (new endpoints, modified models), the OpenAPI test will fail
- Check `target/api.yaml` for the updated spec
- Copy it to `src/integration-test/resources/api/openapi.yaml` after verifying correctness

## Important
- Use `final` on all variables and parameters
- Static imports for HTTP methods, status codes, media types
- No Lombok, no wildcard imports
- Test method names prefixed: `test01_`, `test02_`, etc.
- Error detail messages must match the exact format from the Service class
- `REQUEST_FILE = "request.json"` and `RESPONSE_FILE = "response.json"` constants
- After creating all files, run `mvn verify` to confirm everything passes
