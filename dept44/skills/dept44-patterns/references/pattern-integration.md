# Feign Integration Pattern Reference

Reference for dept44 external service integrations. Look at existing integrations in `integration/` for exact style.

## File Structure

```
integration/{servicename}/
├── {Service}Client.java              # Feign interface
├── {Service}Integration.java         # Wrapper with business logic (optional)
└── configuration/
    ├── {Service}Configuration.java   # Feign config bean
    └── {Service}Properties.java      # Config properties record
```

## Feign Client

```java
@FeignClient(
    name = CLIENT_ID,
    configuration = {Service}Configuration.class,
    url = "${integration.{servicename}.url}")
@CircuitBreaker(name = CLIENT_ID)
public interface {Service}Client {

    @GetMapping("/path/{id}")
    ResponseType getById(@PathVariable final String id);
}
```

- `CLIENT_ID` imported statically from Configuration class
- `@CircuitBreaker` on the interface
- Path variables are `final`

## Configuration

```java
@Import(FeignConfiguration.class)
public class {Service}Configuration {

    public static final String CLIENT_ID = "{servicename}";

    @Bean
    FeignBuilderCustomizer feignBuilderCustomizer(final {Service}Properties properties) {
        return FeignMultiCustomizer.create()
            .withErrorDecoder(new ProblemErrorDecoder(CLIENT_ID, List.of(NOT_FOUND.value())))
            .withRetryableOAuth2InterceptorForClientRegistration(properties.oauth2ClientId())
            .withRequestTimeoutsInSeconds(properties.connectTimeout(), properties.readTimeout())
            .composeCustomizersToOne();
    }
}
```

## Properties

```java
@ConfigurationProperties("integration.{servicename}")
public record {Service}Properties(
    String url,
    String oauth2TokenGrantUrl,
    String oauth2ClientId,
    String oauth2ClientSecret,
    int connectTimeout,
    int readTimeout) {
}
```

## Integration Tests (three per integration)

### 1. `{Service}IntegrationTest` — Unit test

```java
@ExtendWith(MockitoExtension.class)
class {Service}IntegrationTest {

    @Mock
    private {Service}Client clientMock;

    @InjectMocks
    private {Service}Integration integration;

    @Test
    void getById() {
        when(clientMock.getById("id")).thenReturn(new ResponseType());
        final var result = integration.getById("id");
        assertThat(result).isNotNull();
        verify(clientMock).getById("id");
    }
}
```

### 2. `{Service}ConfigurationTest` — Verifies customizer chain

- Uses `MockedStatic<FeignMultiCustomizer>` and `@Spy` on configuration
- Verifies `withErrorDecoder`, `withRetryableOAuth2InterceptorForClientRegistration`, `withRequestTimeoutsInSeconds`

### 3. `{Service}PropertiesTest` — Spring Boot property binding

```java
@SpringBootTest(classes = Application.class)
@ActiveProfiles("junit")
class {Service}PropertiesTest {

    @Autowired
    private {Service}Properties properties;

    @Test
    void testProperties() {
        assertThat(properties.url()).isEqualTo("http://localhost:8080");
        assertThat(properties.connectTimeout()).isEqualTo(5);
        assertThat(properties.readTimeout()).isEqualTo(30);
    }
}
```
