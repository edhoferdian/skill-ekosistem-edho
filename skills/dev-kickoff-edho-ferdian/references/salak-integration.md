# Salak integration (optional, auto-detected)

Salak (`pip install salak` / `uv tool install salak`) is a separate tool — a
deterministic code knowledge graph generator that emits `repo-graph.json`.
This skill treats it exactly like the agent-harness case in
`references/agent-harness.md`: **detect, defer, never require.** If it isn't
installed, every phase runs exactly as if this file didn't exist.

## Step 1 — Detect (silent, every Mode A/C run)

Run `salak version` (or `command -v salak`). Non-zero / not found → Salak is
absent. Say nothing about it, proceed with every phase unchanged. This is not
a BLOCKER and not a gap to report.

## Step 2 — Read the contract from the installed binary, not from docs

A normal `pip install`/`uv tool install` does **not** bundle Salak's `docs/`
folder — only `salak/*.py` and `salak/schema/*.json` ship on PyPI. Do not
rely on finding `AI-CONSUMER-PLAYBOOK.md` or `cli-reference.md` inside the
installed package.

Instead, the sync mechanism is: **run `salak --help` and `salak <cmd> --help`
against whatever is actually installed, every time, and treat that output as
authoritative** — flags and exit codes are printed inline and ship with the
binary itself, so this stays correct across Salak versions without ever
touching this skill. Only if the current project happens to vendor Salak's
source (its `docs/` folder present in-repo, e.g. `docs/AI-CONSUMER-PLAYBOOK.md`)
is a markdown doc worth reading directly — treat that as a bonus and prefer it
when present, since it documents `emits`/provenance nuance `--help` won't.

## Step 3 — Verified exit-code contract — reconfirm via `--help` if in doubt

**Re-verify this table against the installed binary's `--help` output before
trusting it** — exit codes are a contract Salak has renumbered before (see
"why this table exists" below) and a locally installed version may predate or
postdate this note.

- `salak scan [PATH]` → writes `PATH/project-memory/repo-graph.json` (default;
  `--out` overrides).
  - **Exit 0** = clean.
  - **Exit 2** = completed, but a file failed to parse — the graph is still
    written and still usable; treat as success with a noted diagnostic, not a
    failure.
  - **Exit 3** = refused by the overwrite guard (§5): the new scan would drop
    the in-scope node count by more than 20% versus the previous graph. This
    is a deliberate regression guard — **never pass `--force` automatically**.
    Stop and tell the user rather than routing around it.
  - **Exit 1** = fatal — the command did not complete at all (crash, bad args,
    filesystem error).
- `salak check [PATH]` — **do not assume the numbering below is stable long
  term; it is the current one, confirmed against source.**
  - **Exit 0** = fresh. Trust the graph, say nothing.
  - **Exit 1** = internal error — no graph at the default path, or it's
    unreadable/schema-invalid, or the walk hit a filesystem error. The
    comparison never ran; this is not a staleness signal.
  - **Exit 2** = stale — the routine, expected non-zero result: the graph no
    longer matches the working tree. Not a crash. Print/summarize which
    files/rules changed; suggest `salak scan` to refresh, or answer with the
    caveat "graph may be out of date for X."
  - **Why this table exists as a table, not a one-liner:** Salak's own exit
    codes for `check` were renumbered once already (an early build shipped
    them inverted — `1` for stale, `2` for internal error — before being
    fixed to match `scan`'s "1 = fatal, 2 = completed with a finding"
    convention). Treating every non-zero `check` exit the same way
    mis-triages either way the numbering runs. If a locally installed
    `salak`'s `check --help` or behavior doesn't match the table above,
    believe the binary, not this file, and note the discrepancy in
    `02-gap-analysis.md`.
- `salak validate <path-to-repo-graph.json>` → confirms the file matches its
  own declared schema version before you trust it. Exit 0 = valid, non-zero =
  invalid/fatal — confirm exact codes via `--help` if this matters to the task.
- Any other failure (missing binary, crash, timeout, permission error): log
  one line, continue without the graph. Never let a Salak failure stop a
  kickoff or a task.

Two more subcommands exist with their **own, different** exit-code shapes —
don't apply the scan/check/validate table to either:

- `salak diff <old.json> <new.json>` — **Exit 0** = identical graphs. **Exit
  1** = real, reported differences (not an error — the comparison succeeded).
  **Exit 2** = load error, one or both files unreadable/invalid — the
  comparison never ran. Useful in REVIEW: diff the graph from before a task
  against a fresh scan after it, to see the actual structural change instead
  of trusting a self-report.
- `salak doctor` — **Exit 0** = every language adapter healthy. **Exit 1** =
  at least one adapter unhealthy (this only reflects Salak's own ability to
  parse the repo, not whether an optional language server is present/current
  — that's reported informationally and never changes the exit code). Worth
  running once at Step 1 detection if `scan`/`check` behave unexpectedly.

## Step 4 — When to run it

- **Mode A / Mode C**, after Phase 0 classification: if
  `project-memory/repo-graph.json` doesn't exist yet, run `salak scan` once.
  This is a local, non-destructive action (writes one JSON file) — run it the
  same way VERIFY already runs lint/tests, without asking permission, unless
  the 20%-drop guard fires (exit 3 — see above), in which case stop and ask.
- **Before Phase 3 REVIEW/VERIFY** on any task touching pre-existing code
  (not a from-scratch file): run `salak check` first. Exit 2 (stale) →
  refresh with `salak scan` before relying on graph-derived claims such as
  "nothing else imports this module." Exit 1 (internal error) → don't
  refresh, don't trust the graph this round, fall back to reading source
  directly, move on.
- **Mode B** (handoff pack only): still generate/refresh the graph if Salak
  is present — it's worth handing off — but don't let it delay or block
  Mode B's early exit after Phase 2.

## Step 5 — What it's used for once present

- **Phase 0 cross-document validation (Axis 6 — doc vs. existing repo)**:
  check `ARCHITECTURE`-role documents' claimed dependencies against
  `repo-graph.json`'s `depends_on` / `imports` edges. A mismatch is a
  consistency-axis finding to report, not something to resolve silently.
- **Phase 2 Context Pack**: if a graph is present, add a short "Consulting the
  Salak graph" habit to §F of every generated CLAUDE.md/AGENTS.md variant —
  run `salak check .` before trusting a dependency/import answer, read the
  fact off the graph instead of a partial grep. If the repo carries its own
  `docs/AI-CONSUMER-PLAYBOOK.md`, quote its Step 4 snippet verbatim instead of
  writing a new one.
- **Phase 3 IMPLEMENT / REVIEW**: prefer a fact read from `repo-graph.json`
  ("what else imports this file") over inference from reading a handful of
  files by hand.
- **Respect Salak's own provenance tagging** (`extracted` > `inferred` >
  `ambiguous`). Never treat an `ambiguous` edge as settled fact — it's a lead
  to double-check, not a citation. And never read a zero-count edge kind
  (e.g. `calls`, which no Salak build tracks at all) as "this has no callers"
  — check that language's `adapters[].emits` first; a kind absent from
  `emits` was never trackable, so its absence proves nothing.

## Known open gap — not yet closed

This reference has not been dry-run against an actual project with Salak
installed inside this skill's own PLAN→TEST→IMPLEMENT→REVIEW→VERIFY loop —
only the CLI/source contract above was checked directly. Whether the Phase
0/Phase 3 wiring above reads naturally mid-task, and whether the 20%-drop
guard message is easy to act on in practice, is untested. Treat this
integration as unverified-in-context until it's run once for real.
