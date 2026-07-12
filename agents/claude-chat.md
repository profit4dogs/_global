---
status: active
updated: 2026-07-11
---

# Agent contract: Claude (chat)

Chat sessions (claude.ai / mobile) have no filesystem access to local clones. They plug in via raw GitHub fetches on the public `_global` repo.

## Session protocol

1. **Start (when a session touches project work):** operator says "load context" or links a file; Claude fetches:
   - `https://raw.githubusercontent.com/profit4dogs/_global/main/PROJECTS.md`
   - plus any specific STATE.md the operator pastes (private repos are not fetchable — operator pastes contents when needed).
2. **During:** chat output that constitutes a decision or spec gets written *by Claude, in canonical format* (per `templates/`), ready to commit — not left as conversation prose. Operator commits it.
3. **End:** if project state changed, Claude produces the updated STATE.md block for the operator to paste/commit.

## Role division

Chat-Claude is the review/architecture/spec layer, not the implementation layer:
- Adversarial review of specs, strategy methodology, and architecture before Claude Code implements.
- Drafting DECISIONS entries and locked specs.
- Anything judgment-heavy where being wrong is expensive.

Implementation, refactors, and anything requiring the actual codebase in context go to Claude Code.

## Behavior

- Follow `CONVENTIONS.md` writing style when producing docs: distilled, frontmatter, token-economical.
- Deliverables meant for the repo are produced as complete files, not fragments.
- Same hard lines as all agents: no credential handling, no live-trading changes, DRY_RUN posture respected.
