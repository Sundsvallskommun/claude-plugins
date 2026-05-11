# AbstractAppTest API

Source: `<repos>/dept44/dept44-starter-test/src/main/java/se/sundsvall/dept44/test/AbstractAppTest.java`

Base class for all AppTest integration tests. This is the most frequently referenced dept44 class — read the source above instead of searching for it each time.

## When to use

AppTests are end-to-end tests that boot the full Spring application, use WireMock to stub external services, and verify the entire request/response cycle including JSON serialization, validation, and database state. Use them for verifying happy-path flows and error scenarios that span multiple layers.

## Class Setup

```java
@WireMockAppTestSuite(files = "classpath:/{TestClassName}/", classes = Application.class)
@Sql(scripts = {
    "/db/scripts/truncate.sql",
    "/db/scripts/testdata-it.sql"
})
class MyFeatureIT extends AbstractAppTest {

    private static final String PATH = "/{municipalityId}/entities/{id}";
    private static final String MUNICIPALITY_ID = "2281";
    private static final String REQUEST_FILE = "request.json";
    private static final String RESPONSE_FILE = "response.json";

    @Test
    void test01_getEntity() throws Exception {
        // ...
    }
}
```

Test method names must start with `test` (stack-trace introspection). Convention: `test01_`, `test02_` for ordering.

## Fluent Call Chain

Always starts with `setupCall()`. Always ends with `sendRequestAndVerifyResponse()` or `sendRequest()` + `verifyStubs()`.

### GET

```java
setupCall()
    .withHttpMethod(GET)
    .withServicePath("/%s/entities/%s".formatted(MUNICIPALITY_ID, entityId))
    .withExpectedResponseStatus(OK)
    .withExpectedResponseHeader(CONTENT_TYPE, List.of(APPLICATION_JSON_VALUE))
    .withExpectedResponse(RESPONSE_FILE)
    .sendRequestAndVerifyResponse();
```

### POST (with Location header)

```java
setupCall()
    .withHttpMethod(POST)
    .withServicePath("/%s/entities".formatted(MUNICIPALITY_ID))
    .withRequest(REQUEST_FILE)
    .withExpectedResponseStatus(CREATED)
    .withExpectedResponseHeader(LOCATION, List.of("^/.*$"))
    .withExpectedResponseBodyIsNull()
    .sendRequestAndVerifyResponse();
```

### DELETE

```java
setupCall()
    .withHttpMethod(DELETE)
    .withServicePath("/%s/entities/%s".formatted(MUNICIPALITY_ID, entityId))
    .withExpectedResponseStatus(NO_CONTENT)
    .withExpectedResponseBodyIsNull()
    .sendRequestAndVerifyResponse();
```

### With query parameters (URI builder)

```java
setupCall()
    .withHttpMethod(GET)
    .withServicePath(builder -> builder
        .path("/{mId}/entities")
        .queryParam("page", 0)
        .queryParam("size", 10)
        .build(MUNICIPALITY_ID))
    .withExpectedResponseStatus(OK)
    .withExpectedResponse(RESPONSE_FILE)
    .sendRequestAndVerifyResponse();
```

### Multi-step / async side effects

Use `sendRequest()` + `andVerifyThat()` + `verifyStubs()` when you need to assert database state or other side effects after the call:

```java
setupCall()
    .withHttpMethod(POST)
    .withServicePath("/%s/trigger".formatted(MUNICIPALITY_ID))
    .withExpectedResponseStatus(NO_CONTENT)
    .sendRequest()
    .andVerifyThat(() -> repository.count() == 1)
    .verifyStubs();
```

## Method Reference

| Method | Purpose |
|---|---|
| `setupCall()` | Resets state, loads WireMock stubs for current test method |
| `withHttpMethod(HttpMethod)` | Sets HTTP method |
| `withServicePath(String)` | Sets the URL path |
| `withServicePath(Function<UriBuilder,URI>)` | URI builder variant (for query params) |
| `withRequest(String)` | Request body — filename in test dir OR raw JSON string |
| `withRequest(MultiValueMap)` | Multipart request body |
| `withRequestFile(String param, String filename)` | File part in multipart body |
| `withRequestReplacement(String match, String repl)` | String replace in request body (call after `withRequest`) |
| `withContentType(MediaType)` | Override default `application/json` |
| `withHeader(String key, String value)` | Add request header |
| `withExpectedResponseStatus(HttpStatus)` | Expected HTTP status |
| `withExpectedResponse(String)` | Expected JSON body — filename or raw JSON |
| `withExpectedBinaryResponse(String filename)` | Expected binary body |
| `withExpectedResponseBodyIsNull()` | Assert null response body |
| `withExpectedResponseHeader(String, List<String>)` | Assert response headers (values are regex) |
| `withExpectedResponseType(Class<?>)` | Override deserialization type (default: `String`) |
| `withJsonAssertOptions(List<Option>)` | Override JsonAssert options (null = strict mode) |
| `withMaxVerificationDelayInSeconds(int)` | WireMock/Awaitility timeout (default: 5s) |
| `sendRequestAndVerifyResponse()` | Execute + verify all WireMock stubs were hit |
| `sendRequest()` | Execute + assert status/body only (no WireMock verification) |
| `verifyStubs()` | Verify all stubs hit (call after `sendRequest()`) |
| `andVerifyThat(Callable<Boolean>)` | Await async condition (Awaitility, up to max delay) |
| `andReturnBody(Class<T>)` | Deserialize response to type |
| `andReturnBody(TypeReference<T>)` | Deserialize response to generic type |
| `getResponseHeaders()` | Return response headers |
| `getTestDirectoryPath()` | Classpath path to `__files/<testName>/` |

## Resource File Structure

```
src/integration-test/resources/
└── {TestClassName}/
    └── __files/
        ├── shared/                                 # some repos call this `common/` — match the repo
        │   └── responses/
        │       └── api-gateway-token-response.json # body returned by the OAuth token stub
        └── {testMethodName}/                       # one folder per test method
            ├── mappings/                           # WireMock stub mappings
            │   ├── api-some-service-get.json
            │   └── api-gateway-token.json          # references ../../shared/responses/api-gateway-token-response.json
            └── response/
                └── response.json                  # expected response body
```

Always look at an existing AppTest in the repo for the exact directory names and stub conventions — they vary slightly between services. See also the `pattern-apptest` reference in the `dept44-patterns` skill.
