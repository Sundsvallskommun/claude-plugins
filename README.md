# Sundsvalls kommun — Claude Code plugin marketplace

Claude Code plugin marketplace for Sundsvalls kommun developers. Currently hosts the `dept44` plugin; more plugins can be added over time.

## Installation

From within Claude Code, add the marketplace once:

```
/plugin marketplace add Sundsvallskommun/claude-plugins
```

Then install the plugins you want:

```
/plugin install dept44@sundsvall-claude-plugins
```

Run `/reload-plugins` if needed.

## Plugins

### `dept44`

Coding conventions, pattern references, and scaffolding commands for dept44 microservices.

- **SessionStart hook** — automatically injects dept44 coding conventions into every Claude Code session
- **14 slash commands** — pattern references (`/dept44:pattern-resource`, `/dept44:pattern-service`, etc.) and scaffolding (`/dept44:new-entity`, `/dept44:new-endpoint-crud`, etc.). The `pattern-*` and `new-*` commands are thin wrappers — each one points at a reference file under `skills/` so there's a single source of truth.
- **4 skills** — contextual knowledge that Claude activates automatically: `dept44-patterns` (layer patterns + tests), `dept44-scaffold` (generating new components), `dept44-source` (framework internals lookup), `pattern-validator` (field validation)

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

## Development

To test local changes before pushing:

```bash
claude --plugin-dir ./dept44
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
  hooks/
    hooks.json              # SessionStart hook config
    conventions.md          # dept44 coding conventions (injected on session start)
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
