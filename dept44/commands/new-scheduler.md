---
description: "Scaffold a new dept44 scheduled job (@Dept44Scheduled scheduler + worker + config + tests)"
argument-hint: "<JobName> - <what it does>"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-scaffold/references/new-scheduler.md` and follow it step by step to scaffold the scheduled job described below — the `{JobName}Scheduler` (using `@Dept44Scheduled`, never Spring's `@Scheduled`), the `{JobName}Worker`, the `application.yml` / `application-junit.yml` entries, and both test classes. Per-item error handling is required — one failure must not stop the batch. Examine existing schedulers in the repo first to match exact style.

What to scaffold:
$ARGUMENTS
