---
description: "Scaffold a new dept44 JPA entity + repository + Flyway migration (no endpoint)"
argument-hint: "<EntityName> with field:Type, field:Type, ..."
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-scaffold/references/new-entity.md` and follow it step by step to scaffold the entity described below — the `{Name}Entity`, the repository (with mandatory `@CircuitBreaker`), the Flyway migration, the entity bean test, and any integration-test seed data. Examine existing entities in the repo first to match exact column types and relationship styles.

What to scaffold:
$ARGUMENTS
