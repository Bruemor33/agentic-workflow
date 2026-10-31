---
name: setup-destinations
description: Decide where tickets, specs, glossary, ADRs and research notes get written for this project (Linear or local files; a local docs folder). Use when a skill needs a destination and none is configured, or when the user wants to change one.
---

# Setup Destinations

Other skills (`wayfinder`, `to-tickets`, `to-spec`, `domain-modeling`, `research`, `prototype`, `diagnosing-bugs`) never assume a repo issue tracker or write docs into the repo. They read a **destinations config** and write there. This keeps planning artifacts out of the project's own repo and ticketing system.

## The config

One file per project, outside the repo so it works from any git worktree: `~/.claude/destinations/<project>.md`. `<project>` is the basename of the main checkout, found with `basename "$(dirname "$(git rev-parse --path-format=absolute --git-common-dir)")"`.

```markdown
tracker: linear | local
linear_team: <team name or key>        # tracker: linear only
linear_project: <project name>         # tracker: linear only
docs_root: ~/agent-docs/<project>
```

`docs_root` holds everything document-shaped:

```
<docs_root>/
├── GLOSSARY.md            (or GLOSSARY-MAP.md + per-context glossaries)
├── adr/                   0001-slug.md, ...
├── specs/                 <slug>.md
├── research/              <topic>.md
└── tickets/<slug>/        local tracker only: <NN>-<slug>.md, one file per ticket
```

Create directories lazily, only when there is something to write.

## Resolving the destination (every skill does this first)

1. Read `~/.claude/destinations/<project>.md`.
2. If it is missing, run the setup below. Never fall back to the repo's own tracker, never write docs into the repo, and never touch Azure DevOps or any other tracker not named in the config.

## Setup

Ask the user, with a recommendation for each:

1. **Tickets**: `linear` (needs the Linear MCP connected; ask which team and which Linear project to use, and create the project if it doesn't exist) or `local` (no external system; the default if Linear isn't connected).
2. **Docs root**: default `~/agent-docs/<project>`. Create it.

Write the config file and tell the user where it is. Do not put work-project details anywhere other than what the user chose.

## Tracker operations

The map/ticket structure in `wayfinder` and `to-tickets` is identical on both trackers; only the mechanics differ.

### Linear

- **Map**: one Linear issue in the configured project, label `wayfinder:map`. **Tickets**: sub-issues of the map, each with its `wayfinder:<type>` label (create labels if missing).
- **Blocking**: Linear's native "blocked by" relation, set in the second wiring pass.
- **Claim**: assign the issue to the user. **Frontier**: open sub-issues with no open blockers and no assignee.
- **Resolve**: post a comment with the answer, then move the issue to a Done state.
- **to-spec**: a Linear issue in the project (or a Linear document), with the spec template as the body.
- Everything is created in `linear_project` of `linear_team`, nowhere else.

### Local

- **Map**: `<docs_root>/tickets/<slug>/00-map.md`, using the map body template. **Tickets**: `<NN>-<slug>.md` beside it, one per file, numbered in dependency order.
- Each ticket file has a header block: `Type:` (the `wayfinder:<type>`), `Parent:` (the map), `Blocked by:` (titles and filenames), `Claimed by:` (empty until claimed), `Status:` (`open` | `closed`).
- **Claim**: set `Claimed by:` first, before any work. **Frontier**: `Status: open`, empty `Claimed by:`, every blocker `Status: closed`.
- **Resolve**: append a `## Resolution` section with the answer, set `Status: closed`, and add the pointer line to the map's Decisions so far.
- **to-spec**: `<docs_root>/specs/<slug>.md`.
