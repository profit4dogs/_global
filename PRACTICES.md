---
status: active
updated: 2026-07-25
---

# PRACTICES

Engineering practice rules that hold across all projects. Loaded by Claude Code directly (see `agents/claude-code.md`); other agents reach it via `PREFERENCES.md`.

Working style lives in `PREFERENCES.md`. Documentation rules live in `CONVENTIONS.md`.

## Branch discipline

**Never work on `main`.** Not a commit, not a one-line fix, not a docs-only change. This holds in every repo, `_global` included.

- **Fresh work gets a fresh branch.** Fetch first, then cut it from an up-to-date `main`.
- **If `main` is behind `origin/main`, stop and ask.** Do not silently branch from a stale base, and do not silently fast-forward to fix it. Which one is right depends on what is in flight, and that is the human's call.
- **Reusing the current branch is fine when it genuinely fits.** If the branch and `main` are both current and the new work belongs with what is already on it, add it as a separate commit. The rule is against working on `main` — not against reusing a branch that is already the right place.
- **Already sitting on `main` with uncommitted work?** Cut the branch *before* committing; `git switch -c <name>` carries the changes across. Never resolve it by committing first and sorting it out afterwards.

Run this check when work starts, not when the commit is written — discovering it at commit time means the branch point is already wrong.

This is a tier-3 written rule (see the ratchet below) and it is the weak tier on purpose: a global `core.hooksPath` pre-commit hook rejecting commits on `main` would be tier 2 and is genuinely available, but it needs an override path and a decision about which repos are exempt. Worth doing if this rule gets broken again.

## Recurring-failure ratchet

The **second** occurrence of a class of error, failure, or inconsistency triggers a durable prevention before the work continues. Fixing the instance and moving on is not enough — the repeat is the evidence that the instance-fix was not the fix.

**The bar is a genuine pattern:** the same mistake twice, from a cause that will recur. A flaky test is not this — fix the flake. A single mistake is not this — fix it. Two instances of "a maintainer will reach for X, and X is wrong here" **is** this.

Choose the **strongest applicable** prevention:

1. **Structural — make the error inexpressible.** A type, a required argument, a single choke point the operation must pass through. Strongest: nothing to remember and nothing to enforce, because the wrong version no longer compiles or no longer exists.
2. **Mechanical guard — a test or lint rule that fails the build when the error recurs.** Use when the error cannot be designed out. A guard must fail on the *class*, not on the one instance that prompted it.
3. **Written rule — `CLAUDE.md` or the relevant skill doc.** Only when neither of the above is possible: a judgment invariant no machine can check. Weakest, because it depends on someone reading and remembering it.

Prefer 1 over 2 over 3. A written rule is the fallback, not the default response — a guard that fails loudly beats a paragraph nobody re-reads.

**When the ratchet fires, say so explicitly:** name the recurring class, name the tier chosen, and name why no stronger tier was available. Choosing tier 3 requires justifying why 1 and 2 were genuinely impossible — that justification is the check against writing a rule where a guard would have done.
