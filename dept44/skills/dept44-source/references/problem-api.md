# Problem API — Error Throwing

Source: `<repos>/dept44/dept44-starter/src/main/java/se/sundsvall/dept44/problem/Problem.java`

## When to use

`Problem` is the standard way to throw HTTP errors in dept44 services. It wraps errors as RFC 9457 problem detail responses. Use it in service and integration layers — never in Resource classes (those should be thin and delegate to services).

## Factory Methods (most common)

```java
throw Problem.valueOf(NOT_FOUND, "Entity with id '%s' not found".formatted(id));
throw Problem.valueOf(BAD_REQUEST, "Invalid input: %s".formatted(detail));
throw Problem.valueOf(INTERNAL_SERVER_ERROR, "Unexpected error");
throw Problem.valueOf(BAD_GATEWAY, "Downstream service failed");
throw Problem.valueOf(CONFLICT, "Entity already exists");
```

Always use **static imports** for HTTP status codes:
```java
import static org.springframework.http.HttpStatus.NOT_FOUND;
```

`Problem.valueOf` returns `ThrowableProblem` which is a `RuntimeException` — throw it directly.

## Builder (advanced — wrapping a cause)

```java
throw Problem.builder()
    .withStatus(NOT_FOUND)
    .withTitle("Not Found")
    .withDetail("Entity '%s' not found".formatted(id))
    .withCause(originalException)
    .build();
```

## Status Mapping Guide

| Situation | Status |
|---|---|
| Resource not found by ID | `NOT_FOUND` |
| Invalid input / business rule | `BAD_REQUEST` |
| Duplicate / already exists | `CONFLICT` |
| Downstream service failure | `BAD_GATEWAY` |
| Unexpected internal failure | `INTERNAL_SERVER_ERROR` |
