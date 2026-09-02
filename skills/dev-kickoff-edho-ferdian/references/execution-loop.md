# Execution Loop, Snapshot & Recovery

Reference for Phase 3 and Phase 4 of dev-kickoff-edho-ferdian v2.0.

Every task runs: **PLAN → TEST → IMPLEMENT → REVIEW → VERIFY → REMEMBER.**

---

## Stage 1 — PLAN

1. Restate: "Task [id] — [name]. Sprint [n]. Dependencies: [status].
   Acceptance criteria: [from WORK_PLAN; if absent, derive from BEHAVIOR_SPEC
   / PRODUCT_INTENT and label DERIVED]."
2. Dependency pre-check. Unmet dependency → refuse this task, offer the
   nearest unblocked one.
3. State the files you intend to create or modify, and the shape of the change
   in 3–6 lines. For anything beyond a trivial edit, get the user's "lanjut"
   before writing code. A wrong plan caught here costs one message; caught at
   VERIFY it costs the whole task.
4. Big-task check: if this clearly needs more than one session, split it now
   and say so.

## Stage 2 — TEST (write the failing test first)

**What TEST is not:** it is not a review of the plan. The plan is checked
inside Stage 1 (steps 3–4) and, for high-risk work, again in REVIEW. TEST is
where the acceptance criteria stop being prose and become something executable
that can fail. Prose can be satisfied by an agent's opinion; a failing test
cannot.


Write the test that encodes the acceptance criteria, run it, and **show the
RED output**. A test that has never failed proves nothing.

Escape hatches — allowed, but each must be named in the Reflection block with
its reason:

| Situation | What to do instead |
|-----------|--------------------|
| No test infrastructure exists yet | Make "set up the test harness" the current task, then return |
| Task is pure config / scaffolding / docs | Skip TEST; define a concrete manual verification step instead |
| Visual or layout work | Skip automated TEST; define the visual acceptance check and screenshot/inspection step |
| Test framework not chosen (PDR §3 silent) | STOP — this is a binding decision; ask the user |

"There's no time" and "it's obvious" are not on this list.

## Stage 3 — IMPLEMENT

Real code until the test passes. No pseudocode, no `// implement later`.
Follow PDR §3 conventions. Do not edit the test to make it pass — if the test
is wrong, say so explicitly and fix it as a separate, stated step.

If implementation forces a decision not in the PDR → **STOP**, propose it with
options and a recommendation, wait for approval, record it in
`01-decision-register.md`, then continue. The register is living, not frozen.

## Stage 4 — REVIEW (fresh context)

The reviewer must not reuse the implementer's reasoning. If subagents are
available, run the review as a separate agent; otherwise role-play it
deliberately and say which you did.

Review targets, in order: does it meet the acceptance criteria · does it
violate any PDR decision, non-goal, or domain rule · how does it fail or get
abused · is there a simpler correct implementation · error handling and edge
cases · hardcoded secrets.

**Critique-Correction Loop (CCL).**
- **Auto-trigger** for HIGH-RISK tasks: auth, payments, database migrations,
  personal-data handling, credit/quota logic, anything touching permissions.
  Opt-in otherwise.
- **Critic** attacks; **Corrector** accepts or rejects each critique with
  reasoning and revises.
- **Hard cap: 2 rounds.** Stop early on convergence. Only
  correctness/security/behavior disputes justify round 2 — never style.
  Unresolvable disagreement → surface both views with a recommendation; do not
  force a fake resolution.
- End state: RESOLVED or ACCEPTED-WITH-NOTES.

## Stage 5 — VERIFY (tool output or it didn't happen)

Run the real tooling: build, linter, type-checker, and the full relevant test
suite — not just the new test. Paste the outcome.

Confidence labels: tool-confirmed → **[High confidence]**; sound reasoning,
not tool-verified → **[Medium confidence]**; depends on runtime data or
external behavior you can't see → **[Low confidence] — needs verification**.

Never fabricate tool output. If a tool can't run in this environment, say so
plainly and mark the task `VERIFIED: NO — [reason]`. An unverified task may
not be reported as done.

Regression rule: if the full suite was green before this task and is not green
now, the task is not done, regardless of whether the new test passes.

## Stage 6 — REMEMBER

In the same turn, not "later":

1. Update `03-progress.md`: ledger (done / in-progress / next) + new snapshot
   at the top.
2. Update `01-decision-register.md` if a decision was added or changed.
3. Move any resolved open question in `02-gap-analysis.md` to resolved, dated.
4. **Instincts** — if this task produced a lesson that will apply again
   (a repo-specific gotcha, a pattern that failed, a convention the docs
   never stated), append it to `04-instincts.md`:

```
| # | Instinct | Trigger (when it applies) | Evidence (task id) | Confidence |
|---|----------|---------------------------|--------------------|------------|
| I3 | Supabase RLS policies must be added in the same migration as the table | any new table | T-018 | High |
```

Rules for instincts: one real observation each, tied to a task id. No generic
engineering advice — "write clean code" is not an instinct. Three or more
related instincts pointing the same way → propose promoting them into PDR §3
conventions, where they become binding.

## Per-task Reflection (8 gates)

```
CATATAN REFLEKSI — TASK [id]
Gate 1: All acceptance criteria met?                          [.]
Gate 2: Test written before implementation (or escape hatch
        named with reason)?                                   [.]
Gate 3: PDR §3 conventions followed?                          [.]
Gate 4: No NON-GOALS territory touched?                       [.]
Gate 5: Error handling & edge cases present?                  [.]
Gate 6: No hardcoded secrets/credentials?                     [.]
Gate 7: Full verification run — real tool output, not eyeball?[.]
Gate 8: Memory files + snapshot updated this turn?            [.]
```

Any FAIL → fix before moving to the next task, or record it explicitly as
accepted debt in `02-gap-analysis.md`.

## EDHO SCAN — per sprint

- [ ] Task order followed the plan, or deviations were logged + approved
- [ ] No task started before its dependencies finished
- [ ] Every new decision entered `01-decision-register.md`, not just chat
- [ ] Every HIGH-RISK task went through CCL
- [ ] Every completed task has real verification output behind it
- [ ] Progress Ledger matches reality
- [ ] Memory files (01/02/03/04) are not lagging behind actual state

## Anti-pattern detection (warn the user when detected)

| Anti-pattern | Signal |
|--------------|--------|
| Sprint-skipping | Asked for a sprint-5 task while sprint 2 is unfinished |
| Silent scope creep | A "small" feature not in any source doc requested mid-task |
| Register drift | Code deviating from PDR decisions without updating the register |
| Snapshot debt | >3 tasks completed without a fresh snapshot |
| Big-bang task | One task that clearly needs >1 session — split before starting |
| Test theatre | Tests written after the code, or a test that never failed |
| Green-by-deletion | Suite passes because a test was weakened or removed |
| Verification bypass | Task marked done with no tool output |

---

## Phase 4 — Session Snapshot format (English, < 1 page)

```
# SESSION SNAPSHOT — [Project Name]
Snapshot #: [n] | Timestamp: [date/time] | Mode: [A/B/C]

## STATE
- Sprint [n]/[total], Task [id] — [status: done / in-progress at stage X]
- Tasks completed this session: [ids]
- Files written: [paths + one-line purpose each]
- Last verification: [build/lint/test result + when]

## DECISIONS ADDED/CHANGED SINCE LAST SNAPSHOT
- [Dx: decision — rationale]  (or "none")

## INSTINCTS ADDED
- [Ix: ...]  (or "none")

## OPEN THREADS
- [pending questions/decisions, if any]

## RESUME COMMAND
Filesystem: open the repo, read CLAUDE.md + /project-memory/, continue from
the task above at stage [X]. Chat: paste this snapshot + the Execution Context
Pack into a new session and say "resume".
```

Snapshot triggers: task completed · sprint end · new decision recorded · any
interruption you can anticipate. Latest snapshot always at the top of
`03-progress.md`; in pure chat, also emit it as a paste-ready block.

## Resume protocols (Mode C)

**Path 1 — Filesystem (Claude Code / Cursor / repo access):**
1. Read `CLAUDE.md`/`AGENTS.md` + all of `/project-memory/` — primary source
   of truth. A pasted snapshot is supplementary.
2. Cross-check `03-progress.md` against the actual repo: do the claimed files
   exist, does the build still pass, does the test suite still pass?
   Discrepancy → report it; **reality wins**, log it in the next snapshot.
3. Resume mid-loop, not mid-air: if the last task stopped after IMPLEMENT,
   restart it at REVIEW, not at PLAN and not at the next task.
4. Confirm: "Resuming Task [id] at stage [X]. Benar?" → continue Phase 3.

**Path 2 — Pure chat (no filesystem):**
1. Require BOTH the snapshot and the Context Pack. Snapshot alone is state
   without decisions — blind execution. Ask for the missing piece.
2. Sanity-check: does the in-progress task exist in the plan? Is the snapshot
   number plausible in sequence?
3. Reconstruct the Progress Ledger, confirm the resume point and stage,
   continue Phase 3.

**State-corruption rule:** whenever records and reality conflict — reality
wins, and the discrepancy is written into the next snapshot so the history
stays honest.
