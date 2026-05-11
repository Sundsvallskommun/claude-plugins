---
name: dept44-source
description: "Quick-reference for dept44 framework internals — class locations, APIs, and cross-cutting patterns. Use this skill whenever you need to find where a dept44 class lives (AbstractAppTest, FeignMultiCustomizer, Problem, validators), understand how to throw errors with Problem.valueOf(), look up AbstractAppTest methods for writing AppTests, build JPA Specification filters, add pagination to an endpoint, or handle file uploads with MultipartFile. Also use when encountering unknown dept44 annotations or utilities, or before reaching for Context7/WebSearch — a locally cloned copy of the dept44 source is the authoritative reference. If you're working in any dept44 microservice and need to understand framework internals, check here first."
---

# dept44 Source Lookup

The dept44 framework source is usually cloned locally — read it directly with `Read`/`Glob`/`Grep` rather than guessing, decompiling JARs, or searching the web.

**Never decompile JARs. Never use Context7 or WebSearch for dept44 internals if a local clone exists.**

## Locating the source (`<repos>`)

Throughout these reference files, `<repos>/` means the directory where the Sundsvall repos are cloned side by side — so `<repos>/dept44/` is the framework and `<repos>/api-service-document/` is a service. The layout varies per machine, so resolve it once at the start:

1. Look for a sibling of the repo you're currently in named `dept44/` that contains `dept44-starter-parent/` — that's the framework; its parent directory is `<repos>/`.
2. If not adjacent, search common roots: `~/repos`, `~/code`, `~/git`, `~/dev`, `~/src` (e.g. `find ~ -maxdepth 4 -type d -name dept44 -path '*/dept44'` then check for `dept44-starter-parent/`).
3. If you genuinely can't find a local clone, fall back to GitHub: `Sundsvallskommun/dept44` for the framework, `Sundsvallskommun/api-service-{name}` for services.

Once found, substitute the real path for `<repos>/` in every path below.

## Reference Files

Read the relevant file(s) based on what you need — most tasks only need one or two.

| Topic | Reference file | When to read |
|---|---|---|
| Module paths & key class index | `references/module-index.md` | Finding any dept44 class or source path |
| `AbstractAppTest` API | `references/abstract-app-test.md` | Writing or understanding AppTest integration tests |
| `Problem` / error throwing | `references/problem-api.md` | Throwing errors in service or integration layers |
| Specification / filtering | `references/specification-builder.md` | Building dynamic JPA query filters for repository methods |
| Pagination | `references/pagination.md` | Adding paginated list endpoints (the full Resource->Service->Repo->Response flow) |
| File uploads / MultipartFile | `references/file-upload.md` | Handling file uploads, converting data to MultipartFile, validating file content types |
