---
name: guided-onboarding-tours
description: >-
  Build interactive guided tours, onboarding walkthroughs, coach-mark/spotlight
  flows, product tours, and step-by-step in-app tutorials. Use this skill
  whenever the work involves guiding a user through a real app flow — new-user
  checklists, feature walkthroughs, "show me how" tours, tooltips that advance as
  the user acts, or any multi-step interactive tutorial — even if the request
  doesn't say the word "tour." It encodes hard-won rules about advancing tips,
  not blocking or stacking UI, teaching navigation, and testing tours so they
  don't silently regress. Consult it before designing or wiring any walkthrough.
---

# Guided Onboarding Tours

This skill encodes what was learned building a real interactive onboarding
walkthrough — mostly by shipping the wrong thing first and fixing it. Each rule
exists because its absence caused a concrete failure. Read the *why*; it's what
lets you apply the rule to a situation these notes didn't anticipate.

This skill assumes the project's global workflow rules still apply (smallest
independently-testable commits, root-cause before fixing, tests in each commit).
It does not repeat them.

---

## 1. Core principle: guide through the real UI

Guide the user through the *actual working flow*, not a representation of it.
Never a demo screen, a reconstruction, a fake sequence, or a "watch this happen"
animation. The user performs the real actions on the real controls, and the tour
rides alongside.

Everything else in this skill derives from this. The whole value of a guided tour
is that the user *does the thing once, for real, with help* — so they can do it
again alone. A representation teaches recognition; the real flow teaches the
motion. If you catch yourself building a stand-in for the UI because the real UI
is awkward to tour, fix the UI or fix the tour — do not build the stand-in.

---

## 2. When to advance a stop

**Required stops advance on the user's real action** on the real element (a click,
a change, a selection) — never a "Next" button in the tooltip. A Next button on a
required step is almost always a patch over a listener you didn't wire correctly,
and it breaks the "do it for real" principle by letting the user skip past the
action without performing it.

**Optional stops may advance via a Next button.** An optional stop has no required
action to key on — it's informative, or a shortcut the user may decline — so Next
is the *correct* mechanism there, not a crutch. The distinction is the rule:
required → real action; optional → Next allowed.

**The preset-value trap.** A field pre-filled with an acceptable value (a date
defaulting to today, a balance defaulting to 0) will auto-advance the instant you
check "is this valid?" — because it was already valid, with no user action. Two
consequences:
- Gate a required field's advance on a user *edit* that yields a valid value, not
  on validity alone.
- A required field whose preset the user may legitimately accept unchanged is the
  one place a required stop gets a **Next fallback** — otherwise accepting the
  preset (doing nothing) traps them forever. This is the single sanctioned Next on
  a required stop, and only for this reason.

**Custom components need matched listeners.** Date pickers, portal-rendered
dropdowns, and other non-native inputs often fire their events *outside* the
anchored element, or emit no native `input`/`change` event at all. When a stop
"won't advance on input," the cause is usually a listener bound to the wrong
element or the wrong event — diagnose the component's real event, don't reach for
Next.

**Never offer an advance control for an action that cannot succeed.** Wherever a
Next is sanctioned (an optional stop, or the preset fallback above), it must be
reachable only while the action behind it can actually complete. A Next on an
empty required field walks the user onto the next stop, whose submit then fails
validation — and if that stop waits on a result rather than a click, there is no
control left and the tour is stuck. Gate the control, or the escape hatch becomes
the trap.

**One stop, two advance paths, one guard.** A stop often advances more than one
way — focus leaving a filled field *and* a confirm button. If the guard lives on
one path only, the ungated path skips the field and strands the user, while every
test that exercises the guarded path stays green. Extract the predicate into a
function and have both paths call it; two copies of the same question drift apart
the moment either is edited. This is the sharpest edge of the preset-value trap
above: the preset made the ungated path harmless, so removing the preset is what
exposes it.

**A gate computed at render needs live DOM state.** Typing into an input does not
re-render, so a control gated on "is the field filled?" never appears. Subscribe to
the element and mirror it into state — and sync once on mount, because a re-entered
stop may already be satisfied and the control must not stay hidden.

**Platform-divergent advance mechanisms need platform-divergent gates.** Where one
platform advances on the keystroke and another on an explicit confirm, a gate added
to one is invisible to the other. The unguarded platform is usually the one with
the extra control — the confirm — which is also the one that can skip.

---

## 3. Skip only when the skip teaches nothing new

Do **not** skip a step merely because some pre-existing state flag is satisfied.
That robs the user of the lesson the step exists to teach. The purpose of the tour
is instruction, not task-completion bookkeeping.

A step may be skipped **only when all four hold**:
- **(a) Performed, not merely satisfied** — the user actually did the action
  themselves *in this session*, not that prior state happens to satisfy it.
- **(b) Demonstrated while doing** — it was shown/described to them as they did it,
  so the skip doesn't erase a lesson they never received.
- **(c) Repeatable** — they saw *how*, not just watched a result appear, so they
  could perform it again unaided.
- **(d) Undemonstrated variants covered** — any variation not shown is either
  simple enough to need no teaching, or is pointed out elsewhere (a summary card,
  a later tip). See §8.

If any condition fails, walk the step.

**Worked example (keep this behavior):** A user tags a recurring bill to a budget
category and *sees how tagging works*. Tagging a one-off transaction is the same
motion, so re-walking a separate "tag a one-off" step teaches nothing new — (a)
they did it, (b) it was demonstrated, (c) same repeatable motion. The one
undemonstrated wrinkle (that non-recurring items can be tagged too) is surfaced on
a summary card, satisfying (d). Fold is correct.

**Anti-pattern (forbid this):** Skipping a step because a stored flag is set, where
the user never performed the action in the tour and never saw the motion. Fails
(a) and (b). This is "outsmarting the user," and it silently strips the teaching.
Never skip on inferred pre-existing state alone.

---

## 4. Never block, never stack, never obscure

**Never block the UI.** Use click-through anchored tooltips that let the user touch
the real control underneath. A modal the user must dismiss before acting
contradicts §1 — they can't act on the real UI if a box is covering it.

**Never stack tooltips.** Render exactly one tooltip per stop, through a single
shared tooltip component. Do not render a bespoke custom callout *alongside* the
standard tour chrome — that produces two boxes saying the same thing (the
"double-box" bug), often with one clipping off-screen. A step's content flows
*into* the one shared tooltip; it does not spawn its own second box. This also
makes later restyling tractable: one component to restyle, not a standard tooltip
plus scattered bespoke boxes the restyle misses.

**Never obscure what you point at.** If the tooltip covers the element it's
highlighting, move the *tooltip*, not the highlight. The highlighted thing is the
point; the words describing it must not bury it. Pin the tooltip to clear space
(e.g. onto the action button's container) rather than letting it default on top of
its own target.

---

## 5. Navigation is part of the flow

Teach the user how to *reach* the screen where a step happens — not just what to do
once they're there. A user who's shown "fill this field" but not "here's how you
got to this page" hasn't learned the flow.

- A **nav stop precedes the action stops** of any step that lives on a different
  screen, and **advances on route arrival** (the real navigation event), not on a
  Next button.
- **In-page navigation counts.** "Tap a calendar day to create a transaction" is a
  nav action even though it doesn't change routes. Treat it with the same nav-stop
  shape.
- **Navigation that also supplies data demotes the later stop.** If tapping a
  calendar day both navigates to the create sheet *and* prefills that day's date,
  the later date stop becomes informative (confirm/change, don't require) — the
  user may have already set it by how they navigated. See §2's preset handling.
- **Skip the nav stop if already on-screen.** Pointing a user to a nav control that
  goes where they already are is a dead-end stop (§3-style). Evaluate "already on
  the target route" at step entry and skip cleanly; advance on arrival, which the
  user already satisfied.

---

## 6. The step loop and data-driven definition

The per-step shape is one repeated loop:

```
land on checklist → nav stop (reach the screen) → drive action stops (real events)
→ hand back to checklist → advance tracker → launch next step → … → closing screen
```

Build this as **one shared engine consuming step definitions as data** — each step
declares its page, its ordered stops (each with an advance trigger and, for
optional stops, its Next affordance), and its completion signal. Do **not**
hand-author per-step handlers; that path produces divergent, unmaintainable stops
and makes the loop impossible to reason about.

**Commit the completion signal before resuming.** When a step finishes and hands
back, the step's done-flag/state write must land *before* the tour computes where
to resume — otherwise resume reads stale state and lands on the wrong step (often
the one just finished, or one ahead). Sequence the write-then-resume explicitly;
this race does not fail loudly, it just misroutes.

---

## 7. Conditional and late-appearing stops

A stop targeting a field that only appears after a user choice (e.g. a "name your
new category" field that renders only when the user picks "New") must not trap the
flow when that field never appears.

Give such stops a **grace-period auto-skip keyed on a real appearance signal** —
if the conditional element doesn't materialize, the stop advances rather than
waiting forever on an element that will never exist. Key the skip on the actual
render/appearance signal where possible; a fixed timeout is a fallback, not the
design, because timing varies by device and load.

---

## 8. Replay mode vs first-run mode

A tour relaunched from Settings by an *already-set-up* user is a different mode
from first-run, and conflating them produces an empty walkthrough (every step
satisfied → nothing to do).

- **First-run mode:** advances on real actions (§2), applies the §3 skip rule.
- **Replay/learn mode:** walks the *full* sequence ignoring done-flags, because the
  point is to re-teach, not re-complete. Since the user won't redo the real
  mutations, replay advances via **Next**, and performs the cross-surface
  *navigation* between stops (so they see *where* things are) **without performing
  or requiring the mutating action**. If a stop's navigation is entangled with its
  mutation such that you can't navigate without mutating, stop and flag it rather
  than forcing a mutation in replay.

**Corollary (ties to §3d):** any lesson that lives *only* in a step that can be
folded or skipped must *also* live somewhere that always fires — a summary/
things-to-know card. A lesson hostage to a step that may not run is a lesson the
common-path user never gets.

---

## 9. Progress indication

Show the step the user is **on**, 1-indexed — not a 0-indexed completed-count.
"1 of 5" while standing on the first step; "5 of 5" on the last. A "0 of 5" against
steps numbered 1–5 reads as broken even when the count is technically correct.

The progress indicator and the active walkthrough must read from **one shared
source of truth** for "current step," or the tracker and the tooltip will disagree
about where the user is.

---

## 10. Authoring against the real UI (process)

- **Inventory the real click path first, read-only.** Before writing any stop,
  enumerate the actual elements a user touches for each step, in order, from the
  real components. Mirror that path; do not invent an idealized "minimal flow" that
  the real UI doesn't match.
- **Add anchors additively.** Where target elements lack stable hooks, add
  `data-tour-id` (or equivalent) anchors as no-behavior-change additions, and
  record which were added.
- **UI-before-tour ordering.** A stop cannot point at an element that doesn't exist
  yet. Any new UI a stop depends on must land and be verified *before* the stop is
  authored against it. Separate "build the field" from "point the tour at the
  field" and sequence them in that order.
- **Re-verify positioning after layout changes.** Any change that reoffsets the
  layout (a width cap, a container, a nav change) can break anchored-tooltip
  positioning. Re-verify anchor/overlay math after it, as its own step.
- **Region highlights may be needed.** Some stops highlight more than one element —
  e.g. two calendar week-rows as a single outline. If the engine only anchors to a
  single element, adding region/bounding highlighting is a gating capability to
  plan before those stops, not an afterthought.

---

## 11. Testing a tour

Assert the **real outcome**, not a proxy for it. The characteristic failure mode is
a test that passes while the flow is broken — asserting the math is right, or that
the *first* element rendered, while the actual user-visible behavior (later stops,
future pills, correct advancement) is broken. A green test over a broken tour is
worse than no test: it manufactures false confidence and lets the bug ship twice.

Cover at minimum:
- Each stop advances on its **real** trigger (and optional stops on Next).
- A satisfied step skips **only** when §3's conditions hold; an unsatisfied one is
  walked.
- A nav stop **skips when already on-screen** and advances on arrival.
- The full flow completes **end to end**, reaching the closing screen.
- Back-out **restart/resume lands on the correct step** (the §6 write-before-resume
  race).
- Conditional stops (§7) don't trap when their element never appears.
- **Arriving at a stop and deliberately NOT acting** still leaves a way forward.
  Specs that stop short of the action, and specs that perform it correctly, both
  pass while the skip path is broken — the gap sits between them.
- Assert on the **presence or absence of the advance control**, not only on copy.
  Copy assertions break on rewording and stay green on a dead end.
- Assert the specific user-visible thing (a stop present on a specific later step,
  a tooltip on the right element) — not just that upstream state is correct.
- Run on **all target platforms**, not just one.

---

## 12. Baseline practices (general, not battle-tested here)

These are standard tour-building practices included for completeness. Unlike
§§1–11, they were not each stress-tested in the work that produced this skill —
apply judgment rather than treating them as proven.

- **Accessibility:** manage focus onto the active tooltip; support keyboard
  advancement for optional/Next stops; make the tour dismissible with ESC and via
  a visible exit; don't trap focus such that a user can't reach the real control.
- **Persistence:** auto-offer the tour once, remember that it was offered/dismissed,
  and make it re-launchable (which lands you in replay mode, §8). Don't re-offer on
  every load.
- **Copy:** short, present-tense, action-oriented tooltip text ("Tap Accounts to
  open your accounts"), one instruction per stop. The tooltip says what to do now,
  not a paragraph about why.

---

## Quick reference

| Situation | Rule |
|---|---|
| Required stop advances | On the real action; never Next (§2) |
| Optional stop advances | Next allowed (§2) |
| Field has an acceptable preset | Gate on user edit; Next fallback so it can't trap (§2) |
| Stop won't fire on custom input | Wrong element/event, not a reason for Next (§2) |
| Next shown on a required stop | Gate it on the action being able to succeed (§2) |
| Stop advances two ways | One shared predicate, both paths; never two copies (§2) |
| Gate depends on field contents | Subscribe to the element; render state won't update itself (§2) |
| Platforms advance differently | Gate each path; the confirm platform is the skippable one (§2) |
| Considering skipping a step | Only if all four §3 conditions hold |
| Skipping on a pre-existing flag alone | Forbidden (§3) |
| Tooltip covers its target | Move the tooltip, not the highlight (§4) |
| Two boxes appear | One shared tooltip; kill the bespoke second box (§4) |
| Step is on another screen | Nav stop first, advance on arrival (§5) |
| Already on the target screen | Skip the nav stop (§5) |
| Resume lands on wrong step | Commit completion signal before resuming (§6) |
| Conditional field may not appear | Grace-period auto-skip on appearance signal (§7) |
| Relaunched by a set-up user | Replay mode: full sequence, Next, navigate-don't-mutate (§8) |
| Lesson lives in a skippable step | Also put it on an always-fires summary card (§3d, §8) |
| Progress counter | 1-indexed, step you're on, shared source of truth (§9) |
| New stop needs new UI | Build and verify the UI before authoring the stop (§10) |
| Writing tests | Assert real user-visible outcome; never green-over-broken (§11) |
