# Mapper Pattern Reference

Reference for dept44 mapper classes and their test patterns. Look at existing mappers in `service/mapper/` for exact style.

## Mapper Class

- Package: `service/mapper/`
- `{Domain}Mapper` class with **private constructor** (utility class)
- All methods are `static` — no instance state
- Null-safe: heavy use of `ofNullable()`, `emptyList()`, `emptyMap()`
- Method naming: `to{Target}()`, `to{Target}Entity()`, `to{Target}List()`, `updateEntity()`
- Import statically into service classes

## Mapper Test Pattern

```java
@ExtendWith(MockitoExtension.class)
class {Name}MapperTest {

    // Systematic input coverage with parameterized tests
    @ParameterizedTest
    @MethodSource("toEntityArguments")
    void toEntity(final {Name} input, final {Name}Entity expected) {
        final var result = {Name}Mapper.to{Name}Entity(input);

        if (expected == null) {
            assertThat(result).isNull();
        } else {
            assertThat(result)
                .usingRecursiveComparison()
                .isEqualTo(expected);
        }
    }

    private static Stream<Arguments> toEntityArguments() {
        return Stream.of(
            Arguments.of(null, null),
            Arguments.of(
                {Name}.create().withField("value"),
                {Name}Entity.create().withField("value"))
        );
    }

    // List mapping verification
    @Test
    void toList() {
        final var entities = List.of(
            {Name}Entity.create().withId("1").withField("a"),
            {Name}Entity.create().withId("2").withField("b"));

        final var result = {Name}Mapper.to{Name}List(entities);

        assertThat(result)
            .extracting({Name}::getId, {Name}::getField)
            .containsExactly(tuple("1", "a"), tuple("2", "b"));
    }

    // Null input returns empty collection
    @Test
    void toListWithNull() {
        assertThat({Name}Mapper.to{Name}List(null)).isEmpty();
    }

    // Uses hasAllNullFieldsOrPropertiesExcept for partial mapping
    @Test
    void toEntityWithMinimalInput() {
        final var result = {Name}Mapper.to{Name}Entity({Name}.create().withField("only-this"));

        assertThat(result).hasAllNullFieldsOrPropertiesExcept("field");
        assertThat(result.getField()).isEqualTo("only-this");
    }
}
```

Key conventions:
- `@ParameterizedTest` with `@MethodSource` for systematic coverage
- `private static Stream<Arguments>` as test data providers
- Always test null inputs (should return null or empty collections)
- `extracting(...).containsExactly(tuple(...))` for list verification
