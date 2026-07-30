---
name: spec
description: "Write spec.md directly from an existing brainstorm report or a short user description, with no exploration phase. Use when the user already has a clear idea and just wants a spec written (\"write a spec for X\", \"spec this out\", \"I know what I want, just write the spec\") — not when they want to explore options first (that's brainstorm). Writes exactly one file, no plan or code."
---

# spec — Write a Spec Directly

No exploration, no options comparison — the user already knows what they want.
If they're still weighing approaches, redirect to `$brainstorm` instead.

---

### Step 1 — Gather Input

Accept either:
- A path to an existing brainstorm report (`plans/reports/*-brainstorm.md`) — read it for context.
- A short description from the user — use as-is.

If required template sections (User Stories, Functional Requirements, Success Criteria) can't be filled from the input, ask the user directly rather than inventing content. Max 2–3 targeted questions.

---

### Step 2 — Write the Spec

Fill `.agents/skills/brainstorm/references/spec-template.md` using only the input gathered — no new sections, no extra files (no plan, no report, no code).

Write to `plans/{slug}/spec.md`, where `{slug}` is a short kebab-case name derived from the feature.

Leave `[NEEDS CLARIFICATION]` for anything still unresolved after Step 1's questions.

---

### Step 3 — Handoff

Ask via `ask the user directly`:

**"Spec written at `plans/{slug}/spec.md`. What next?"**
- `→ $plan plans/{slug}/spec.md` — proceed to planning
- `Keep editing` — revise the spec further

## Codex compatibility

Use the currently available Codex tools and skills for this workflow. If a referenced Claude agent, hook, MCP tool, or slash command is unavailable, perform the equivalent step inline, preserve the same artifact and verification requirements, and state the fallback briefly.
