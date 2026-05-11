---
description: "dept44 mapper (static utility) pattern + parameterized mapper test pattern"
argument-hint: "[optional: mapper class or notes]"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-patterns/references/pattern-mapper.md` — the canonical dept44 mapper pattern (private-constructor utility class, all `static`, null-safe with `ofNullable()`, `to{Target}` / `to{Target}Entity` / `to{Target}List` / `updateEntity` naming) and its `@ParameterizedTest` / `@MethodSource` test pattern — and follow it exactly when writing, reviewing, or fixing `service/mapper/` classes. Look at existing mappers in the repo for exact style.

$ARGUMENTS
