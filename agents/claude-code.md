---
status: active
updated: 2026-07-25
---

# Agent contract: Claude Code

Install: point the global `~/.claude/CLAUDE.md` at the local clone using imports:

```markdown
# ~/.claude/CLAUDE.md
@C:\_global\PREFERENCES.md
@C:\_global\PRACTICES.md
@C:\_global\agents\claude-code.md
```

Project-level `CLAUDE.md` files stay in each project repo and add project specifics on top of this.

## Session protocol

1. **Start:** read the project's `STATE.md` before writing code. If it contradicts what you see in the codebase, stop and flag it — do not guess which is right.
2. **During:** decisions that change architecture, dependencies, or locked specs get a `DECISIONS.md` entry in the same commit as the change.
3. **End:** update `STATE.md` if project state changed (what was done, what is now blocked/next). This is the handoff to Gerald and chat sessions — skipping it breaks the system. `CLAUDE.md` is **not** part of this reflex — it holds invariants, not status.

## Behavior

- Follow `_global/CONVENTIONS.md` for all doc reads/writes.
- Standing docs (`CLAUDE.md`, agent contracts, skill docs) carry invariants only, never point-in-time facts; a branch that invalidates one fixes it in the same PR; verify each claim against the code before rewriting, trusting neither the file nor the prompt. Full rule: `CONVENTIONS.md` § Doc currency.
- Scoped diffs. Touch the minimum surface that fixes the problem. No drive-by refactors.
- Never weaken tests, gates, or safety flags to get green. `ALLOW_*_FALLBACK` flags default false and stay false.
- Windows environment: mind path separators, UTF-8 encoding on all file I/O (`encoding="utf-8"` explicitly in Python — silent encoding failures have bitten this codebase before).
- Native+web dual-target apps (Capacitor/PWA): gate native-only features on `Capacitor.isNativePlatform()`; they must never break the web build, and UI copy must not claim a capability the shipped config doesn't guarantee (e.g. "offline"). An unsupported platform degrades to the existing manual path, never a broken state.
- Specs under `specs/` marked `locked` are implemented exactly as written. Deviation requires a DECISIONS entry approved by the operator first.
- Trading repos: never modify order-execution paths, risk gates, or live-account config unless the operator's current-session instruction explicitly names the change.
