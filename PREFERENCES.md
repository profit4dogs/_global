---
status: active
updated: 2026-07-25
---

# PREFERENCES (operator: Liam)

Working-style contract. All agents load this.

## Communication

- Direct and honest. Critique over validation. If the plan is bad, say so before executing it.
- Lead with the answer, then the reasoning. No preamble, no restating the question.
- Flag uncertainty explicitly. "I don't know" beats a confident guess.
- One clarifying question max, and only if genuinely blocked. Otherwise state the assumption inline and proceed.

## Execution

- Targeted, scoped changes. No broad rewrites unless explicitly requested.
- Fail fast. Surface errors loudly; never silently fall back or degrade. Placeholder/fallback behavior must be gated behind explicit flags that default to off.
- Exploration findings are hypotheses, not conclusions. In-sample and out-of-sample stay separated. Walk-forward before belief.
- Promote fixes as complete units, not piecemeal. A spec marked `locked` is implemented as written.
- Prefer boring, verifiable steps over clever, opaque ones.
- Never work on `main` — fresh work gets a fresh branch off an up-to-date `main`; if `main` is behind, ask before branching. Full rule: `PRACTICES.md` § Branch discipline.
- Second occurrence of a failure class triggers a durable prevention, at the strongest available tier, before the work continues. Full rule: `PRACTICES.md` § Recurring-failure ratchet.

## Environment

- OS: Windows. Primary: Ryzen 5 5500 / RX 6600 / 32GB. Secondary: i7 / GTX 1080 laptop.
- New and migrated project clones go under `E:\projects` (`{PROJECTS_ROOT}`); older clones still under C: move as they are touched. Never assume either root — check. The `_global` clone stays at `C:\_global`.
- Languages: Python (uv), Pine Script v6, TypeScript/Next.js.
- Local LLM: Ollama pinned at v0.22.1 (RX 6600 Vulkan stability). Do not upgrade without explicit approval.
- `DRY_RUN=true` is the default posture for anything that publishes or trades. Going live is always an explicit human decision.

## Hard lines

- Never commit credentials. Never weaken a safety gate to make a test pass.
- Trading: no live execution changes without explicit instruction in the current session.
- Publishing: nothing posts publicly while `DRY_RUN=true`.
