# Agent Roster & Harness Integration

Reference for Phase 2 of dev-kickoff-edho-ferdian v2.0.

Goal: give the project a small set of **scoped agents** that map onto the
six-stage loop, written in the layout the detected harness actually reads —
without duplicating a harness the user has already installed.

---

## Step 1 — Detect the harness BEFORE writing anything

Check the repo and the user's environment for:

| Signal | Means |
|--------|-------|
| `.claude/agents/`, `~/.claude/agents/`, `CLAUDE.md` | Claude Code |
| `.claude-plugin/`, plugin `ecc@ecc`, `.ecc/`, `RULES.md` + `skills/` at root | ECC installed |
| `.cursor/rules/`, `.cursorrules`, `.cursor/agents/` | Cursor |
| `AGENTS.md`, `.codex/` | Codex / OpenCode / generic |
| `.github/copilot-instructions.md`, `.github/prompts/` | Copilot |
| none of the above | pure chat → emit paste-ready blocks |

**Rule: an installed harness wins.** If ECC (or any comparable system) is
present, do **not** write a competing planner/reviewer/TDD instruction set.
Two systems giving the agent different definitions of "review" is worse than
having one. Instead:

- Write the **project-specific** layer only: the Decision Register, non-goals,
  domain rules, conventions, stack versions, and the current task pointer.
- Map this skill's stages onto the harness's own surfaces and say so
  explicitly in CLAUDE.md, e.g. plan → the harness's planning workflow, test →
  its TDD workflow, review → its fresh-context review agent, verify → its
  quality gate. Verify those surfaces exist before referencing them; do not
  cite commands from memory.
- Record the mapping as a decision in `01-decision-register.md` so a future
  session doesn't re-litigate it.

If no harness is installed, this skill supplies the roster itself (Step 2).

## Step 2 — The roster (only when no harness owns these roles)

Six core agents, one per stage, plus optional stack reviewers. Keep each
definition short — a long agent file is a context tax paid on every
invocation.

| Agent | Stage | Scope | Must NOT |
|-------|-------|-------|----------|
| `planner` | PLAN | Read specs + memory, produce a task plan with acceptance criteria and file list | Write code |
| `test-author` | TEST | Write failing tests from acceptance criteria | Change implementation to make tests pass |
| `implementer` | IMPLEMENT | Make the failing test pass, follow PDR §3 | Modify or weaken the test |
| `reviewer` | REVIEW | Fresh-context critique: correctness, security, spec conformance | See the implementer's reasoning |
| `verifier` | VERIFY | Run build/lint/types/tests, report raw tool output | Fix anything it finds — it reports |
| `memory-keeper` | REMEMBER | Update project-memory, write the snapshot, propose instincts | Invent progress not backed by tool output |

Add stack-specific reviewers **only when the stack warrants it** (a database
reviewer for heavy schema work, a security reviewer when the project handles
payments or personal data). Every extra agent is a maintenance cost; a roster
nobody invokes is dead weight.

**The separation that does the work:** `reviewer` must not inherit the
implementer's context, and `verifier` must not be allowed to fix what it
finds. Those two boundaries are where most of the value comes from — a
reviewer that already believes the code is correct reviews nothing.

## Step 3 — Write the roster into the right layout

| Target | Path | Format |
|--------|------|--------|
| Claude Code | `.claude/agents/<name>.md` | frontmatter `name`, `description`, `tools` + body |
| Cursor | `.cursor/agents/<name>.md` (build-dependent) | keep short; Cursor loading behavior varies |
| Codex / generic | `AGENTS.md` section per role | one file, clearly sectioned |
| Copilot | `.github/prompts/<name>.prompt.md` | prompt files only |
| Pure chat | one paste-ready block per agent | user stores them |

Before writing: check the path for an existing definition and MERGE, marking
edits with `<!-- updated by Dev Kickoff [date] -->`. Never overwrite an agent
someone else wrote.

Agent definitions are always **English** (`artifact_lang`), whatever
`doc_lang` is.

## Step 4 — Agent content template

```
---
name: <role>
description: <one line — when this agent should be used>
---

# <Role> — [Project Name]

## Scope
<what this agent does, one paragraph>

## Project constraints (from the Decision Register)
- Stack: <exact versions>
- Conventions: <pointer to PDR §3, plus the 3 rules most often violated>
- NON-GOALS: <verbatim list — hard boundaries>
- Domain rules that must never be violated: <list>

## Inputs
project-memory/01-decision-register.md · 03-progress.md · <task id>

## Output contract
<exactly what this agent returns, and in what shape>

## Hard limits
- <the "must not" from the roster table>
- Any new binding decision → stop and ask the user; never decide silently.
```

## Step 5 — If the user wants ECC itself

ECC (`github.com/affaan-m/ECC`, MIT) is an external agent-harness system with
its own agents, skills, hooks, and memory vault. This skill does **not**
bundle, vendor, or reimplement it.

If the user wants ECC in the project:
- Point them at the official install path for their harness and let them run
  it themselves — installation touches their global config, and stacking
  install methods is the documented way to break it.
- After it is installed, re-run Phase 2: detect it, defer to it, and write
  only the project-specific layer.
- Treat ECC's counts, command names, and install commands as
  **verify-before-quoting**. The project moves fast; do not state a command
  from memory. Check the repo docs at the time of use.

Attribution: if you adopt ECC's loop or file layout in project docs, credit it
by name and link. It is MIT-licensed — attribution is cheap and correct.

## Reflection add-on for the roster

Fold these into the Phase 2 Reflection block:

```
Gate R1: Harness detected and deferred to (or absence confirmed)?  [PASS/FAIL]
Gate R2: No agent duplicates a workflow the installed harness owns? [.]
Gate R3: reviewer isolated from implementer context?                [.]
Gate R4: verifier has no fix authority?                             [.]
Gate R5: Every agent file merged, not overwritten?                  [.]
```
