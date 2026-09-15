---
name: pr-template
description: "How to write pull requests and issues for Sundsvallskommun repos (api-service-*, dept44, pw-*, …). Use whenever you are about to run `gh pr create` or `gh issue create`, draft a PR description or issue body, or are asked to open, prepare or fill in a PR for one of these repos. Also trigger when a per-repo search finds no PR template — the template is org-level and lives in Sundsvallskommun/.github, not in the service repo."
---

# Pull requests and issues — the org template

## Language: English, always

**Everything that lands on GitHub is written in English** — PR titles and bodies, commit messages,
issue titles and bodies, review comments, code comments and any documentation committed to the repo.
This holds however the work was discussed: a Swedish Slack thread, a Swedish ticket, Swedish domain
terms (mantal, sjöman, juridisk person, arkivbildare) or a Swedish conversation with the author are
not a reason to write the PR in Swedish. Keep the domain terms themselves when they name a table, a
column or a concept that has no English equivalent — describe them in English rather than translating
them away.

Artefacts that never reach GitHub are outside this rule and keep their own language: Jira tickets and
the Swedish backlog documents that live only on a developer's machine.

## Where the template is

There is **no per-repo PR template** in the `api-service-*` repos, `dept44`, or the `pw-*` wrappers.
They all inherit the org-level one from the repo **`Sundsvallskommun/.github`**:

- PR: `.github/PULL_REQUEST_TEMPLATE.md`
- Issues: `.github/ISSUE_TEMPLATE/bug_report.md` and `.github/ISSUE_TEMPLATE/feature_request.md`

Searching only the target repo will wrongly conclude that no template exists. Get the current one from:

1. A local clone of `Sundsvallskommun/.github` next to the service repos (`<repos>/.github/.github/PULL_REQUEST_TEMPLATE.md`), or
2. GitHub: `gh api repos/Sundsvallskommun/.github/contents/.github/PULL_REQUEST_TEMPLATE.md --jq .content | base64 -d`

Always read it fresh — it is the source of truth and can change.

## Filling in the PR template

The template has three sections: **Types of changes** (checkbox list), **Does this PR introduce a
breaking change?** (Yes/No) and **Checklist:** (three items).

- Keep all three headings, every checkbox line and the HTML comments, in the template's order.
- Tick (`[x]`) the boxes that apply. Leave the rest unticked — an unticked "tests added (if applicable)" with a
  one-line reason in the description is better than a false tick.
- Put your narrative in a `## Description` section **above** the template's first heading. Say what changed,
  why, and how it was verified. If a behaviour change is arguable, say so explicitly so a reviewer can disagree.
- Answer the breaking-change question honestly. "Yes" means the version was stepped accordingly.
- Write the body to a file and pass it with `gh pr create --body-file <file>` rather than a long `--body` string.

## Issues

Use the matching issue template (`bug_report.md` for defects, `feature_request.md` for enhancements) and keep its
bold section headings. For a bug, the reproduction steps should be concrete HTTP calls (method, path, the
parameter that triggers it), and logs go under **Logs**, not inline in the description.
