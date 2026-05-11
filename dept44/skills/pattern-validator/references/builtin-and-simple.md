# Built-in Validators & Simple String Validator

## Built-in dept44 Validators

Always check these before writing a custom validator.
Source: `<repos>/dept44/dept44-common-validators/src/main/java/se/sundsvall/dept44/common/validators/annotation/`

| Annotation | Validates | Notes |
|---|---|---|
| `@ValidUuid` | UUID format | |
| `@ValidMunicipalityId` | Municipality code (e.g. `2281`) | Always use on `{municipalityId}` path variables |
| `@ValidPersonalNumber` | Swedish personal number | |
| `@ValidOrganizationNumber` | Swedish org number | |
| `@ValidMSISDN` | International mobile number | |
| `@ValidMobileNumber` | Mobile number | |
| `@ValidBase64` | Base64-encoded string | |
| `@ValidNamespace` | Namespace format | |
| `@OneOf({"VALUE_A", "VALUE_B"})` | Must be one of the given literals | `nullable = true` to allow null |
| `@MemberOf(MyEnum.class)` | Must match one of the enum's names | |

**Key rule:** Never use actual enum types in API model fields. Use `String` + `@OneOf` or `@MemberOf` instead.

## Simple String Field Validator

Use when you need to validate the format of a single `String` field.

### File structure

```
api/
└── validation/
    ├── ValidMyField.java
    └── impl/
        └── ValidMyFieldConstraintValidator.java
```

### Annotation (`api/validation/ValidMyField.java`)

```java
package se.sundsvall.{servicename}.api.validation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import se.sundsvall.{servicename}.api.validation.impl.ValidMyFieldConstraintValidator;

import static java.lang.annotation.ElementType.CONSTRUCTOR;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.TYPE_USE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Documented
@Target({CONSTRUCTOR, FIELD, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Constraint(validatedBy = ValidMyFieldConstraintValidator.class)
public @interface ValidMyField {
    String message() default "must be a valid ...";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

### Implementation (`api/validation/impl/ValidMyFieldConstraintValidator.java`)

```java
public class ValidMyFieldConstraintValidator implements ConstraintValidator<ValidMyField, String> {

    @Override
    public boolean isValid(final String value, final ConstraintValidatorContext context) {
        if (value == null) {
            return true; // Let @NotNull handle null separately
        }
        return /* your validation logic */;
    }
}
```

Rules:
- `isValid` returns `true` for `null` — compose with `@NotNull` if null must be rejected
- No state mutation in `isValid` — validators may be reused
- No Spring context needed
