---
description: "dept44 JPA entity + repository pattern + entity test pattern"
argument-hint: "[optional: entity class or notes]"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-patterns/references/pattern-entity.md` — the canonical dept44 JPA entity pattern (`@Entity`, `@UuidGenerator` `String id`, `@Column`, `@TimeZoneStorage`, `create()`/`with*()`), the repository pattern (mandatory `@CircuitBreaker`, derived queries, `@Lock`), and the entity bean test pattern — and follow it exactly when writing, reviewing, or fixing `integration/db/` classes. Look at existing entities and repositories in the repo for exact style.

$ARGUMENTS
