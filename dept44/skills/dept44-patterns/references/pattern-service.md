# Service Layer Pattern Reference

Reference for dept44 service classes and their test patterns. Look at existing services in `service/` for exact style.

## Service Class

- Package: `service/`
- `@Service` annotation
- Constructor injection with all dependencies `final`
- Error messages as `private static final String` constants with `%s` format placeholders, formatted via `.formatted(...)`
- `Problem.valueOf(NOT_FOUND, message)` for not-found cases — never return null
- Static imports for mapper methods (e.g. `toNotificationEntity`, `updateEntity`)
- All parameters are `final`
- `@Transactional` on methods with multiple write operations
- Delegates mapping to `Mapper` classes — no mapping logic in service
- Delegates external calls to `Integration` classes or repositories
- Business logic in `private` helpers: `applyBusinessLogicForCreate`, `applyBusinessLogicForUpdate`
- Uses `Optional` fluent chains: `.map(...).orElseThrow(() -> Problem.valueOf(...))`
- `hasText()` from Spring for null/empty string checks

## Service Test Pattern

```java
@ExtendWith(MockitoExtension.class)
class {Name}ServiceTest {

    private static final String NAMESPACE = "my.namespace";
    private static final String MUNICIPALITY_ID = "2281";
    private static final String ERRAND_ID = "cb20c51f-fcf3-42c0-b613-de563634a8ec";

    @Mock
    private {Name}Repository repositoryMock;

    @Mock
    private {Other}Integration integrationMock;

    @InjectMocks
    private {Name}Service service;

    @Captor
    private ArgumentCaptor<Specification<{Name}Entity>> specificationCaptor;

    @Test
    void getErrand() {
        // Setup
        final var entity = {Name}Entity.create().withId(ERRAND_ID);

        // Mock
        when(repositoryMock.findByIdAndNamespaceAndMunicipalityId(ERRAND_ID, NAMESPACE, MUNICIPALITY_ID))
            .thenReturn(Optional.of(entity));

        // Act
        final var result = service.getErrand(NAMESPACE, MUNICIPALITY_ID, ERRAND_ID);

        // Verify
        assertThat(result).isNotNull();
        verify(repositoryMock).findByIdAndNamespaceAndMunicipalityId(ERRAND_ID, NAMESPACE, MUNICIPALITY_ID);
    }

    @Test
    void getErrandNotFound() {
        // Mock
        when(repositoryMock.findByIdAndNamespaceAndMunicipalityId(ERRAND_ID, NAMESPACE, MUNICIPALITY_ID))
            .thenReturn(Optional.empty());

        // Act
        final var exception = assertThrows(ThrowableProblem.class,
            () -> service.getErrand(NAMESPACE, MUNICIPALITY_ID, ERRAND_ID));

        // Verify
        assertThat(exception.getStatus()).isEqualTo(NOT_FOUND);
        verify(repositoryMock).findByIdAndNamespaceAndMunicipalityId(ERRAND_ID, NAMESPACE, MUNICIPALITY_ID);
    }
}
```

Key conventions:
- `@Mock` for all dependencies, `@InjectMocks` for service
- `@Captor` for `ArgumentCaptor` when capturing complex arguments (e.g. `Specification`)
- Verify every mock interaction explicitly
- `verifyNoInteractions()` for mocks that should NOT have been called
- `usingRecursiveComparison().isEqualTo(...)` for complex object comparisons
