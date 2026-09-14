---
name: greve-usage
description: "How to use the greve MCP tools (mcp__greve__*) when working across the Sundsvall dept44 fleet. Trigger whenever the user asks which service calls or is called by another, what endpoints or schemas a service exposes, the impact of changing an endpoint or payload, which services are on a given dept44/dependency version, where a config key is used, what a service's DB schema or scheduled jobs look like, or wants a service reviewed against team conventions. Also trigger before writing code that talks to a sibling service — greve's catalogue is the fastest way to ground it. Prefer these tools over grepping ~10 repos by hand."
---

# greve — fleet code intelligence

greve is a catalogue over every locally cloned Sundsvall service (OpenAPI specs, `application*.yml`,
Flyway-derived DB schemas, Feign integrations in both directions, call graph, dependency/parent
versions, owners). Reach for it first at every phase, instead of raw `grep`/`find` across repos.

## Which tool for which question

| Phase | Tools |
|---|---|
| **Orient** on a service | `context_pack` (compact card), `get_service`, `list_services` |
| **Find** things | `search_endpoints` / `endpoint_schema`, `search_config` / `config_surface`, `db_schema`, `scheduler_jobs`, `service_graph` / `path_between`, `dependency_versions`, `git_activity` |
| **Ground new code** | `usage_examples` (how others call this endpoint), `pattern_examples` (canonical component examples in the fleet) |
| **Verify a change** | `impact_analysis` (who breaks), `integration_consistency`, `stale_clients`, `resilience_report`, `fleet_report` |
| **Review** | `review_diff` (deterministic convention linter), `convention_rules`, `standards_for_file` — or the `/greve:review` command for the full pass |
| After pulling / switching branches | `refresh_catalog` |

## How much to trust the output

- **Catalog facts** — endpoints, integrations, versions, call graph, db schema, config keys/values,
  owners — are **reliable grounding. Act on them directly.**
- **Heuristic flags** — `integration_consistency`, `test_coverage`, pattern reports, review
  *warnings* — are **leads, not verdicts. Confirm against source before they drive an action.**
  A flag may be a tool gap or genuinely true. Review *errors* are deterministic and count as blockers.

## Caveats

- greve only indexes **locally cloned** repos and reads the **working tree**: a service's catalogued
  version reflects the local clone, not necessarily the org's latest. Integration names that resolve to
  no local repo show up in `greve unresolved`.
- A running MCP server rescans when the catalogue is older than five minutes; after a greve upgrade or
  a scanner fix, reconnect the server so you are talking to the new build.
- When greve misses something real, prefer fixing greve's scanner (issue/PR on the greve repo) over
  memorising the exception.
