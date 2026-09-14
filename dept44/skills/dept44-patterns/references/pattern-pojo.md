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
- **Multi-line `@Schema` descriptions use Java text blocks (`"""…"""`), not `+` string concatenation.** Use `\` line-continuations to keep one logical line so the generated OpenAPI text is unchanged. (Exception: a description that splices in a constant — `"… " + SOME_CONSTANT` — must stay concatenated; text blocks can't interpolate.)
- Jakarta validation: `@NotBlank`, `@Null(groups = OnCreate.class)`, etc.
- **`@Schema(accessMode = READ_ONLY)` is documentation-only and does NOT enforce anything at runtime** — it just sets `readOnly: true` in the OpenAPI spec; Jackson still binds the field from an incoming request body. Server-side read-only fields (e.g. `id`, `errandNumber`) still need `@Null(groups = OnCreate.class)` to actually reject a client setting them on create. Don't drop the `@Null` thinking `accessMode` covers it. (The Jackson annotation `@JsonProperty(access = READ_ONLY)` *would* strip it on input — but that's a different annotation and not the convention here.)
- No enums — use `String` + `@MemberOf(MyEnum.class)` or `@OneOf({"val1", "val2"})` for validation + `@Schema(allowableValues = {"VAL1", "VAL2"})` to expose values in OpenAPI
- **Records** (used for simple immutable DTOs) get `equals`/`hashCode`/`toString` for free — but a `record` with a `byte[]` component needs a manual content-aware `equals`/`hashCode`/`toString` (array identity is the default and is almost always wrong)
- **Name fields for what they hold, not where a call came from.** A field whose values classify *what something is* should not be named `origin`/`source` (those read as provenance). E.g. an attachment's `APPLICATION/CONVERSATION/CASE_DATA/DECISION` facet is a `documentType`, not an `origin`.

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
import org.hamcrest.MatcherAssert;                            // class import for the bean check
import static org.assertj.core.api.Assertions.assertThat;     // bare assertThat for everything else
// ...
// 1. BeanMatchers validation — Hamcrest assertThat, class-qualified (see import note below)
@Test
void testBean() {
    MatcherAssert.assertThat(ErrandAttachment.class, allOf(
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

### `assertThat` import collision (AssertJ vs Hamcrest)

Both AssertJ (`org.assertj.core.api.Assertions.assertThat`) and Hamcrest (`org.hamcrest.MatcherAssert.assertThat`, used by BeanMatchers `testBean`) export a method named `assertThat`. Only one can be static-imported per file. **Static-import the AssertJ one** (used many times — every field/builder assertion, written bare) and **class-import the Hamcrest one** so the single bean check reads `MatcherAssert.assertThat`:

```java
import org.hamcrest.MatcherAssert;                          // class import for the one bean check
import static org.assertj.core.api.Assertions.assertThat;   // the many AssertJ assertions, bare
// ...
assertThat(result.getId()).isEqualTo(id);                  // AssertJ, bare
MatcherAssert.assertThat(Foo.class, allOf(...));           // Hamcrest bean check, class-qualified
```

Do **not** static-import Hamcrest and fully-qualify every AssertJ call (litters the file with `org.assertj.core.api.Assertions.assertThat(...)`), and do **not** use the full `org.hamcrest.MatcherAssert.assertThat(...)` package path — class-qualify via the import. The `greve` plugin's linter flags both (`static-import-assertj`), if you use it. If the test has no BeanMatchers check, just static-import AssertJ normally.
