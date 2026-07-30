---
name: cook
description: "Implement a planned feature phase by phase. Use when the user says \"cook this\", \"implement it\", \"let's build\", \"start coding\", or passes a plan.md path. Spec-aware — auto-loads spec.md alongside plan for SDD+TDD. Modes (pick one): --fast (skip test/review), --hard (mandatory human approval). Composable flag: --tdd (write failing tests before implementing)."
---

# cook — Structured Implementation Pipeline

Modes — mutually exclusive, pick one (default = Standard):
- **Standard** — test + review, auto-approve if score ≥ 9.5 with 0 CRITICAL
- **`--fast`** — skip tester and code-reviewer; git-manager only in Step 5
- **`--hard`** — mandatory test + mandatory review, no auto-approve
- **`--parallel`** — phases have exclusive File Ownership (from `plan --parallel`); auto-continue between phases (no per-phase review gate), full test + review at end

Composable flag — combine with any mode:
- **`--tdd`** — write failing tests first, then implement until they pass

**Flag default** (no flag given): `--tdd` is off — standard test behavior applied.

---

### Step 0 — Plan Check

When no plan path provided:
1. Search `plans/` for any `plan.md` → ask: "Found `{path}`. Use this? [Y/n]"
2. If none found → ask: "No plan found. Continue anyway? [y/N]" — if No, suggest `$plan`

After resolving plan path: check for `spec.md` in the same directory. If found, load it — activates **spec-driven mode** for Steps 1 and 2.

---

### Step 1 — Load Plan / Detect Mode

After loading plan.md, determine the execution mode:

1. If the user supplied a mode flag (`--fast`, `--hard`, `--parallel`), use it — **user flag always wins**.
2. If no mode flag was supplied, scan plan.md for a line matching `^Risk: (tiny|normal|high-risk)\b` (anchored to line start, word boundary after tier — avoids false matches on Risks section bullets, tolerates trailing reason text like `Risk: high-risk — touches auth`). Apply the lane mapping:
   - `tiny` → `--fast`
   - `normal` → Standard
   - `high-risk` → `--hard`
3. If no `Risk:` line is found (plan written before Phase 1), fall through to Standard.
4. If the user explicitly passed `--fast` and the plan carries `Risk: high-risk`, print one non-blocking warning before proceeding: `[WARN] --fast override on high-risk plan — skipping tests and review.`

Report what will be cooked:

```
Plan: {Feature Name}
Status: {status from plan.md}
Risk Lane: {tiny|normal|high-risk|none} → Mode: {Standard|Fast|Hard} ({default|user override: --flag})
Test:  {default | --no-test | --tdd}
Spec:  {plans/{slug}/spec.md — N P1 stories, N success criteria | none}
Phases remaining:
  [ ] Phase 1: ...
  [ ] Phase 2: ...
```

If spec loaded + `--tdd` not set:
`Spec detected. Consider --tdd: acceptance criteria in spec.md are ready-made test anchors.`

If `## Session Notes` exists in plan.md: output resume state and continue from where it left off.

When no plan file provided: read the feature request, ask 2–3 clarifying questions, proceed once clear.

---

### Step 2 — Implement

For each `phase-XX-*.md` in order:

1. Read phase file — understand requirements, architecture, steps, success criteria
2. **Feature state → active**: locate `feature_list.json` at project root (same dir as `.ck.json`). If it exists and has an entry whose `id` matches the current phase slug, set `status: "active"`. Skip silently if file absent or no matching entry — never let this block implementation. If JSON is malformed, log one warning and skip. **`--parallel` mode**: reads and writes to `feature_list.json` must be serialized — always re-read the file immediately before writing to avoid overwriting concurrent phase updates (read-modify-write per update, not cached reads).
3. Implement following codebase conventions
4. Verify success criteria for the phase
5. **Feature state → passing** (immediately after verification — do not defer to Step 5):
   - Standard / `--hard`: set `status: "passing"`, `evidence`: one-line summary of verification output (e.g. `"Tests: 12 passed, 0 failed"`)
   - `--fast`: set `status: "passing-unverified"`, `evidence`: `"fast mode — no test evidence"`
   - Write the updated entry back to `feature_list.json` at project root before moving to the next sub-step.
6. **If spec loaded**: `P1 coverage: {N}/{total} stories addressed this phase`
7. Write (overwrite) `## Session Notes` in plan.md, then mark phase complete `- [x] Phase N: {name}`
8. Report what was done

**Session Notes template** (overwrite, never append):

```markdown
## Session Notes
<!-- Updated by cook automatically — do not edit manually -->

**Last active:** {YYYY-MM-DD HH:MM}
**Phase in progress:** {phase-XX-name}
**Status:** {one-line status}

### Decisions made this session
{bullet list of non-obvious decisions, or "(none)"}

### Next immediate action
{what cook will do next}
```

**Review Gate** — after each phase:
- **Standard / `--hard`**: pause and wait for user approval
- **`--fast`** / **`--parallel`**: continue automatically

Stop if: success criterion unverifiable, unexpected blocker, or phase needs user decisions not in the plan.

---

### Step 3 — Test (tester sub-agent)

**`--fast`**: skip → Step 3.S.

**[Build Gate]**: verify compilation before tests. On failure: `[GATE FAIL] Build gate: compilation errors — fix before testing.`

**Default**: spawn **`tester`** → writes tests, runs full suite (100% pass required) → on failure: spawn **`debugger`** → fix → re-test.

**Keep/Discard prompt** (after `tester` reports PASS, default mode only — not `--tdd`, not `--fast`):

1. Before spawning `tester`, snapshot `PRE_TRACKED` (`git ls-files`) and `PRE_DIRS` (`find <repo-root> -type d`, excluding `.git`/`node_modules`). `<repo-root>` via `git rev-parse --show-toplevel`.
2. After `tester`'s `Test files written:` list comes back: reject any path whose `realpath` doesn't start with `<repo-root>`. Eligibility is decided by the orchestrator, never by `tester`'s say-so — a path already in `PRE_TRACKED` is never eligible, regardless of what `tester` reported.
3. Append eligible paths to `plans/scratch-tests.json` as `{path, phase, createdAt, status: "pending"}` (create as `[]` if missing; if existing JSON fails to parse, rename to `.corrupt-<timestamp>` and start fresh — never abort or silently overwrite).
4. `ask the user directly`: keep or discard, as one batch covering all eligible files this phase.
5. **Keep**: mark entries `"kept"`, proceed normally.
6. **Discard**: per file — `rm`, mark that entry `"discarded"` and persist the ledger immediately (not batched), then walk up from its parent directory toward `<repo-root>`, stopping at the first ancestor present in `PRE_DIRS` (never touch it or above); remove each directory below that point only if it is now empty AND was absent from `PRE_DIRS`.

**Remediation cycles**: each of cycles 1–3 must use a different approach than previous. Cycle 4: STOP.

```
[ESCALATION] Test remediation exhausted
File:    {path/to/failing_test}
Error:   {exact error message}
Cycles:  {approach 1} | {approach 2} | {approach 3}
Action:  Awaiting user guidance
```

**`--tdd`**: invert per phase:
1. `tester` writes failing tests (red) — from `### Tests to Write First` or spec acceptance criteria
2. Confirm red before implementing
3. Implement until green, full suite passes

---

### Step 3.S — Auto-Simplify

Check if `SIMPLIFY_TRIGGERED` in context (emitted by `code-simplifier` hook).

If triggered: invoke `simplify` skill on files edited this phase → delete simplify tracker → proceed to Step 4.
If not triggered: skip silently.

Thresholds (`.ck.json` → `simplify.threshold`): `totalLoc` 400, `fileCount` 8, `singleFileLoc` 200.

---

### Step 4 — Code Review

**`--fast`**: skip → Step 5.

**`--parallel`**: run code review across all phases at once (not per-phase).

**[Test Gate]**: all tests must pass (or `--fast` set).

Spawn **`code-reviewer`**: correctness, security, regressions, quality → APPROVED / WARNING / BLOCK.
- **Standard**: auto-approve if score ≥ 9.5 with 0 CRITICAL
- **`--hard`**: no auto-approve — human must approve before Step 5
- Fix/re-review up to 3 cycles (different approach each), then escalate

---

### Step 5 — Finalize (MANDATORY)

**[Approval Gate]**: code-reviewer APPROVED required (or `--fast` bypass).

**`project-manager`** (skip `--fast`): mark phases `[x]`, update plan status.

**Feature state consistency check** (skip `--fast`): read `feature_list.json` at project root if it exists. Log each feature's final status — this is read-only; Step 2 already owns all state writes. If any entry is still `active` (cook was interrupted mid-phase), log a warning: `[WARN] feature {id} still active — verify manually before $handoff`.

**`docs-manager`** (skip `--fast`): update docs, README, API contracts.

**If spec loaded**: output before git-manager:
```
# Spec Coverage
P1 stories:        {N}/{total} covered
Success criteria:  {N}/{total} verifiable
Uncovered P1:      {list any, or "none"}
```

**`git-manager`** (always): conventional commits → ask to push.

---

## Agents

| Agent / Skill     | Step | Modes |
|-------------------|------|-------|
| `tester`          | 3    | Standard, `--hard`, `--parallel` (skip for `--fast`) |
| `debugger`        | 3    | When tests fail |
| `simplify` skill  | 3.S  | All (hook-driven) |
| `code-reviewer`   | 4    | Standard, `--hard`, `--parallel` (skip for `--fast`) |
| `project-manager` | 5    | Standard, `--hard`, `--parallel` (skip for `--fast`) |
| `docs-manager`    | 5    | Standard, `--hard`, `--parallel` (skip for `--fast`) |
| `git-manager`     | 5    | Always (mandatory) |

## Codex compatibility

Use the currently available Codex tools and skills for this workflow. If a referenced Claude agent, hook, MCP tool, or slash command is unavailable, perform the equivalent step inline, preserve the same artifact and verification requirements, and state the fallback briefly.
