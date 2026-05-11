---
description: "dept44 scheduled job pattern (@Dept44Scheduled + worker) + scheduler / Shedlock test patterns"
argument-hint: "[optional: scheduler or notes]"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-patterns/references/pattern-scheduler.md` — the canonical dept44 scheduler pattern (`@Dept44Scheduled` — never Spring's `@Scheduled` — config via `@Value`, `Dept44HealthUtility`, delegating work to a `{Job}Worker`, per-item error handling) and its unit test + Shedlock integration test patterns — and follow it exactly when writing, reviewing, or fixing `service/scheduler/` jobs. Look at existing schedulers in the repo for exact style.

$ARGUMENTS
