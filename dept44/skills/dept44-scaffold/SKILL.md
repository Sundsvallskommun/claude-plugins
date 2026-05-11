---
name: dept44-scaffold
description: "Scaffold new components in dept44 Spring Boot microservices. Use this skill whenever the user wants to create, generate, or scaffold a new CRUD endpoint, proxy endpoint, JPA entity, Feign integration, scheduled job, or integration tests / AppTests in a dept44 service. Triggers on: 'new endpoint', 'create entity', 'add integration', 'scaffold', 'new scheduler', 'add a feign client', 'proxy endpoint', 'create CRUD', 'new job', 'add app tests', 'write integration tests for this service', or any request to generate boilerplate for a Spring Boot microservice following dept44 conventions. Also trigger when the user mentions creating something 'like the other endpoints' or 'following the existing pattern' in a dept44 repo."
---

# dept44 Scaffolder

Generate new components in dept44 Spring Boot microservices. This skill routes to the appropriate scaffolding template based on what the user needs.

## How to use

1. Figure out which type of component the user needs (see table below)
2. Read the corresponding reference file for the full instructions
3. Follow those instructions to scaffold all required files

## Component Types

| User wants... | Read this reference |
|---|---|
| CRUD endpoint backed by a local database | `references/new-endpoint-crud.md` |
| Endpoint that proxies to an external service via Feign | `references/new-endpoint-proxy.md` |
| JPA entity + repository + Flyway migration (no endpoint) | `references/new-entity.md` |
| Feign client integration to an external service (no endpoint) | `references/new-integration.md` |
| Scheduled background job | `references/new-scheduler.md` |
| Integration tests (AppTests) + OpenAPI contract test for an existing service | `references/new-apptest.md` |

## Routing hints

- If the user mentions "database", "table", "CRUD", or "persist" → **new-endpoint-crud**
- If the user mentions "proxy", "forward", "external service", "feign" with an endpoint → **new-endpoint-proxy**
- If the user only needs the data layer (entity/repo/migration) without an API → **new-entity**
- If the user only needs to call another service without exposing an endpoint → **new-integration**
- If the user mentions "cron", "scheduled", "batch", "job", "cleanup", "periodic" → **new-scheduler**
- If the user wants "AppTests", "integration tests", "IT tests", or "OpenAPI contract test" for an existing service → **new-apptest**
- For just the *pattern* of a single component (not generating a whole new one), the `dept44-patterns` skill is a better fit

## Cross-cutting rules

These apply to all scaffolding regardless of type — they're the dept44 way:

- Always examine existing code in the repo first to match exact style
- `final` on all variables and parameters
- Static imports for enums and constants
- No Lombok, no wildcard imports
- `@CircuitBreaker` is mandatory on all repositories and Feign clients
- Tests are not optional — every component gets full test coverage
- After scaffolding, run `mvn test` then `mvn verify` to confirm everything passes
