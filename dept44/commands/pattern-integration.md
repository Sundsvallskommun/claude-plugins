---
description: "dept44 Feign client integration pattern + the three integration test patterns"
argument-hint: "[optional: integration or notes]"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-patterns/references/pattern-integration.md` — the canonical dept44 external-service integration pattern (Feign `{Service}Client` with `@CircuitBreaker`, `{Service}Configuration` with `FeignMultiCustomizer`, `{Service}Properties` record, optional `{Service}Integration` wrapper) and its three test classes (`IntegrationTest`, `ConfigurationTest`, `PropertiesTest`) — and follow it exactly when writing, reviewing, or fixing `integration/` clients. Look at existing integrations in the repo for exact style.

$ARGUMENTS
