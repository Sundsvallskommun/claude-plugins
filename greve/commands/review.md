---
description: "One-pass senior review of a dept44 service: greve linter + convention rulebook + SonarCloud + judgment"
argument-hint: "[service] [--changed] [--base <ref>] [--fix]"
---

# /greve:review — dept44 review robot

Review a dept44 service's code the way a senior reviewer would, but in one pass and without the hours. Combine greve's deterministic linter, the standards corpus, live SonarCloud findings, and your own judgment over the diff, then report categorized findings with concrete fixes — and apply them on request.

## Arguments
`$ARGUMENTS` — the target and options. Examples:
- `api-service-document` — review the whole working tree of that repo
- `api-service-document --changed` — only what changed vs `main`
- `api-service-document --changed --base develop`
- `--fix` anywhere in the args — after reporting, apply the safe fixes to the working tree
- no service given — use the repo in the current working directory

`greve` must be on `PATH` (see the plugin README). Prefer the `greve` CLI (deterministic, always present); the same data is available via the greve MCP tools `review_diff`, `convention_rules`, `standards_for_file` when that server is connected. The SonarCloud layer needs the `sonarqube` MCP — if it is not connected, say so and skip that layer.

## Procedure

### 1. Scope the change
Resolve the service name and base ref. If `--changed`, get the changed Java files:
`git -C <repo> diff --name-only <base>...HEAD -- '*.java'` plus working-tree changes. Note the count — if it's large (say >40 files), tell the user you can fan this out as a multi-agent Workflow (one reviewer per module/dimension + adversarial verification) and offer that; otherwise proceed in one pass.

### 2. Deterministic layer (greve review)
Run `greve review <service> [--changed --base <base>] --json`. These are high-precision, zero-judgment violations (banned ternaries/imports/Lombok, missing `@CircuitBreaker`, missing `{Resource}FailureTest`, `@Enumerated` not STRING, field injection, enums in api/model, …). Every error-severity finding is a merge blocker. Keep each finding's `corpus_id` — it links to the rule's rationale and good example.

### 3. Static-analysis layer (SonarCloud)
SonarCloud is **reactive**: it only has results for code that has been pushed and analysed by its CI — it does NOT scan the local working tree. So you must query the right *scope*, or you'll miss everything.
- **If reviewing a PR (the common case): query PR-scoped, not project-scoped.** Find the PR number (`gh pr list --repo Sundsvallskommun/<service> --head <branch> --json number`), then:
  - `search_sonar_issues_in_projects(projects=["Sundsvallskommun_<service>"], pullRequestId="<n>", issueStatuses=["OPEN"])` — the PR's new-code bugs/vulns/smells. **This is the step that catches the per-PR findings; querying without `pullRequestId` returns main-branch issues and silently misses them.**
  - `get_project_quality_gate_status(projectKey="Sundsvallskommun_<service>", pullRequest="<n>")` and `search_security_hotspots(... pullRequestId="<n>")`.
- If reviewing a plain branch with no PR, there may be no SonarCloud analysis at all yet — say so explicitly.
- **Proactive/local Sonar is only partial.** `analyze_code_snippet(projectKey, fileContent, language, scope)` runs analysers on un-pushed content, but single-file (no classpath) so it MISSES rules needing type/library resolution (e.g. the AssertJ `java:S5853` "join assertions" rule). Use it as a best-effort supplement for changed files, never as the authority. True local parity needs the full scanner (`mvn sonar:sonar`).
- Always state which scope you queried and whether the new code is actually Sonar-analysed yet — a clean result on the wrong scope is the failure mode.

### 4. Convention context (corpus + greve facts)
For the layers present in the diff, pull the rulebook so your judgment pass is grounded in the team's actual conventions, not generic Java advice:
- `greve standards --file <path> --json` (or the `standards_for_file` MCP tool) per distinct component type in the diff.
- `greve standards <category> --json` for the relevant categories (resource, service, entity, integration, …).
Also consider blast radius and drift: `greve impact <service> …` before any endpoint/schema change, and `greve consistency <service>` for integration drift.

### 5. Judgment layer (you, over the diff)
Read the actual changed code (`git -C <repo> diff <base>...HEAD` or the files). Review against the **non-deterministic** corpus rules (`deterministic: false`) and the dept44 patterns (`/dept44:pattern-*`) — the things a linter can't see:
- Resources thin / no business logic; service-layer placement correct.
- Error handling: `Problem.valueOf(...)`, batch lookups name the *missing* items, message constants with `%s`.
- `Optional` instead of null-checks/ternaries; method references over lambdas; `final` where possible; unused lambda params as `_`.
- POJO shape (create()/with*()/manual equals-hashCode-toString), `@Schema(allowableValues=...)` for string-enum fields, `@DateTimeFormat` on OffsetDateTime.
- Entity column lengths/`@TimeZoneStorage`/`@UuidGenerator`; mapper null-safety and naming.
- Test completeness beyond the FailureTest rule: service tests mock+verify all deps, AppTest structure, bean tests, **85% line+branch** plausibility.
- Sensitive data: no logging of personal data (personnummer, income, case contents).
- OpenAPI kept in sync with the Resource; no enums leaking into the API model.
Be specific and cite `file:line`. Prefer confirmed issues over speculation; mark genuinely uncertain ones as such.

### 6. Report
Group by severity, then file. For each finding: `file:line`, a one-line description, the `rule_id`/`corpus_id` or Sonar key, and a concrete suggested fix (show the corrected snippet for non-trivial ones). End with a short verdict: blockers (must fix before merge) vs. advisories, and the merge-readiness call.

### 7. Fix (only if `--fix` given)
Apply the safe, mechanical fixes to the working tree (ternary→if/else or Optional, add missing `@CircuitBreaker`, static-import constants, drop redundant `@PathVariable` names, add a `{Resource}FailureTest` skeleton, etc.). For each fix, follow the corresponding `/dept44:pattern-*` reference exactly. After fixing, re-run `greve review` to confirm errors are cleared and run `mvn -q verify` (or at least `mvn -q test-compile`) if the user wants build confirmation. Never push or commit — leave that to the user.

## Notes
- This is decision support: surface everything, recommend, but the human merges.
- The deterministic + Sonar layers are exhaustive within their scope; your judgment layer is where review hours actually go — spend the effort there.
- For an exhaustive multi-agent pass on a big PR, escalate to a Workflow: fan out reviewers per module and per dimension (correctness / conventions / tests / security), adversarially verify each finding, then synthesize and optionally `--fix`.
