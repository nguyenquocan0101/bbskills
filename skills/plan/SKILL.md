---
name: plan
description: "Plan a feature or system before implementation. Use when the user says \"plan this\", \"I want to build X\", \"how do I implement Y\", or when $brainstorm produces a spec.md. Always run before $cook. Modes (pick one): --fast (skip all, instant plan), --hard (2 researchers + red-team + validate), --two (2 approaches → compare → pick → cook), --parallel (parallel-impl plan → cook --parallel), --auto (full pipeline + auto-cook). Composable flags: --tdd, --no-task — propagate into the cook pipeline."
---

# plan — Structured Planning Pipeline

## Mode Reference

| Mode         | Research                         | Red-Team     | Validate    | Cook handoff                     |
| ------------ | -------------------------------- | ------------ | ----------- | -------------------------------- |
| `--fast`     | —                                | —            | —           | `$cook --fast`                |
| `--hard`     | 2 researchers                    | ✓            | ✓ (wait)    | `$cook --hard`                |
| `--two`      | 2 researchers (one per approach) | ✓ both plans | pick A or B | `$cook [user-chosen mode]`    |
| `--parallel` | 2 researchers                    | ✓            | optional    | `$cook --parallel`            |
| `--auto`     | 1 researcher                     | ✓            | ✓ (wait)    | auto-invoke cook (detected mode) |

**Auto-detect** (no mode given): Fast if single-file / familiar / ≤ 2 components; Hard otherwise.

**Flag defaults** (no composable flags given): `--tdd` and `--no-task` are both off — no special behavior applied.

---

### Step 0 — Scope Challenge

Before spawning any agents, detect mode and challenge scope:

```
# Scope Challenge:
#   Exists?     → [does this feature already exist in the codebase?]
#   Minimum?    → [smallest impl that satisfies requirements]
#   Complexity? → [Fast | Hard | Two | Parallel | Auto]
#
# Mode: [detected or explicit]
# Test:  [default | --tdd]
# Tasks: [default | --no-task]
```

If scope is too large: suggest splitting and **wait for user confirmation**.

If `--hard` / `--two` / `--parallel` and novel/ambiguous with no brainstorm report: "No brainstorm found. Run `$brainstorm` first? [Y/n]" — if Yes, stop; if No, proceed.

If a spec file path is provided or `plans/{slug}/spec.md` exists adjacent to any plan: run a **Spec Quality Check** inline:

```
# Spec Quality Check:
#   [NEEDS CLARIFICATION] remaining? → CRITICAL — resolve before continuing
#   Success criteria measurable?     → HIGH if vague adjectives (fast, scalable, reliable)
#   User stories P1/P2/P3?           → HIGH if missing
#   Acceptance criteria testable?    → MEDIUM if vague ("works correctly")
#
# Verdict: [PASS | WARN (list) | BLOCK (list)]
```

- **BLOCK**: resolve before proceeding
- **WARN**: user acknowledges — proceed
- **PASS**: continue

---

### Step 0.5 — Risk Classification

Classify the request **before spawning any agents**. This step runs in all modes including `--fast` — it cannot be deferred or skipped.

**Tier definitions:**
- `tiny` — single-file change, fully reversible, no user-facing behavior change
- `normal` — multi-file, testable, no auth/data/infra risk
- `high-risk` — touches auth, schema changes, external APIs, infrastructure, or any irreversible operation

Output one line inline:

```
Risk: tiny | normal | high-risk — {one-line reason}
```

Write this line into the `plan.md` header block immediately after the `Mode:` line. If planning from scratch (no plan.md yet), write it after the mode is determined in Step 0.

**`--fast` mode**: classification still runs — output inline and write to plan.md before emitting the cook command. This is the only mandatory step in `--fast` mode that is not skippable.

---

### Step 1 — Research

**`--fast`**: skip entirely.

**`--auto`**: spawn **1 `researcher` agent** — primary approach and best practices.

**`--hard` / `--parallel`**: spawn **2 `researcher` agents in parallel**:

- Instance A — role: `Primary` — recommended approach and best practices
- Instance B — role: `Alternative` — alternative approach and tradeoffs

**`--two`**: spawn **2 `researcher` agents in parallel**, each investigating one distinct approach:

- Instance A — role: `Approach A` — first viable approach (architecture, tradeoffs)
- Instance B — role: `Approach B` — second viable approach (meaningfully different strategy)

```
// Researcher A: [approach] → [verdict]
// Researcher B: [approach] → [verdict]
```

---

### Step 2 — Plan Creation

Spawn the **`planner` agent** with: feature description + mode + research reports + test flag + spec file path (if any).

**After planner returns**: capture the plan directory path from its "Directory: plans/{date}-{slug}/" line — you'll need it in Step 3.

- **`--tdd`**: planner adds `### Tests to Write First` to each phase, derived from spec acceptance criteria
- **Spec provided**: planner maps each phase to the P1/P2/P3 stories it covers
- **`--two`**: planner writes `plan-a.md` + `plan-b.md` (one per approach) — no `plan.md` yet
- **`--parallel`**: planner adds `## File Ownership` section to each phase file

Output structure:

```
plans/{slug}/
  plan.md            ← all modes except --two
  plan-a.md          ← --two only
  plan-b.md          ← --two only
  phase-01-{name}.md
  phase-02-{name}.md
  ...
```

---

### Step 2.5 — Write feature_list.json

**`--two` mode**: skip this step — defer to Step 4 (after approach merge).

After the planner returns and plan files are confirmed on disk:

1. Locate **project root** — the directory containing `.ck.json` (walk up from the plan directory).
2. Determine feature entries. Each entry requires **all** of these fields — never omit any:
   - **If spec.md was loaded**: extract P1 user stories → one entry per story. `id` = story slug, `title` = user-visible behavior, `user_visible_behavior` = "so that {value}" clause, `verification_command` = "Accepted when" condition, `status` = `"not_started"`, `evidence` = `""`, `notes` = `""`, `priority` = P1/P2/P3 marker from the story heading. Do not populate `blocked_by` — it is user-managed and set manually when a cross-feature dependency is identified during implementation.
   - **No spec**: use plan phases as fallback → one entry per phase. `id` = phase slug, `title` = phase name, `user_visible_behavior` = `""`, `verification_command` = `""`, `status` = `"not_started"`, `evidence` = `""`, `notes` = `""`. Do not populate `priority` or `blocked_by`.
3. **Idempotency**: if `feature_list.json` already exists at project root, merge — add new entries, preserve `status` and `evidence` for any `id` already present.
4. Write `{project_root}/feature_list.json` (NOT inside the plan subdirectory).
5. Log: `feature_list.json → {project_root}/feature_list.json ({N} features)` or `({N} new + {M} preserved)` on merge.

---

### Step 3 — Red-Team Review

**`--fast`**: skip.

**All other modes**: before spawning `plan-reviewer`, **verify plan files exist on disk** using Glob on the captured plan directory:
- Normal modes: `plans/{date}-{slug}/plan.md` must exist
- `--two` mode: `plans/{date}-{slug}/plan-a.md` + `plans/{date}-{slug}/plan-b.md` must exist

If files are missing: **stop** — output `"Planner failed to write files. Do not proceed."` Do not fall back to writing the plan inline.

Spawn **`plan-reviewer`** with all plan files (+ spec.md if present).

**`--two`**: reviewer evaluates both plan-a and plan-b — flag risks in each separately.

Adjudicate each finding:

- `ACCEPTED` → edit the relevant plan file immediately
- `NOTED` → append to Risks section of `plan.md` (or `plan-a.md` / `plan-b.md` in `--two` mode)
- `REJECTED` → document reason

If `plan-reviewer` returns `BLOCK`: revise the flagged phase and re-run before proceeding.

---

### Step 4 — Validation + Handoff

**`--fast`**: skip questions — proceed to the Detailed Phase Summary, then output the cook command.

**`--two`**: present a side-by-side comparison, then wait for the user to pick:

```
## Approach Comparison
Plan A: {1-line summary}
  Pros: {key strengths}  |  Cons: {key tradeoffs}

Plan B: {1-line summary}
  Pros: {key strengths}  |  Cons: {key tradeoffs}

Which approach? [A/B]
```

After selection: ask 2–3 targeted questions about the chosen plan. Merge chosen plan into `plan.md`, delete the rejected file. Then run Step 2.5 (write `feature_list.json` at project root) — this is the deferred trigger for `--two` mode.

**`--hard` / `--parallel` / `--auto`**: ask 3–5 targeted questions about the plan's riskiest points. **Wait for user answers.**

After validation: hydrate tasks via TodoWrite (skip if `--no-task`). Recommend `--tdd` if spec.md exists and it's not already set.

Output the exact cook command:

| Mode         | Cook command                                                                                 |
| ------------ | -------------------------------------------------------------------------------------------- |
| `--fast`     | `$cook --fast [--tdd] plans/{slug}/plan.md`                                               |
| `--hard`     | `$cook --hard [--tdd] plans/{slug}/plan.md`                                               |
| `--two`      | `$cook [--fast\|--hard] [--tdd] plans/{slug}/plan.md`                                     |
| `--parallel` | `$cook --parallel [--tdd] plans/{slug}/plan.md`                                           |
| `--auto`     | Automatically invoke `$cook --{detected-mode} plans/{slug}/plan.md` — no separate command |

---

### Step 5 — Detailed Phase Summary

After all plan files have been written, reviewed, and (when required) validated, read every
`phase-*.md` file from the plan directory and include a detailed summary in the final chat
response. Do not stop at showing only the directory tree or file names. Summarize the actual
contents from disk; do not invent details that are not present in the phase files.

For each phase, include:

- Phase name and objective
- Scope and concrete implementation tasks
- Files/modules affected
- Dependencies on earlier or later phases
- Tests and acceptance criteria
- Risks, decisions, or open questions

Use this format:

```
## Phase 01 — {name}
Mục tiêu: {objective}
Phạm vi:
- {task or scope item}

Files/modules: {files or modules}
Dependencies: {dependencies or none}
Tests/acceptance: {tests and measurable criteria}
Risks/notes: {risks, decisions, or none}
```

For `--two`, provide the detailed summaries only for the selected and merged plan after the
user chooses an approach. For `--fast`, provide the same summary from the generated phase files
before emitting the cook command. If a phase file is missing or cannot be read, report that
fact explicitly instead of fabricating a summary.

## Agents

| Agent           | Step | Modes                                                      |
| --------------- | ---- | ---------------------------------------------------------- |
| `researcher`    | 1    | `--auto` (×1), `--hard`/`--parallel`/`--two` (×2 parallel) |
| `planner`       | 2    | All                                                        |
| `plan-reviewer` | 3    | All except `--fast`                                        |

## Codex compatibility

Use the currently available Codex tools and skills for this workflow. If a referenced Claude agent, hook, MCP tool, or slash command is unavailable, perform the equivalent step inline, preserve the same artifact and verification requirements, and state the fallback briefly.
