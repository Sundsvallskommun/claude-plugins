# Scaffold New Feign Integration

Create a complete Feign client integration for an external service following dept44 patterns. Use existing integrations in this repo as reference.

## Arguments
- `$ARGUMENTS` — the external service name (e.g. "citizen", "emailreader", "messaging")

## What to create

Examine the existing `integration/` packages to match the exact style used in this repo.

### 1. Feign Client (`integration/{servicename}/`)
- `{Service}Client.java` — interface with `@FeignClient`
- `@FeignClient(name = CLIENT_ID, configuration = {Service}Configuration.class, url = "${integration.{servicename}.url}")`
- `@CircuitBreaker(name = CLIENT_ID)` on the interface
- Standard Spring MVC method annotations (`@GetMapping`, etc.)
- `CLIENT_ID` imported statically from the Configuration class

### 2. Configuration (`integration/{servicename}/configuration/`)
- `{Service}Configuration.java`
  - `@Import(FeignConfiguration.class)`
  - `public static final String CLIENT_ID = "{servicename}"`
  - `@Bean FeignBuilderCustomizer` using `FeignMultiCustomizer.create()` chain:
    - `.withErrorDecoder(new ProblemErrorDecoder(CLIENT_ID, List.of(NOT_FOUND.value())))`
    - `.withRetryableOAuth2InterceptorForClientRegistration(...)`
    - `.withRequestTimeoutsInSeconds(properties.connectTimeout(), properties.readTimeout())`
    - `.composeCustomizersToOne()`

### 3. Properties (`integration/{servicename}/configuration/`)
- `{Service}Properties.java` — record with `@ConfigurationProperties("integration.{servicename}")`
- Fields: `url`, `oauth2TokenGrantUrl`, `oauth2ClientId`, `oauth2ClientSecret`, `connectTimeout`, `readTimeout`

### 4. Integration wrapper (optional, `integration/{servicename}/`)
- `{Service}Integration.java` — `@Component`, wraps the Client
- Only create if business logic is needed around the client calls (error handling, transformation)

### 5. Application config
- Add to `application.yml` and `application-junit.yml`:
  ```yaml
  integration.{servicename}:
    url: <base-url>
    oauth2-token-grant-url: <token-url>
    oauth2-client-id: <client-id>
    oauth2-client-secret: <client-secret>
    connect-timeout: 5
    read-timeout: 30
  spring.security.oauth2.client:
    provider.{servicename}.token-uri: <token-url>
    registration.{servicename}:
      client-id: <client-id>
      client-secret: <secret>
      authorization-grant-type: client_credentials
  ```

### 6. OpenAPI spec
- Add the external service's OpenAPI spec to `src/main/resources/integrations/api-{servicename}.yaml`
- Add `openapi-generator-maven-plugin` execution in `pom.xml` for model generation

### 7. Tests
- `{Service}IntegrationTest.java` — `@ExtendWith(MockitoExtension.class)`, mocks Client, verifies delegation
- `{Service}ConfigurationTest.java` — `MockedStatic<FeignMultiCustomizer>`, `@Spy`, verifies customizer chain
- `{Service}PropertiesTest.java` — `@SpringBootTest`, `@ActiveProfiles("junit")`, asserts values from `application-junit.yml`

### 8. Dependencies (`pom.xml`)
- Ensure `dept44-starter-feign` is present as a dependency

## Important
- Look at existing integrations in this repo for exact patterns
- Use `final` on all variables and parameters
- Static imports for `CLIENT_ID` and enum values
- No Lombok, no wildcard imports
