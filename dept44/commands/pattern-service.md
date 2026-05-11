---
description: "dept44 service layer pattern + Mockito service test pattern"
argument-hint: "[optional: service class or notes]"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-patterns/references/pattern-service.md` — the canonical dept44 service pattern (`@Service`, constructor injection with `final` deps, `Problem.valueOf(...)` for errors, error messages as `%s`-format constants, `Optional` fluent chains, delegate mapping to mappers and external calls to integrations/repositories) and its `@ExtendWith(MockitoExtension.class)` test pattern — and follow it exactly when writing, reviewing, or fixing `service/` classes. Look at existing services in the repo for exact style.

$ARGUMENTS
