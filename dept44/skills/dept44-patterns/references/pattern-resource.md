# Resource (Controller) Pattern Reference

Reference for dept44 Resource classes and their test patterns. Look at existing Resources in this repo for exact style.

## Resource Class

- Package: `api/`
- Class name: `{Entity}Resource` (not Controller), **package-private** (no `public`)
- `@RestController`, `@Validated`, `@RequestMapping("/{municipalityId}/{namespace}/...")`
- `@Tag(name = "...")` for Swagger grouping
- Constructor injection (no `@Autowired`), all params `final`
- Every `@PathVariable` parameter is `final` — do NOT include redundant name (use `@PathVariable`, not `@PathVariable("name")`)
- Use dept44 validators: `@ValidMunicipalityId`, `@ValidUuid` on path variables
- `@Pattern` with constants for namespace validation
- **Class-level `@ApiResponses`** for 400 and 500 to reduce duplication:
  ```java
  @ApiResponses(value = {
      @ApiResponse(responseCode = "400", description = "Bad request", content = @Content(mediaType = APPLICATION_PROBLEM_JSON_VALUE, schema = @Schema(oneOf = {Problem.class, ConstraintViolationProblem.class}))),
      @ApiResponse(responseCode = "500", description = "Internal server error", content = @Content(mediaType = APPLICATION_PROBLEM_JSON_VALUE, schema = @Schema(implementation = Problem.class)))
  })
  ```
- Method-level `@Operation` only declares success response + endpoint-specific errors (404, 409, etc.)
- Error responses use `APPLICATION_PROBLEM_JSON_VALUE` with `Problem.class` and `ConstraintViolationProblem.class`
- Return types: `created(location).build()` for POST, `ok(result)` for GET, `noContent().build()` for DELETE
- Static imports for `ResponseEntity.created`, `ResponseEntity.ok`, `ResponseEntity.noContent`, media types, etc.

## Resource Test — Happy Path (`{Resource}Test`)

```java
@SpringBootTest(classes = Application.class, webEnvironment = RANDOM_PORT)
@AutoConfigureWebTestClient
@ActiveProfiles("junit")
class {Name}ResourceTest {

    @MockitoBean
    private {Name}Service serviceMock;

    @Autowired
    private WebTestClient webTestClient;

    private static final String NAMESPACE = "my.namespace";
    private static final String MUNICIPALITY_ID = "2281";
    private static final String ERRAND_ID = "cb20c51f-fcf3-42c0-b613-de563634a8ec";
    private static final String PATH = "/{municipalityId}/{namespace}/errands/{errandId}";

    @Test
    void getErrand() {
        // Parameter values
        final var errandId = ERRAND_ID;

        // Mock
        when(serviceMock.getErrand(NAMESPACE, MUNICIPALITY_ID, errandId)).thenReturn(Errand.create());

        // Call
        final var response = webTestClient.get()
            .uri(builder -> builder.path(PATH).build(Map.of(
                "municipalityId", MUNICIPALITY_ID,
                "namespace", NAMESPACE,
                "errandId", errandId)))
            .exchange()
            .expectStatus().isOk()
            .expectBody(Errand.class)
            .returnResult()
            .getResponseBody();

        // Verification
        assertThat(response).isNotNull();
        verify(serviceMock).getErrand(NAMESPACE, MUNICIPALITY_ID, errandId);
    }
}
```

## Resource Failure Test (`{Resource}FailureTest`)

```java
@SpringBootTest(classes = Application.class, webEnvironment = RANDOM_PORT)
@AutoConfigureWebTestClient
@ActiveProfiles("junit")
class {Name}ResourceFailureTest {

    @MockitoBean
    private {Name}Service serviceMock;

    @Autowired
    private WebTestClient webTestClient;

    // URL-safe invalid values — do NOT use # (URL fragment delimiter)
    private static final String INVALID_MUNICIPALITY_ID = "bad-municipality-id";
    private static final String INVALID_UUID = "not-a-valid-uuid";

    @Test
    void getErrandWithInvalidMunicipalityId() {
        final var response = webTestClient.get()
            .uri(builder -> builder.path(PATH).build(Map.of(
                "municipalityId", INVALID_MUNICIPALITY_ID,
                "namespace", NAMESPACE,
                "errandId", ERRAND_ID)))
            .exchange()
            .expectStatus().isBadRequest()
            .expectBody(ConstraintViolationProblem.class)
            .returnResult()
            .getResponseBody();

        assertThat(response).isNotNull();
        assertThat(response.getTitle()).isEqualTo("Constraint Violation");
        assertThat(response.getStatus()).isEqualTo(BAD_REQUEST);
        assertThat(response.getViolations())
            .extracting(Violation::field, Violation::message)
            .containsExactlyInAnyOrder(tuple("methodName.municipalityId", "not a valid municipality ID"));

        verifyNoInteractions(serviceMock);
    }
}
```
