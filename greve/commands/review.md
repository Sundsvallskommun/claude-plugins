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

### What has to be available
Check before step 1, and state in the report which layers ran:
- **The `greve` binary** — `command -v greve`. This command is built on it; if it is missing, stop and say so (install steps in the plugin README) rather than improvising a review without the deterministic layer. Prefer the CLI; the same data is available via the greve MCP tools `review_diff`, `convention_rules`, `standards_for_file` when that server is connected.
- **The `dept44` plugin** — steps 6 and 9 point at its `/dept44:pattern-*` references. If they are not in the session, ground fixes in the matching rule from `greve standards <category> --json` (its `good_example`, where it has one) and a real in-org example from `greve patterns <type>` instead, and say that the pattern references were unavailable.
- **The `sonarqube` MCP** — needed for step 3. If it is not connected, say so and skip that layer.

## Procedure

### 1. Scope the change
Resolve the service name and base ref. If `--changed`, get **every** changed file — not just Java:
`git -C <repo> diff --name-only <base>...HEAD` plus working-tree changes.

**Escalate on the raw file count.** If `git diff --name-only` returns more than ~40 paths, offer the multi-agent Workflow fan-out (one reviewer per module/dimension + adversarial verification) — count the paths git prints, never your own estimate of the "real" diff after discounting generated files. Talking yourself out of the offer because most of the diff "is just generated" is the exact reasoning that suppresses it when it is most needed.

**No changed file is dismissed unexamined on the grounds that a tool produced it** — but a large generated file does not need reading end to end. For checked-in specs, fixtures and other generated artifacts, check what they declare about the outside world with a targeted pass:
- **Modified file:** read only its hunks (`git -C <repo> diff <base>...HEAD -- <file>`). Stable sections like `servers:` rarely change, so a hunk touching them is worth a close look.
- **New file** (e.g. a provider's OpenAPI spec vendored into `src/main/resources/integrations/`): the whole file is new, so grep it instead of reading it — `grep -nEi 'https?://|servers:|host:|basePath|securitySchemes|tokenUrl|password|secret|api[-_]?key|token' <file>` — and read the hits in context. A spec copied from a provider's live `/api-docs` carries that environment's `servers:` block, which is exactly how a production or internal hostname reaches a public repo.
- For the rest of a spec, check only what the change relies on: the schemas and paths the new client actually calls.

List in the final report every file you only checked with a targeted pass, and how. An unexamined file is a stated limitation of the review, not a silent one.

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

### 5. Sibling diff (the step that finds the most)
For **each new component in the change**, find the nearest existing sibling of the same kind in the same repo and compare the *approach*, not just the conventions:
- `greve example <provider>` — a real Feign client + config from an existing consumer of that service.
- `greve patterns <type> --json` — in-org examples of a scheduler, feign client, validator, apptest, mapper, resource or entity.
- Plus the obvious local one: the other repository, the other integration's `Configuration`, the other mapper reading the same upstream response.

Ask one question of each pair: **does the existing code already answer this differently, and if so, which one is wrong?** A new Feign config with a bare `JsonPathSetup("$.message", …)` beside an existing one using `concat($[?(@.X != null)].X)`; a new repository that does not redeclare inherited `save` beside a sibling that does and carries a javadoc explaining why; a new mapper calling `.findFirst()` on a POB response that every other mapper filters by type first — those are the highest-confidence findings available, because the codebase has already decided and one side is out of step.

A diff-scoped read cannot see any of this: the correct answer lives in a file the diff does not touch. Budget real time here.

### 6. Judgment layer (you, over the diff)
Read the actual changed code (`git -C <repo> diff <base>...HEAD` or the files). Review against the **non-deterministic** corpus rules (`deterministic: false`) and the dept44 patterns (`/dept44:pattern-*`) — the things a linter can't see:
- Resources thin / no business logic; service-layer placement correct.
- Error handling: `Problem.valueOf(...)`, batch lookups name the *missing* items, message constants with `%s`.
- `Optional` instead of null-checks/ternaries; method references over lambdas; `final` where possible; unused lambda params as `_`.
- POJO shape (create()/with*()/manual equals-hashCode-toString), `@Schema(allowableValues=...)` for string-enum fields, `@DateTimeFormat` on OffsetDateTime.
- Entity column lengths/`@TimeZoneStorage`/`@UuidGenerator`; mapper null-safety and naming.
- Test completeness beyond the FailureTest rule: service tests mock+verify all deps, AppTest structure, bean tests, **85% line / 50% branch per class** plausibility.
- Sensitive data: no logging of personal data (personnummer, income, case contents).
- OpenAPI kept in sync with the Resource; no enums leaking into the API model.

**Treat code comments as claims under review, not as conclusions.** Well-commented code raises trust and should not lower scrutiny — the more persuasive the comment, the more worth checking. When a comment says *"deliberately not X"*, *"safe because Y"*, or *"the worst case is Z"*, go and establish what happens at X, whether Y holds, and do the arithmetic on Z yourself. Two comments in the same change that argue opposite policies for the same kind of decision (one property validated at startup, its sibling deliberately not) is a finding in itself, and only visible if you are reading them adversarially.

**For anything with a queue, scheduler, retry or lock: reason about state over time, not just control flow.** Tracing each method is not enough — these defects only appear across runs. Ask explicitly:
- What does the table look like after N runs with a dependency down? What accumulates at the head of the page?
- Does any row leave the queue by a path other than success? Is there a way *out* of the terminal state, or does a health indicator latch forever once the first row reaches it?
- Multiply it out: `page-size × read-timeout` against `lockAtMostFor`. If a run can outlive its lock, what gets sent twice?
- Is there a claim step (`@Version`, conditional update, claimed-by column) before the side effect, or is at-most-once resting entirely on the lock holding?

Be specific and cite `file:line`. Prefer confirmed issues over speculation; mark genuinely uncertain ones as such.

### 7. Build and coverage (always, not just with `--fix`)
Run `mvn -B verify` in the service (add `-o` only if you know the local repository is already warm — a dependency bump on the branch will not resolve offline, and a spurious build failure is worse than a slow one). A review that never compiled the branch cannot say whether any finding is currently breaking anything, and the answer changes how the whole list should be read.

Then read the branch-level coverage rather than trusting the gate: parse `target/jacoco-merge-report/jacoco.xml` — the merged unit + integration data, which is what the gate checks; `jacoco-ut-report` alone under-reports anything covered only by AppTests — for the classes the change touched and look for missed branches (`mb`) on the new code. The gate is per class at 85% line and only 50% branch, so it passes comfortably while a specific method is untested, and this is where you find the branches that are not merely uncovered but *unreachable at the configured values* — a guarded helper whose other formats can never run at the threshold actually set in `application.yml`.

### 8. Report
Group by severity, then file. For each finding: `file:line`, a one-line description, the `rule_id`/`corpus_id` or Sonar key, and a concrete suggested fix (show the corrected snippet for non-trivial ones). State the build result from step 7. List the files you only checked with a targeted pass, per step 1. End with a short verdict: blockers (must fix before merge) vs. advisories, and the merge-readiness call.

### 9. Fix (only if `--fix` given)
Apply the safe, mechanical fixes to the working tree (ternary→if/else or Optional, add missing `@CircuitBreaker`, static-import constants, drop redundant `@PathVariable` names, add a `{Resource}FailureTest` skeleton, etc.). For each fix, follow the corresponding `/dept44:pattern-*` reference exactly. After fixing, re-run `greve review` to confirm the errors are cleared and re-run the step 7 build — a fix you have not compiled is a guess. Never push or commit — leave that to the user.

## Notes
- This is decision support: surface everything, recommend, but the human merges.
- The deterministic + Sonar layers are exhaustive within their scope; your judgment layer is where review hours actually go — spend the effort there.
- **A clean deterministic layer is not evidence of a clean change.** greve and Sonar coming back quiet, a green build and high coverage constrain only what they measure. Careful, well-commented code does not have fewer defects than sloppy code — it has subtler ones, concentrated exactly where a careful author could not see them from inside. Read visible quality as a reason to look harder, never as a reason to stop early.
- For an exhaustive multi-agent pass on a big PR, escalate to a Workflow: fan out reviewers per module and per dimension (correctness / conventions / tests / security), adversarially verify each finding, then synthesize and optionally `--fix`.
