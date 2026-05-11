---
name: dept44-patterns
description: "Reference patterns for dept44 Spring Boot microservices. Use this skill whenever the user asks about dept44 coding conventions, needs to understand how a specific layer works (entity, repository, service, resource, mapper, POJO, scheduler, integration test), wants to write or fix tests following dept44 style, or asks 'how do we do X in dept44'. Triggers on: 'dept44 pattern', 'how should I write', 'test pattern', 'what does the entity look like', 'service layer pattern', 'mapper pattern', 'resource test', 'apptest', 'integration test pattern', 'POJO style', 'bean test', reviewing or fixing code in a dept44 repo, or any question about dept44 coding conventions and test patterns. Also trigger when the user is working in a dept44 codebase and asks about the correct way to structure something."
---

# dept44 Pattern Reference

Quick-reference patterns for every layer in a dept44 Spring Boot microservice. Read the relevant reference file(s) based on what the user is working on.

## Pattern Index

| Layer / Concern | Reference file | When to read |
|---|---|---|
| API Models (POJOs) | `references/pattern-pojo.md` | Writing or testing `api/model/` classes |
| JPA Entities | `references/pattern-entity.md` | Writing or testing `integration/db/model/` classes and repositories |
| Feign Integrations | `references/pattern-integration.md` | Writing or testing external service clients in `integration/` |
| Mappers | `references/pattern-mapper.md` | Writing or testing `service/mapper/` classes |
| Services | `references/pattern-service.md` | Writing or testing `service/` classes |
| Resources (Controllers) | `references/pattern-resource.md` | Writing or testing `api/` REST controllers |
| Schedulers | `references/pattern-scheduler.md` | Writing or testing `service/scheduler/` jobs |
| AppTests (E2E) | `references/pattern-apptest.md` | Writing integration tests in `src/integration-test/` |

## Usage

Most tasks only need 1-2 reference files. Read the ones relevant to the user's question — no need to load everything.

If the user is writing a new component from scratch (not just asking about patterns), consider whether the **dept44-scaffold** skill would be more appropriate — it has full step-by-step scaffolding instructions.
