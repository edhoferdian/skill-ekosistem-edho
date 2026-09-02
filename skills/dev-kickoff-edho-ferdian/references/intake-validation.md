# Intake, Role Mapping, Cross-Document Validation & Decision Register

Reference for Phase 0 and Phase 1 of dev-kickoff-edho-ferdian v2.0.

---

## Step 0.1 — Discover the documents

Look in this order and stop when you have a coherent set:

1. Uploaded files in this session.
2. `architecture-docs/[Product_Name]/` — the Architect V1.2 output folder.
3. `docs/`, `spec/`, `specs/`, `rfc/`, `adr/`, repo root (`README.md`,
   `ARCHITECTURE.md`, `openapi.yaml`, `schema.prisma`, `*.sql`).
4. `/project-memory/` — if present, you are in Mode C, not Mode A.
5. Anything the user pastes into chat.

**Read before classifying.** Never infer a document's type from its filename;
misnamed files are common and one mislabel corrupts the whole intake.

## Step 0.2 — Handle Architect V1.2 per-phase files

The V1.2 Architect tools emit one `.md` file per approved phase plus a
`_MANIFEST.md`, then a final consolidated document:

```
architecture-docs/[Product_Name]/
  _MANIFEST.md
  SRS_[Product_Name]_Phase1_System_Overview_Context.md
  SRS_[Product_Name]_Phase2_Use_Cases_Data_Behavior.md
  SRS_[Product_Name]_v1_0.md          ← final
```

Precedence rules:

- **`_MANIFEST.md` is read first.** It gives the phase list, versions,
  `status` (draft/locked), `lang`, ID ranges, and the still-open OPEN ITEMS.
  Those open items go straight into PDR §6 — do not re-derive them.
- **A final `v[X_Y].md` supersedes its phase files.** Read the final document
  as the source of truth; use phase files only to resolve ambiguity or to
  recover DECISIONS / CARRY-FORWARD blocks the final document dropped.
- **No final document yet** → assemble from phase files in order, and flag
  every phase with `status: draft` as unstable input (WARNING, not BLOCKER).
- **A phase file listed in the manifest but missing on disk** → BLOCKER. The
  reconciliation the Architect promised did not happen.
- `Product_Name` must be identical across all documents. A mismatch
  (`Jeruk_AI` vs `JerukAI`) is a BLOCKER — it means two pipelines got mixed.

## Step 0.3 — Role mapping

Classify every document into one or more roles. Show this matrix — it replaces
the fixed five-document matrix from v1:

```
DOCUMENT INTAKE MATRIX
| Document (as found)              | Role(s)                    | Ver  | Lang | Notes                    |
|----------------------------------|----------------------------|------|------|--------------------------|
| PRD_Jeruk_AI_v1_2.md             | PRODUCT_INTENT             | v1.2 | id   | final, locked            |
| SRS_Jeruk_AI_v1_2.md             | BEHAVIOR_SPEC              | v1.2 | id   | 47 FR / 12 NFR           |
| SDD_Jeruk_AI_v1_2.md             | ARCHITECTURE               | v1.2 | id   | —                        |
| WBS_Jeruk_AI_v1_0.md             | WORK_PLAN                  | v1.0 | id   | 66 tasks / 9 sprints     |
| openapi.yaml                     | BEHAVIOR_SPEC, ARCHITECTURE| —    | en   | 23 endpoints             |
| (none)                           | UX_SPEC                    | —    | —    | MISSING → UI tasks will  |
|                                  |                            |      |      | need per-screen decisions|

ROLE COVERAGE
PRODUCT_INTENT ✅ · BEHAVIOR_SPEC ✅ · ARCHITECTURE ✅ (mandatory)
UX_SPEC ❌ · WORK_PLAN ✅ (mandatory) · OPS_CONSTRAINTS ⚠ partial (repo lint config only)
```

**Mandatory roles for Mode A: `ARCHITECTURE` and `WORK_PLAN`.**

Missing mandatory role → offer, in this order:
1. User uploads the missing document.
2. User runs the upstream Architect tool (SDD Architect / Task Breakdown
   Architect) first.
3. You derive a **provisional** artifact, clearly labelled
   `[DERIVED — NOT APPROVED]`, presented for approval before any execution.
   Every derived task carries `source: DERIVED` so it can be audited later.

Missing non-mandatory role → allowed. State the concrete impact, e.g. "no
UX_SPEC → screen layout decisions will surface as OPEN QUESTIONS during UI
tasks, expect more interruptions."

## Step 0.4 — Language detection

Read `lang:` from frontmatter, or `_MANIFEST.md`, or infer from the body. Set
`doc_lang`. If documents disagree on language, that is a WARNING: the
Architect V1.2 rules forbid mixing languages within one pipeline, so a
mismatch usually means two different pipeline runs got combined. Ask which one
is current. Record `doc_lang` and `artifact_lang` in PDR §8.

## Cross-consistency checklist (minimum 4 axes, by role)

| # | Axis | Question | Typical finding |
|---|------|----------|-----------------|
| 1 | ARCHITECTURE ↔ WORK_PLAN | Do task types and estimates make sense for the declared stack? | Plan assumes a service the architecture never defines |
| 2 | PRODUCT_INTENT ↔ BEHAVIOR_SPEC | Does every feature have at least one requirement? | Orphan feature, no requirements |
| 3 | UX_SPEC ↔ WORK_PLAN | Every screen has a task; every UI task has a screen? | Screen designed but never scheduled |
| 4 | Non-goals ↔ WORK_PLAN | Any task building declared out-of-scope work? | Scope creep baked into the plan |
| 5 | BEHAVIOR_SPEC ↔ ARCHITECTURE | Any requirement with no architectural home? | NFR (rate limiting) with no owning component |
| 6 | Any doc ↔ existing repo | Does the code already contradict the spec? | Spec says Postgres, repo runs SQLite |
| 7 | ID integrity | Do cross-references resolve (FR-012 cited by a task actually exists)? | Dangling ID after a doc revision |

Axis 6 applies whenever there is an existing repo — it is the axis most often
skipped and the one that produces the most expensive surprises.

**Findings block format:**

```
CONSISTENCY FINDINGS
[BLOCKER] B1 — WBS task T-041 implements social login; PRD §7 lists
          third-party auth as a non-goal. Decide: drop task or amend PRD.
[WARNING] W1 — UIX screen "Empty State — History" has no task.
[NOTE]    N1 — SRS uses "credits", SDD uses "PULP". Same concept; will
          standardize on "PULP" in code. Confirm.
```

- **BLOCKER** = execution would violate a source document. User decides before
  Phase 1.
- **WARNING** = gap that will surface mid-execution. Proceed; log in
  `02-gap-analysis.md`.
- **NOTE** = terminology or minor drift. State the normalization you'll apply.

---

## Phase 1 — Project Decision Register (PDR) format

Register content is **English** (it is machine-facing). Every claim needs a
source: document + section.

```
# PROJECT DECISION REGISTER — [Project Name]
Generated: [date] | Sources: [doc, role, version — one line each]

## 1. BINDING TECHNICAL DECISIONS
| # | Decision | Source | Rationale (1 line) | Reversal cost |
|---|----------|--------|--------------------|---------------|
| D1 | Next.js 15 App Router | SDD §2.1 | ... | HIGH |

## 2. STACK & VERSIONS
Exact versions only. "latest" is not a version.

## 3. CODING CONVENTIONS
Naming, directory structure, error-handling pattern, commit format, test
framework and test file location. If the sources don't define one, propose it
and mark [PROPOSED] until the user confirms.

## 4. NON-GOALS
Copied verbatim from PRODUCT_INTENT — not paraphrased into vagueness.

## 5. DOMAIN RULES
Business logic that must never be violated — credit deduction, quota limits,
permission boundaries, data retention.

## 6. OPEN QUESTIONS
Everything the documents don't answer, plus unresolved OPEN ITEMS carried from
_MANIFEST.md. Do NOT answer these yourself.
| # | Question | Blocks task(s) | Source | Status |

## 7. RISK REGISTER
Top 5 execution risks + one-line mitigation each.

## 8. LANGUAGE & ARTIFACT CONTRACT
doc_lang: [id|en] · artifact_lang: [en by default]
Which outputs follow which, and any user override recorded as a decision.
```

**Rules:**
- Decision without traceable source → OPEN QUESTIONS, not section 1.
- `Reversal cost: HIGH` decisions may not change mid-execution without
  explicit user confirmation — re-checked in Phase 3.
- OPEN QUESTIONS are a feature: six honest questions beat six hidden
  assumptions.
- Derived content (from a missing mandatory role) is tagged `[DERIVED]`
  everywhere it appears, permanently.
- Intent Preservation check before presenting: does the register still reflect
  the original product intent, or has interpretation drifted?

**Done criteria Phase 1:** all 8 sections filled · every decision sourced ·
user confirms the register.
