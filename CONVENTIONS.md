# CONVENTIONS

Rules for all markdown docs across all repos. All agents follow these when reading or writing project documentation.

## Document types

Every project repo carries at most these doc types, at repo root or under `docs/`:

- **STATE.md** — current status snapshot. Overwritten in place, always current. One per project. Template: `_global/templates/STATE.md`.
- **DECISIONS.md** — append-only architecture decision log. Never edit past entries except to change `status`. Template: `_global/templates/DECISIONS.md`.
- **specs/*.md** — locked specifications. A spec marked `status: locked` is implemented as written; changing it requires a DECISIONS entry first. Template: `_global/templates/SPEC.md`.

Nothing else. No journals, no meeting notes, no chat dumps.

## Core rules

1. **Single source of truth.** Each fact lives in exactly one file. Other files link to it; they do not restate it. If two docs disagree, that is a bug — fix it in the same commit you notice it.
2. **Supersede, don't delete.** Set `status: superseded-by <path-or-entry>` in frontmatter. History stays greppable.
3. **Distill, don't dump.** A decision entry is 3–8 lines. If you are pasting conversation output, you are doing it wrong. Write the conclusion, the reason, and what it replaced.
4. **Status frontmatter is mandatory** on every doc:
   ```yaml
   ---
   status: active | locked | superseded-by <path> | parked
   updated: YYYY-MM-DD
   ---
   ```
5. **Agents update STATE.md at the end of any session that changed project state.** This is the handoff mechanism between Claude Code, Gerald, and chat sessions. If it isn't in STATE.md, the next session doesn't know it.
6. **Token economy.** These files are loaded into model context. Target: STATE.md under 60 lines, DECISIONS entries under 8 lines, agent contracts under 80 lines. Cut adjectives, keep facts.
7. **Public/private split.** `_global` is public: no strategy parameters, thresholds, client names, account details, API endpoints, or credentials — link to the private repo instead. Private project repos may contain anything except credentials (those stay in `.env`, never committed).

## Writing style

- Declarative, present tense. "Renders use Cloudinary" not "we decided to switch to Cloudinary".
- Dates in ISO (YYYY-MM-DD).
- Name the constraint, not the history, unless the history is the reason (then it goes in DECISIONS).
- `parked` means intentionally stopped with a reason recorded. It is not a soft delete.
