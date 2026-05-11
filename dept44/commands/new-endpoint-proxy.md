---
description: "Scaffold a new dept44 endpoint that proxies to an external service via a Feign integration (no local DB)"
argument-hint: "<DomainName> via <targetService>"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-scaffold/references/new-endpoint-proxy.md` and follow it step by step to scaffold the proxy endpoint described below — reusing the target Feign integration if it already exists, creating it (per the new-integration steps) if not, and including all test classes. Examine existing code in the repo first to match its exact style. When done, run `mvn test` then `mvn verify`.

What to scaffold:
$ARGUMENTS
