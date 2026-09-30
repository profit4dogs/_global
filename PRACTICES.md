---
status: active
updated: 2026-09-30
---

# PRACTICES

Engineering practice rules that hold across all projects. Loaded by Claude Code directly (see `agents/claude-code.md`); other agents reach it via `PREFERENCES.md`.

Working style lives in `PREFERENCES.md`. Documentation rules live in `CONVENTIONS.md`.

## Recurring-failure ratchet

The **second** occurrence of a class of error, failure, or inconsistency triggers a durable prevention before the work continues. Fixing the instance and moving on is not enough — the repeat is the evidence that the instance-fix was not the fix.

**The bar is a genuine pattern:** the same mistake twice, from a cause that will recur. A flaky test is not this — fix the flake. A single mistake is not this — fix it. Two instances of "a maintainer will reach for X, and X is wrong here" **is** this.

Choose the **strongest applicable** prevention:

1. **Structural — make the error inexpressible.** A type, a required argument, a single choke point the operation must pass through. Strongest: nothing to remember and nothing to enforce, because the wrong version no longer compiles or no longer exists.
2. **Mechanical guard — a test or lint rule that fails the build when the error recurs.** Use when the error cannot be designed out. A guard must fail on the *class*, not on the one instance that prompted it.
3. **Written rule — `CLAUDE.md` or the relevant skill doc.** Only when neither of the above is possible: a judgment invariant no machine can check. Weakest, because it depends on someone reading and remembering it.

Prefer 1 over 2 over 3. A written rule is the fallback, not the default response — a guard that fails loudly beats a paragraph nobody re-reads.

**When the ratchet fires, say so explicitly:** name the recurring class, name the tier chosen, and name why no stronger tier was available. Choosing tier 3 requires justifying why 1 and 2 were genuinely impossible — that justification is the check against writing a rule where a guard would have done.

## Deploy pipeline for public-facing apps

Any repo connected to a production environment follows the three-tier flow: **feature branch -> preview -> main**. Production serves `preview` and `main` only; feature branches never deploy.

**Versioning.** Semver, bumped automatically when a PR merges to `preview`. The version increment travels with the `preview -> main` merge. Bump type comes from the PR title prefix: `feat:` = minor, `breaking:` = major, everything else = patch.

**When to wire this up.** Before the first public deploy — not after. Connecting a repo to a production hosting environment and adding the pipeline are the same step.

**Implementation.** Template files in `templates/deploy-pipeline/`:
- `version-bump.yml` — GitHub Actions workflow, copy to `.github/workflows/`.
- `vercel-ignore.md` — the ignore-command pattern for Vercel projects (add to `vercel.json`). Swap this for the equivalent CI/CD gate when the hosting target changes.

Both are reference files to copy and adapt, not drop-in. The branching rule is the invariant; the hosting target is the variable.
