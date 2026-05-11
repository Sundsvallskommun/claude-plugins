# Composite Validator (OR / AND Logic)

Use when you want to combine two or more **existing** validators without writing a new `ConstraintValidator` implementation.

## Pattern (from `api-service-byggr-integrator`)

Accept either a personal number OR an organisation number:

```java
package se.sundsvall.byggrintegrator.api.validation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;
import jakarta.validation.ReportAsSingleViolation;
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import org.hibernate.validator.constraints.CompositionType;
import org.hibernate.validator.constraints.ConstraintComposition;
import se.sundsvall.dept44.common.validators.annotation.ValidOrganizationNumber;
import se.sundsvall.dept44.common.validators.annotation.ValidPersonalNumber;

import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Documented
@ConstraintComposition(CompositionType.OR)   // field is valid if ANY composed constraint passes
@Constraint(validatedBy = {})                // no impl class needed
@ValidPersonalNumber
@ValidOrganizationNumber
@ReportAsSingleViolation                     // report as one message, not one per failed constraint
@Target({FIELD, PARAMETER})
@Retention(RUNTIME)
public @interface ValidPersonalOrOrgNumber {
    String message() default "Invalid personal or organization number";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

Key points:
- `@ConstraintComposition(CompositionType.OR)` — valid if ANY composed annotation passes
- `@ConstraintComposition(CompositionType.AND)` — valid only if ALL pass (default, so rarely needed explicitly)
- `@Constraint(validatedBy = {})` — empty implementation list, composed validators do the work
- `@ReportAsSingleViolation` — stops after first failure and reports only the outer annotation's `message()`; omit this if you want each composed constraint to report separately

## Usage

```java
@ValidPersonalOrOrgNumber
private String identifier;
```
