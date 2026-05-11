---
description: "dept44 AppTest / integration test pattern (AbstractAppTest, WireMock, resource layout)"
argument-hint: "[optional: feature or notes]"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-patterns/references/pattern-apptest.md` — the canonical dept44 end-to-end AppTest pattern (`{Feature}IT extends AbstractAppTest`, `@WireMockAppTestSuite`, `@Sql`, the `setupCall()...sendRequestAndVerifyResponse()` fluent chain, `test01_`/`test02_` ordering, and the `__files/` resource layout) — and follow it exactly when writing, reviewing, or fixing `src/integration-test/` tests. (For the deeper `AbstractAppTest` method reference, use the `dept44-source` skill.) Look at existing AppTests in the repo for exact style.

$ARGUMENTS
