# greve plugin

Opt-in tooling around [greve](https://github.com/CheeziCrew/greve) — a catalogue of the Sundsvall
dept44 microservices served as a CLI and an MCP server from one binary. It scans the locally cloned
repos (no index, no daemon, sub-second) and answers: what does service X do, who calls whom, which
endpoint schema would break, who is still on which dept44 version — plus a deterministic
convention linter (`greve review`) used by the `/greve:review` command.

This plugin does **not** ship the binary. It wires the MCP server into Claude Code, adds the review
command, and a skill that teaches Claude when to reach for the tools. Pair it with the `dept44`
plugin — the review command refers to `/dept44:pattern-*` for the canonical fixes.

## Prerequisites

1. Go toolchain, then:
   ```sh
   go install github.com/CheeziCrew/greve@latest
   ```
   The MCP server is started as plain `greve mcp`, so the binary must be on your `PATH`: either add
   `$(go env GOPATH)/bin` (usually `~/go/bin`) to it, or symlink the binary into a directory already
   on your `PATH`, e.g. `ln -s ~/go/bin/greve ~/.local/bin/greve`.
2. The Sundsvall repos cloned side by side in one directory. greve defaults to `~/Code/scit`; if
   yours live elsewhere, set `GREVE_ROOT` or create `~/.config/greve/config.yml`:
   ```yaml
   root: ~/repos/sundsvall
   ```
3. Smoke test from a terminal: `greve services` should list your clones. Then
   `/plugin install greve@sundsvall-claude-plugins` and `/reload-plugins`.

## What you get

- **MCP server `greve`** — 26 tools mirroring the CLI: `context_pack`, `get_service`,
  `search_endpoints`, `endpoint_schema`, `service_graph`, `path_between`, `impact_analysis`,
  `stale_clients`, `integration_consistency`, `db_schema`, `config_surface`, `search_config`,
  `scheduler_jobs`, `resilience_report`, `dependency_versions`, `fleet_report`, `git_activity`,
  `usage_examples`, `pattern_examples`, `test_coverage`, `review_diff`, `convention_rules`,
  `standards_for_file`, `refresh_catalog`, … The catalogue rescans itself when older than five minutes.
- **`/greve:review [service] [--changed --base main] [--fix]`** — a one-pass senior review:
  greve's deterministic linter + the convention rulebook + SonarCloud findings (when the
  `sonarqube` MCP is connected) + Claude's judgment over the diff. Reports blockers vs advisories;
  `--fix` applies the safe mechanical fixes to the working tree. Never commits or pushes.
- **Skill `greve-usage`** — auto-activated guidance on which tool answers which question, and how
  much to trust each kind of output.

## Trust model

Two tiers of output, treated differently:

- **Catalog facts** (endpoints, integrations, versions, call graph, db schema, config keys, owners)
  are reliable grounding — act on them.
- **Heuristic flags** (`integration_consistency`, `test_coverage`, pattern reports, review
  warnings) are leads, not verdicts — confirm against source before they drive a change.

greve only sees what is cloned locally and reads the working tree, so a service's catalogued
version is your clone's, not necessarily the org's latest. `greve review` is a local gate only —
it never touches CI, the Maven build or GitHub.
