# Complex Object Validator (Multiple Violations)

Use when you need to validate an object with multiple fields, or when you need to emit multiple distinct violation messages — potentially attached to different property nodes.

## Pattern (from `api-service-billing-preprocessor`)

```java
public class ValidInvoiceRowsConstraintValidator implements ConstraintValidator<ValidInvoiceRows, BillingRecord> {

    private static final String ERROR_MISSING_VAT = "must contain vat code when billing record is of type EXTERNAL";
    private static final String ERROR_MISSING_ACCOUNT = "at least one invoice row must have accountInformation";
    private static final String VALIDATED_NODE = "invoice.invoiceRows";

    @Override
    public boolean isValid(final BillingRecord record, final ConstraintValidatorContext context) {
        var isValid = true;

        if (!vatCodesAreValid(record)) {
            addViolation(context, ERROR_MISSING_VAT);
            isValid = false;
        }
        if (!accountInfoIsPresent(record)) {
            addViolation(context, ERROR_MISSING_ACCOUNT);
            isValid = false;
        }

        return isValid;
    }

    // Attach violation to a specific property node path
    private void addViolation(final ConstraintValidatorContext context, final String message) {
        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate(message)
            .addPropertyNode(VALIDATED_NODE)
            .addConstraintViolation();
    }
}
```

Key points:
- The validator type parameter is the whole object (`BillingRecord`), not a field type — the annotation goes on the class or a field of that type
- Collect ALL violations before returning (don't short-circuit) so all messages are reported at once
- `disableDefaultConstraintViolation()` suppresses the annotation's `message()` default — always call this when using custom node placement
- `addPropertyNode(VALIDATED_NODE)` attaches the violation to a dot-notation path in the response (e.g. `"invoice.invoiceRows"`)

## Annotation for a class-level or field-of-complex-type constraint

```java
@Documented
@Target({FIELD, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Constraint(validatedBy = ValidInvoiceRowsConstraintValidator.class)
public @interface ValidInvoiceRows {
    String message() default "one or more invoice rows contains invalid data";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

Apply on the field:
```java
@ValidInvoiceRows
private BillingRecord billingRecord;
```
