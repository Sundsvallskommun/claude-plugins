# Sundsvalls kommun — Claude Code plugin marketplace

Claude Code plugin marketplace for Sundsvalls kommun developers. Hosts the `dept44` plugin (conventions, patterns, scaffolding) and the opt-in `greve` plugin (fleet catalogue MCP + review robot); more plugins can be added over time.

## Installation

From within Claude Code, add the marketplace once:

```
/plugin marketplace add Sundsvallskommun/claude-plugins
```

Then install the plugins you want:

```
/plugin install dept44@sundsvall-claude-plugins
/plugin install greve@sundsvall-claude-plugins   # optional, needs the greve binary — see below
```

Run `/reload-plugins` if needed.

## Plugins

### `dept44`

Coding conventions, pattern references, and scaffolding commands for dept44 microservices.

- **SessionStart hook** — automatically injects dept44 coding conventions into every Claude Code session
- **14 slash commands** — pattern references (`/dept44:pattern-resource`, `/dept44:pattern-service`, etc.) and scaffolding (`/dept44:new-entity`, `/dept44:new-endpoint-crud`, etc.). The `pattern-*` and `new-*` commands are thin wrappers — each one points at a reference file under `skills/` so there's a single source of truth.
- **5 skills** — contextual knowledge that Claude activates automatically: `dept44-patterns` (layer patterns + tests), `dept44-scaffold` (generating new components), `dept44-source` (framework internals lookup), `pattern-validator` (field validation), `pr-template` (the org-level PR/issue template and how to fill it in)

The dept44 plugin's skills, pattern references, and conventions were created by [Linus Sjölinder](https://github.com/Cheezi747).

| Command | Description |
|---|---|
| `/dept44:pattern-resource` | REST controller (Resource) pattern |
| `/dept44:pattern-pojo` | API model / DTO pattern |
| `/dept44:pattern-entity` | JPA entity + repository pattern |
| `/dept44:pattern-service` | Service layer pattern |
| `/dept44:pattern-mapper` | Mapper (static utility) pattern |
| `/dept44:pattern-integration` | Feign client integration pattern |
| `/dept44:pattern-scheduler` | Scheduled job pattern |
| `/dept44:pattern-apptest` | Integration / AppTest pattern |
| `/dept44:new-entity` | Scaffold new JPA entity |
| `/dept44:new-endpoint-crud` | Scaffold new CRUD endpoint |
| `/dept44:new-endpoint-proxy` | Scaffold new proxy endpoint |
| `/dept44:new-integration` | Scaffold new Feign integration |
| `/dept44:new-scheduler` | Scaffold new scheduler job |
| `/dept44:new-apptest` | Scaffold integration tests |

#### Disable per project

To disable the plugin for a non-dept44 project, add this to that project's `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "dept44@sundsvall-claude-plugins": false
  }
}
```

### `greve`

Opt-in wrapper around [greve](https://github.com/CheeziCrew/greve), a CLI + MCP server that catalogues the locally cloned dept44 services (endpoints, schemas, call graph, config, DB schema, dependency versions) and ships a deterministic convention linter. Pairs with `dept44` — the review command points at `/dept44:pattern-*` for fixes.

- **MCP server `greve`** — 26 tools (`context_pack`, `search_endpoints`, `impact_analysis`, `service_graph`, `dependency_versions`, `review_diff`, …) over your local clones
- **`/greve:review [service] [--changed --base <ref>] [--fix]`** — one-pass review: greve linter + convention rulebook + SonarCloud (if the `sonarqube` MCP is connected) + judgment; `--fix` applies safe mechanical fixes, never commits
- **Skill `greve-usage`** — which tool answers which question, and how much to trust catalog facts vs heuristic flags

**Prerequisite:** `go install github.com/CheeziCrew/greve@latest` with `~/go/bin` on `PATH`, and the repos cloned side by side (default root `~/Code/scit`; override with `GREVE_ROOT` or `~/.config/greve/config.yml`). Details in [greve/README.md](greve/README.md).

## Development

To test local changes before pushing:

```bash
claude --plugin-dir ./dept44
claude --plugin-dir ./greve
```

Run `/reload-plugins` after making changes to pick them up without restarting.

## Structure

```
.claude-plugin/
  marketplace.json          # Marketplace manifest (lists all plugins)
dept44/                     # The dept44 plugin
  .claude-plugin/
    plugin.json             # Plugin manifest
  commands/                 # Slash commands — thin wrappers over the skill reference files
  skills/                   # Contextual skills (auto-triggered by Claude)
    dept44-patterns/        # references/pattern-*.md — canonical layer patterns + tests
    dept44-scaffold/        # references/new-*.md — canonical scaffolding instructions
    dept44-source/          # references/*.md — framework internals lookup
    pattern-validator/      # references/*.md — field validation
    pr-template/            # org-level PR/issue template: where it is, how to fill it in
  hooks/
    hooks.json              # SessionStart hook config
    conventions.md          # dept44 coding conventions (injected on session start)
greve/                      # The greve plugin (opt-in, needs the greve binary)
  .claude-plugin/
    plugin.json
  .mcp.json                 # starts `greve mcp` as an MCP server
  commands/review.md        # /greve:review
  skills/greve-usage/       # when/how to use the greve tools
  README.md                 # prerequisites
```

## Adding a new plugin to this marketplace

1. Create a new top-level directory next to `dept44/` with the plugin's contents (must contain `.claude-plugin/plugin.json`).
2. Add an entry to `.claude-plugin/marketplace.json` under `plugins`:
   ```json
   {
     "name": "my-plugin",
     "description": "...",
     "source": "./my-plugin"
   }
   ```
3. Commit and push. Colleagues who already added this marketplace can install it with `/plugin install my-plugin@sundsvall-claude-plugins`.
