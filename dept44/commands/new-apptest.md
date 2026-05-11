---
description: "Scaffold dept44 integration tests (AppTests) + the OpenAPI contract test for an existing service"
argument-hint: "[optional: resource/feature name or notes]"
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dept44-scaffold/references/new-apptest.md` and follow it step by step to scaffold the AppTests and OpenAPI contract test described below — `{Feature}IT extends AbstractAppTest` classes, the `__files/` WireMock stub + response resource tree, the DB scripts (if the service uses a database), and `OpenApiSpecificationIT` with its baseline at `src/integration-test/resources/api/openapi.yaml`. Examine existing AppTests in the repo (and sibling repos) first to match exact style. When done, run `mvn verify`.

Notes / what to test:
$ARGUMENTS
