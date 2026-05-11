# POJO (API Model) Pattern Reference

Reference for dept44 API model classes and their test patterns. Look at existing models in `api/model/` for exact style.

## POJO Class

- Package: `api/model/`
- No Lombok. No inner builder class. The class itself is the builder.
- `static create()` factory method returning new instance
- Standard getters and setters (setter params are `final`)
- Fluent `with{Field}()` methods that set field and return `this`
- Manual `equals()`, `hashCode()` (using `Objects.equals`/`Objects.hash`), and `toString()` — use concatenation style:
  ```java
  return "ClassName{" +
      "field='" + field + '\'' +
      ", other=" + other +
      '}';
  ```
- `@DateTimeFormat(iso = DATE_TIME)` on all `OffsetDateTime` fields (import `org.springframework.format.annotation.DateTimeFormat` and static import `DateTimeFormat.ISO.DATE_TIME`)
- `@Schema` annotations on class and each field
- Jakarta validation: `@NotBlank`, `@Null(groups = OnCreate.class)`, etc.
- No enums — use `String` + custom validation annotations

```java
@Schema(description = "ErrandAttachment model")
public class ErrandAttachment {

    @Schema(description = "Unique identifier", examples = "cb20c51f-...", accessMode = READ_ONLY)
    @Null(groups = OnCreate.class)
    protected String id;

    @Schema(description = "Name of the file", examples = "my-file.txt")
    @NotBlank(groups = OnCreate.class)
    protected String fileName;

    public static ErrandAttachment create() {
        return new ErrandAttachment();
    }

    public String getId() {
        return id;
    }

    public void setId(final String id) {
        this.id = id;
    }

    public ErrandAttachment withId(final String id) {
        this.id = id;
        return this;
    }

    // ... same pattern for all fields

    @Override
    public boolean equals(final Object o) {
        if (o == null || getClass() != o.getClass()) return false;
        final ErrandAttachment that = (ErrandAttachment) o;
        return Objects.equals(id, that.id) && Objects.equals(fileName, that.fileName);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, fileName);
    }

    @Override
    public String toString() {
        return "ErrandAttachment{" +
            "id='" + id + '\'' +
            ", fileName='" + fileName + '\'' +
            '}';
    }
}
```

## POJO Test Pattern

Every POJO gets **three** tests in a single test class:

```java
// 1. BeanMatchers validation (uses Hamcrest assertThat, not AssertJ)
@Test
void testBean() {
    assertThat(ErrandAttachment.class, allOf(
        hasValidBeanConstructor(),
        hasValidGettersAndSetters(),
        hasValidBeanHashCode(),
        hasValidBeanEquals(),
        hasValidBeanToString()));
}

// 2. Fluent builder chain (uses AssertJ)
@Test
void testBuilderMethods() {
    // Arrange
    final var id = "id";
    final var fileName = "file.txt";

    // Act
    final var result = ErrandAttachment.create()
        .withId(id)
        .withFileName(fileName);

    // Assert
    assertThat(result).hasNoNullFieldsOrProperties();
    assertThat(result.getId()).isEqualTo(id);
    assertThat(result.getFileName()).isEqualTo(fileName);
}

// 3. Clean default state
@Test
void testNoDirtOnCreatedBean() {
    assertThat(ErrandAttachment.create()).hasAllNullFieldsOrProperties();
}
```

Note: Both AssertJ and Hamcrest `assertThat` are used — Hamcrest for BeanMatchers, AssertJ for everything else.
