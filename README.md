# _global

Meta-architecture and system prompting for a multi-agent setup: Claude (chat), Claude Code, and Gerald.

This repo is the **contract layer**, not the brain. Per-project state and decisions live in each project's own repo (see `templates/`). This repo holds what is true across all projects: conventions, agent contracts, operator preferences, and the project registry.

## How each agent consumes this

| Agent | Mechanism |
|---|---|
| Claude Code | Global `~/.claude/CLAUDE.md` imports from the local clone at `{GLOBAL_ROOT}` (see `agents/claude-code.md`) |
| Gerald | Profile loader reads `agents/gerald.md` + `PREFERENCES.md` from `{GLOBAL_ROOT}` at session start |
| Claude (chat) | Fetches raw files from GitHub at session start, e.g. `https://raw.githubusercontent.com/profit4dogs/_global/main/PROJECTS.md` |

`{GLOBAL_ROOT}` = the fixed local clone path on the machine (set once, referenced everywhere). Convention: `C:\_global`.

## Repo map

- `CONVENTIONS.md` — rules of the vault. Read this before writing anything.
- `PREFERENCES.md` — operator working style. Loaded by all agents.
- `PROJECTS.md` — registry of all active projects, one line each, with repo links.
- `agents/` — one contract file per agent.
- `templates/` — canonical templates for per-project docs (`STATE.md`, `DECISIONS.md`, `SPEC.md`).

## Rules (summary — full version in CONVENTIONS.md)

1. Single source of truth per topic. A contradiction between two docs is a bug.
2. Supersede, don't delete. Mark docs `status: superseded-by <path>`.
3. No chat-summary dumping. Decisions and specs only, distilled.
4. This repo is PUBLIC. No strategy parameters, client names, credentials, or edge specifics. Those live in private project repos.
5. Token economy is a design constraint. Every file here gets loaded into agent context. Shorter is better.
