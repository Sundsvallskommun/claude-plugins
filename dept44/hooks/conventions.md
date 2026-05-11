# dept44 Microservices — Claude Code Instructions

## Organization

- **GitHub org:** Sundsvallskommun
- **Framework:** dept44 (se.sundsvall.dept44)
- **All services** follow the naming pattern `api-service-{name}` in GitHub
- **Municipality ID** `2281` is Sundsvall's municipality code, used as path parameter in APIs
- **All services** are in the Sundsvallskommun GitHub org. If a sibling service repo is cloned locally, prefer reading from the filesystem over fetching from GitHub.

## Tech Stack

- Java 25 (`maven.compiler.release=25`)
- Spring Boot (managed via dept44-starter-parent / dept44-service-parent)
- Maven (no Gradle)
- MariaDB with Flyway migrations (disabled by default)
- Docker for containerization
- OAuth2 client credentials for service-to-service auth
- OpenAPI / Swagger for API documentation at `/api-docs`

## Framework Modules (dept44)

Parent POM: `se.sundsvall.dept44:dept44-service-parent` — all microservices extend this.

Key starters (add as needed, don't include what you don't use):

- `dept44-starter` — core (always included via parent)
- `dept44-starter-feign` — Feign clients for service-to-service calls
- `dept44-starter-webclient` — Spring WebClient setup
- `dept44-starter-webservicetemplate` — SOAP integration
- `dept44-starter-authorization` — authorization/security config
- `dept44-starter-test` — test utilities
- `dept44-starter-logback-logserver` — structured logging
- `dept44-starter-scheduler` — scheduled job support
- `dept44-common-validators` — shared validation annotations
- `dept44-models` — shared domain models
- `dept44-build-tools` — build checks for OpenAPI properties, truststores
- `dept44-formatting-plugin` — enforces code formatting standard

## Package Structure

Base package: `se.sundsvall.{servicename}` (e.g. `se.sundsvall.supportmanagement`)

```
se.sundsvall.{servicename}/
├── Application.java
├── api/                    # REST controllers (Resource classes)
│   ├── model/              # API request/response DTOs
│   ├── validation/         # Custom validation annotations
│   └── validation/impl/    # Annotation validator implementations
├── service/                # Business logic
│   └── mapper/             # Object transformation (static utility classes)
├── integration/            # External service clients (Feign, WebClient)
│   └── {servicename}/      # One sub-package per external dependency
├── integration/db/         # JPA entities and Spring Data repositories
│   └── model/              # Entity classes
├── configuration/          # Spring @Configuration classes
└── util/                   # Utility classes
```

## Coding Conventions

- REST controllers in `api/` package, named `{Entity}Resource` (not Controller)
- API paths include municipality ID: `/{municipalityId}/...`
- **No enums in API model layer** — use `String` fields + custom validation annotations. Enums fine internally.
- **No Lombok** — write getters, setters, constructors, builders explicitly
- Use records for simple immutable DTOs when no mutation is needed
- **Prefer `Optional` over ternary (`? :`) and null checks** — use `Optional.ofNullable()`, `.map()`, `.orElse()`, `.ifPresent()` etc. for all nullable handling
- **Variables should be `final` whenever possible**
- **Always use explicit imports — never wildcard imports (`import x.y.*`)**
- **Always import classes** rather than using fully-qualified names inline (e.g. `new Party()` not `new generated.se.sundsvall.comfact.Party()`). Only use fully-qualified names when two classes share the same simple name and both are needed in the same file.
- **Static imports for enums and constants** — write `BAD_REQUEST`, not `HttpStatus.BAD_REQUEST`
- Custom validation annotations in `api/validation/` with impls in `api/validation/impl/`
- **AssertJ** for test assertions (except BeanMatchers which use Hamcrest)
- Format with `mvn dept44-formatting:format`; checkstyle enforced by CI
- Component structure & tests (Resource / POJO / Entity / Service / Mapper / Integration / Scheduler): see the `pattern-*` commands or the `dept44-patterns` skill for the canonical layout of each — Resources are package-private and thin (delegate all logic to the service layer); services use constructor injection; mappers are private-constructor static utility classes
- Error handling: dept44 Problem (`se.sundsvall.dept44.problem.Problem`) — `Problem.valueOf(STATUS, message)` in the service/integration layers, never in Resources
- All Feign clients and repositories require `@CircuitBreaker`
- Schedulers use `@Dept44Scheduled` (NOT Spring's `@Scheduled`)

## Testing Requirements

- **85% line + 85% branch coverage** (JaCoCo enforced, builds fail below this)
- JUnit 5 with `dept44-starter-test` utilities
- Resource: two test classes — `{Resource}Test` (happy path) + `{Resource}FailureTest` (validation)
- POJO/Entity: `testBean()` (BeanMatchers), `testBuilderMethods()`, `testNoDirtOnCreatedBean()`
- Service: `@ExtendWith(MockitoExtension.class)`, mock deps, verify all interactions
- Integration tests: WireMock + Docker compose in `src/test/resources/docker/`
- AppTests: extend `AbstractAppTest` with `@WireMockAppTestSuite`
- Use URL-safe invalid values in failure tests — do NOT use `#` (URL fragment delimiter truncates silently)
- OpenAPI contract test: `OpenApiSpecificationIT` compares the runtime-generated spec against the checked-in baseline at `src/integration-test/resources/api/openapi.yaml`

## Database

- MariaDB with Flyway migrations in `src/main/resources/db/migration/`
- Naming: `V{version}__{description}.sql` (e.g. `V1_0__create_tables.sql`)
- Enable with `spring.flyway.enabled=true`
- Integration test SQL in `src/integration-test/resources/db/scripts/` (truncate.sql, testdata-it.sql)
- DB scripts may also be in `tools/db_scripts/`

## OpenAPI / Contract-First

- The contract-test baseline is `src/integration-test/resources/api/openapi.yaml`, loaded by `OpenApiSpecificationIT` as `classpath:/api/openapi.yaml`. Some older services keep it elsewhere (e.g. `src/test/resources/api/openapi.yml`) — when you touch such a service, move it to the canonical path and update the test.
- The spec is also **generated at build time** — `OpenApiSpecificationIT` writes the live spec to `target/api.yaml`. If the baseline is missing or stale, run `mvn verify` then copy `target/api.yaml` over `src/integration-test/resources/api/openapi.yaml` (after eyeballing the diff).
- Keep the baseline in sync with the Resource classes — the contract test fails on any mismatch.
- External service specs in `src/main/resources/integrations/api-{servicename}.yaml`
- Models generated via `openapi-generator-maven-plugin` into `generated.se.sundsvall.*`
- Only models generated (`generateApis: false`, `generateSupportingFiles: false`)
- Don't manually edit generated classes — update spec and regenerate
- **Generated sources live in `target/generated-sources/`** and are NOT checked in. If they're missing or stale (e.g. after a clean, spec change, or dependency upgrade), run `mvn generate-sources` to regenerate them. Compilation errors in `generated.se.sundsvall.*` usually mean regeneration is needed.

## Service-to-Service Communication

- OAuth2 client credentials flow for auth between services
- Feign clients (via `dept44-starter-feign`) or WebClient for HTTP calls
- Config pattern in `application.yml`:

  ```yaml
  spring.security.oauth2.client.provider.{service}.token-uri: <url>
  spring.security.oauth2.client.registration.{service}:
    client-id: <id>
    client-secret: <secret>
    authorization-grant-type: client_credentials
  integration.{service}.url: <base-url>
  ```

## Build & Run

```bash
mvn clean verify              # Build + all tests
mvn test                      # Unit tests only
mvn spring-boot:run           # Run locally
mvn dept44-formatting:format  # Format code
```

## Things to Avoid

- **Never extract or decompile JARs** (no `jar xf`, `javap`, `unzip` on JARs) — if the dept44 framework source is cloned locally, read it directly. Use WebSearch only as a fallback.
- Don't add dependencies not managed by dept44-service-parent without good reason
- Don't skip tests — 85% coverage enforced, builds fail
- Don't put business logic in Resource classes — keep them thin
- Don't create endpoints without updating the OpenAPI spec
- Don't ignore the formatting plugin — CI rejects unformatted code
- Don't hardcode municipality IDs — always accept as path parameter

## Pattern References (MANDATORY)

**Before writing or modifying any component, you MUST invoke the corresponding slash command and follow the pattern exactly.** These patterns define the canonical structure, naming, testing, and style for every component type. Do not deviate from them.

| Component | Slash command to invoke |
|---|---|
| Resource (controller) | `/dept44:pattern-resource` |
| API model (POJO/DTO) | `/dept44:pattern-pojo` |
| JPA entity + repository | `/dept44:pattern-entity` |
| Service | `/dept44:pattern-service` |
| Mapper | `/dept44:pattern-mapper` |
| Feign integration | `/dept44:pattern-integration` |
| Scheduler | `/dept44:pattern-scheduler` |
| Integration/AppTest | `/dept44:pattern-apptest` |
| Custom validators | Use the `pattern-validator` skill |

When scaffolding new components from scratch, also invoke:
- `/dept44:new-entity`
- `/dept44:new-endpoint-crud`
- `/dept44:new-endpoint-proxy`
- `/dept44:new-integration`
- `/dept44:new-scheduler`
- `/dept44:new-apptest`

For dept44 internals lookup (class APIs, annotations, key source paths): use the `dept44-source` skill
