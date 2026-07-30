# Design Review Checklist

Use this when reviewing a finished UI (yours or someone else's) before calling it done.

## Step 0 — design-system drift

Polish without alignment is decoration on top of drift — it makes the next person's job harder. Before scoring anything, classify every deviation from the existing design system by root cause, since the fix differs by category:

| Cause | What it looks like | Fix |
|---|---|---|
| Missing token | The value should exist in the system but doesn't | Add the token, don't hard-code the value |
| One-off implementation | A shared component already exists but wasn't used | Swap to the shared component |
| Conceptual misalignment | The feature's flow/IA/hierarchy doesn't match neighboring features (e.g. a settings page exposing 40 fields when the rest of the app reveals 5 at a time) | Rework the flow, not just the surface |

If anything about the existing system is ambiguous, ask — never guess at design-system principles.

## Step 1 — Anti-pattern / AI-slop pass

Run first. If this fails, fix it before scoring anything else.

**Absolute bans** (see SKILL.md):
- [ ] Side-stripe borders (`border-left/right` >1px as colored accent)
- [ ] Gradient text (`background-clip: text` + gradient)
- [ ] Glassmorphism used decoratively, not purposefully
- [ ] Hero-metric template (big number, label, gradient) with no real data behind it
- [ ] Identical card grids (icon + heading + text, repeated)
- [ ] Tiny uppercase tracked eyebrows above every section
- [ ] Numbered section markers (01/02/03) with no real sequence
- [ ] Headline/body text overflowing its container at any breakpoint
- [ ] Light gray body text that fails 4.5:1 contrast "for elegance"
- [ ] Cream/sand/parchment body background chosen by default, not by brand
- [ ] Purple-blue gradient as a default decorative choice

**The slop test, two altitudes**:
1. First-order: could someone guess the theme/palette from the category alone? (e.g. fintech → navy-and-gold, SaaS → purple gradient)
2. Second-order: could someone guess the aesthetic family from category + "not the obvious thing"? (e.g. "AI tool that's not SaaS-cream" → everyone converges on editorial-typographic). Both answers need to be non-obvious.

For product UI specifically, the test is different: would a user fluent in Linear/Stripe/Notion trust this without pausing, or does some component feel subtly wrong? Product's failure mode is strangeness without purpose (over-decorated buttons, invented affordances for standard tasks), not flatness.

## Step 2 — Nielsen's 10 heuristics (score 0-4 each)

Be honest — 4 means genuinely excellent, not "good enough." Most real interfaces score 20-32/40.

| # | Heuristic | What to check |
|---|---|---|
| 1 | Visibility of system status | Loading indicators, action confirmations, progress, current location |
| 2 | Match system/real world | No unexplained jargon, logical info order, recognizable icons |
| 3 | User control and freedom | Undo/redo, cancel buttons, clear path back, escape from multi-step flows |
| 4 | Consistency and standards | Same terminology/components/interactions everywhere |
| 5 | Error prevention | Confirm destructive actions, constrain invalid input, smart defaults |
| 6 | Recognition over recall | Visible options, autocomplete, labeled icons (not icon-only) |
| 7 | Flexibility and efficiency | Keyboard shortcuts, bulk actions, power-user paths that don't complicate basics |
| 8 | Aesthetic and minimalist design | Only necessary info visible, clear hierarchy, no decorative clutter |
| 9 | Error recovery | Plain-language errors, specific problem + fix, doesn't wipe user input |
| 10 | Help and documentation | Searchable, contextual, task-focused, easy to reach without losing context |

Rating bands: 36-40 excellent (ship), 28-35 good (fix weak areas), 20-27 acceptable (real work needed), 12-19 poor (major overhaul), 0-11 critical (redesign).

## Step 3 — Cognitive load checklist

8 items, 0-1 failures = low load (good), 2-3 = moderate (fix soon), 4+ = critical:

- [ ] Single focus — no competing elements distract from the primary task
- [ ] Chunking — info grouped in digestible chunks (≤4 items per group)
- [ ] Grouping — related items visually grouped (proximity, border, shared background)
- [ ] Visual hierarchy — immediately clear what matters most
- [ ] One decision at a time
- [ ] ≤4 visible options at any single decision point
- [ ] No working-memory demand — user doesn't need to recall info from a prior screen
- [ ] Progressive disclosure — complexity revealed only when needed

Practical caps: nav ≤5 top-level items, form sections ≤4 visible fields per group, ≤3 pricing tiers, ≤4 dashboard metrics above the fold.

## Step 4 — Severity tagging

Tag every finding:

| Tag | Meaning | Action |
|---|---|---|
| P0 Blocking | Prevents task completion | Fix immediately |
| P1 Major | Significant difficulty, or a WCAG AA violation | Fix before release |
| P2 Minor | Annoyance, workaround exists | Fix next pass |
| P3 Polish | No real user impact | Fix if time permits |

If unsure between two levels: "would a user contact support about this?" — if yes, it's at least P1.

## Step 5 — Persona spot-check (optional, for higher-stakes reviews)

Walk the primary action through 2-3 of these lenses; report what specifically breaks for each, not a generic description:

| Persona | Watches for |
|---|---|
| Power user | Forced onboarding, no keyboard shortcuts, one-at-a-time where bulk should exist |
| First-timer | Icon-only nav, unexplained jargon, no help, unclear next step after an action |
| Accessibility-dependent | Keyboard-only flow breaks, missing focus indicators, color-only meaning, alt text |
| Stress tester | Empty/edge states, refresh mid-flow losing data, silent failures |
| Mobile/distracted | Actions outside thumb zone, no state persistence across interruption, heavy assets |

## Reporting format

State what's wrong AND why it matters to the user, then give a concrete fix — never "consider exploring...". Prioritize ruthlessly; if everything is P0, nothing is. Don't soften technically accurate criticism — vague feedback wastes the implementer's time.

## Final polish checklist

Run once functionally complete, in this order (functional issues ship before cosmetic ones):

- [ ] Drift named and resolved by root cause (table above)
- [ ] Visual alignment correct at every breakpoint, spacing uses tokens (no arbitrary 13px gaps)
- [ ] All 8 interactive states implemented per component
- [ ] Transitions smooth at 60fps, reduced-motion respected
- [ ] Copy terminology and capitalization consistent throughout
- [ ] Forms: labels present, errors helpful, tab order logical
- [ ] Empty/loading/error/success states all designed, not just the happy path
- [ ] Touch targets ≥44×44px, contrast meets WCAG AA, keyboard nav works
- [ ] No console errors, no layout shift on load, no debug code left in

A clean automated check (linter, detector, accessibility scanner) is defect evidence, never proof the design is good — walk the actual interaction path yourself before marking done.
