# Testing Validators

## 1. Unit test the ConstraintValidator directly (no Spring context)

```java
class ValidMyFieldConstraintValidatorTest {

    private final ValidMyFieldConstraintValidator validator = new ValidMyFieldConstraintValidator();

    @Test
    void testValidValue() {
        assertThat(validator.isValid("valid-value", null)).isTrue();
    }

    @Test
    void testNullIsValid() {
        // Null must return true — @NotNull handles null separately
        assertThat(validator.isValid(null, null)).isTrue();
    }

    @Test
    void testInvalidValue() {
        assertThat(validator.isValid("bad-value", null)).isFalse();
    }
}
```

## 2. Integration test via `{Resource}FailureTest` (WebTestClient)

```java
@Test
void testInvalidMyField() {
    final var response = webTestClient.post()
        .uri(builder -> builder.path(PATH).build(Map.of("municipalityId", MUNICIPALITY_ID)))
        .contentType(APPLICATION_JSON)
        .bodyValue(MyRequest.create().withMyField("INVALID_VALUE"))
        .exchange()
        .expectStatus().isBadRequest()
        .expectBody(ConstraintViolationProblem.class)
        .returnResult()
        .getResponseBody();

    assertThat(response).isNotNull();
    assertThat(response.getTitle()).isEqualTo("Constraint Violation");
    assertThat(response.getStatus()).isEqualTo(BAD_REQUEST);
    assertThat(response.getViolations())
        .extracting(Violation::getField, Violation::getMessage)
        .containsExactlyInAnyOrder(tuple("create.request.myField", "must be a valid ..."));
}
```

**Note:** Use URL-safe invalid values. Do NOT use `#` (URL fragment delimiter — silently truncates path).

## 3. Testing multiple violations (complex validator)

```java
assertThat(response.getViolations())
    .extracting(Violation::getField, Violation::getMessage)
    .containsExactlyInAnyOrder(
        tuple("create.record.invoice.invoiceRows", "must contain vat code when type is EXTERNAL"),
        tuple("create.record.invoice.invoiceRows", "at least one invoice row must have accountInformation"));
```

## 4. Testing composite validators

Test each accepted variant as a separate `@ParameterizedTest`:

```java
@ParameterizedTest
@ValueSource(strings = {"199001011234", "5566112233"})
void testValidIdentifiers(final String identifier) {
    assertThat(validator.isValid(identifier, null)).isTrue();
}

@ParameterizedTest
@ValueSource(strings = {"not-a-number", "12345", "INVALID"})
void testInvalidIdentifiers(final String identifier) {
    assertThat(validator.isValid(identifier, null)).isFalse();
}
```
