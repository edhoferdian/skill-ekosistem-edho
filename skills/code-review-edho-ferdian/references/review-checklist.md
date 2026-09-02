# Review Checklist, Severity, Report & Fix Formats

This is the detailed reference for Phases 1 and 5. Read it when you reach those
phases. Table of contents:

1. Domain 1 — Code Quality (CQ)
2. Domain 2 — Security (SEC)
3. Domain 3 — Performance (PERF)
4. Domain 4 — Blueprint / Consistency (BC)
5. Severity system & scoring
6. Report format (Phase 5)
7. Adaptive fix: Full Rewrite vs Patch
8. Changelog + clean-code standard + anti-hallucination guard
9. Saved `.md` report structure
10. Special scenarios

---

## 1. Domain 1 — Code Quality (CQ)

- **CQ-01 Single Responsibility** — does each function/component do exactly one thing?
- **CQ-02 Naming** — descriptive, non-misleading names? (avoid `data`, `temp`, `x`, `val`, `res`)
- **CQ-03 Hardcoded values** — values that belong in env/constants?
- **CQ-04 DRY** — repeated logic that should be abstracted into a reusable unit?
- **CQ-05 Error handling** — every async op has try/catch or a proper handler?
- **CQ-06 Typing** — types correct, no needless `any`? (where applicable)
- **CQ-07 Dead code** — unused code, imports, or variables?
- **CQ-08 Comments** — explain *why*, not *what*? Over/under-commented?
- **CQ-09 Consistent style** — formatting/style consistent across the file?
- **CQ-10 Anti-patterns** — stack-specific anti-patterns present?
- **CQ-11 Edge cases** — null, undefined, empty collections, boundaries handled?
- **CQ-12 Magic numbers** — unexplained numbers/strings that should be named constants?

## 2. Domain 2 — Security (SEC)

- **SEC-01 Input sanitization** — all user input sanitized before use/storage?
- **SEC-02 Secret exposure** — API keys, passwords, tokens in code? (verify with a secret scanner in Phase 2)
- **SEC-03 Auth check** — every protected route/function checks authn/authz?
- **SEC-04 Injection** — SQL injection, XSS, or command injection risk?
- **SEC-05 IDOR** — can a user reach another user's resource by changing an ID?
- **SEC-06 Sensitive data** — passwords/PII not sent to client or logged?
- **SEC-07 Dependency risk** — known-vulnerable libraries imported? (verify with audit tool in Phase 2)
- **SEC-08 Rate limiting** — abusable endpoints rate-limited?
- **SEC-09 CORS/CSRF** — overly permissive CORS or missing CSRF protection?
- **SEC-10 Token handling** — tokens stored safely with correct expiry?

## 3. Domain 3 — Performance (PERF)

- **PERF-01 N+1 query** — DB query inside a loop?
- **PERF-02 Unnecessary re-render** — components rendering more than needed? (frontend)
- **PERF-03 Missing memoization** — heavy computation that should be memoized/cached?
- **PERF-04 Blocking ops** — heavy sync work that should be async?
- **PERF-05 Memory leak** — listeners/timers/subscriptions not cleaned up?
- **PERF-06 Bundle size** — large imports replaceable with specific ones? (where applicable)
- **PERF-07 DB index** — queries hitting indexed fields? (where queries are visible)
- **PERF-08 Payload size** — API returning more than the client needs?
- **PERF-09 Lazy loading** — large components/resources that could be lazy-loaded?
- **PERF-10 Async efficiency** — independent promises run sequentially instead of `Promise.all`?

## 4. Domain 4 — Blueprint / Consistency (BC)

**If a blueprint/spec is provided:**
- **BC-01 Feature completeness** — all blueprint features implemented?
- **BC-02 Business logic** — code logic matches the spec?
- **BC-03 Edge-case coverage** — blueprint edge cases handled in code?
- **BC-04 Naming consistency** — names match blueprint terminology?
- **BC-05 Data structure** — types/fields match the blueprint?
- **BC-06 Error-handling spec** — error handling matches the spec?
- **BC-07 Missing implementation** — any blueprint part not implemented at all?
- **BC-08 Over-implementation** — features beyond blueprint scope (scope creep)?

**If no blueprint:**
- **BC-01 Architectural consistency** — architectural patterns consistent throughout?
- **BC-02 Convention consistency** — naming conventions and folder structure consistent?
- **BC-03 Pattern consistency** — same pattern used for similar cases?

---

## 5. Severity system & scoring

Classify every finding on five levels:

- 🔴 **CRITICAL (10/10)** — crash, data leak, or direct security breach. Must fix;
  code must not ship until resolved.
- 🟠 **HIGH (7–9/10)** — serious issue affecting core function, significant
  security, or severe production performance. Fix before merge/deploy.
- 🟡 **MEDIUM (4–6/10)** — maintainability/efficiency/best-practice issue. Fix next iteration.
- 🔵 **LOW (1–3/10)** — minor issue, style, small tech debt. Backlog.
- ⚪ **INFO (0/10)** — non-urgent suggestion or observation.

**Overall score:** failed CRITICAL ×3 weight, failed HIGH ×2, failed MEDIUM ×1;
LOW/INFO don't affect score.
`Overall Score = (passed points / max points) × 10`.

Each finding also carries a **confidence label** from Phase 2
(`[High] / [Medium] / [Low — needs verification]`). Severity ≠ confidence: a HIGH
finding can be Low-confidence if it depends on runtime data you couldn't observe.

---

## 6. Report format (Phase 5)

Report in **English**. Present once, after Phases 0–4 are complete.

```
┌──────────────────────────────────────────────┐
│ 🔍 CODE REVIEW REPORT                          │
│ File(s)     : [name / module]                  │
│ Scope       : [whole file | git diff vs base]  │
│ Tech Stack  : [stack]                          │
│ Blueprint   : [Provided | Not provided]        │
│ Verified by : [tools run, or "reasoning only"] │
│ Date        : [date]                           │
│ Overall     : [X/10] — [CRITICAL/HIGH/…/CLEAN] │
├──────────────────────────────────────────────┤
│ SUMMARY                                        │
│ 🔴 Critical : [N]   🟠 High : [N]              │
│ 🟡 Medium   : [N]   🔵 Low  : [N]              │
│ ⚪ Info     : [N]   ✅ Passed: [N]             │
└──────────────────────────────────────────────┘
```

Per finding:

```
[DOMAIN-ID-###] 🔴/🟠/🟡/🔵/⚪ SEVERITY · [High/Medium/Low confidence] — Short title
Location : [file + line/function/variable]
Issue    : [specific problem]
Evidence : [the line/tool output that proves it]
Impact   : [what happens if left]
Fix      : [summary of the fix]
```

Then a **Top-5 Priority Fixes** block (id — title — why it's prioritized), a
**Reflection Notes** block (what was dropped/downgraded/merged in Phase 3 and why),
and a one-line **Critique-Correction outcome** (converged in N rounds; key
disputes, if any).

---

## 7. Adaptive fix: Full Rewrite vs Patch

After the report, proceed to fixes (don't wait for confirmation to *produce*
them; do confirm before *writing to files* in a repo).

**Mode A — Full Rewrite (file < 100 lines).** State: "This file is N lines —
Full Rewrite Mode." Output the entire corrected file, English names/comments,
header:
```
// FILE: [path]
// REVIEWED: [date]  MODE: Full Rewrite
// CHANGES: [N] fixed ([c] critical, [h] high, [m] medium, [l] low)
```
Rules: rewrite the whole file with all fixes applied; keep correct logic intact;
add no unrequested features; don't change behavior — only fix implementation;
no placeholders (must run as-is).

**Mode B — Patch (file ≥ 100 lines).** State: "This file is N lines — Patch Mode."
Per finding that needs a fix, show BEFORE/AFTER with a few lines of surrounding
context and a `// WHY` comment:
```
// FILE: [path]   PATCH: [ID] — [title]
// ─── BEFORE ───
[original problematic code with context]
// ─── AFTER ───
[fixed code]
// ─── WHY ───
[short reason]
```
Rules: rewrite only the affected span; include context for locating it; don't
refactor outside the finding; don't add features; every patch has a WHY.

Order fixes by severity (Critical → High → Medium). Low/Info optional.

## 8. Changelog + clean-code standard + anti-hallucination guard

**Changelog** (reasons in **Bahasa Indonesia**):
```
📝 CHANGELOG — [file] ([Full Rewrite | Patch])
[ID] Yang berubah : [perubahan spesifik]
     Baris        : [lama → baru, jika relevan]
     Alasan       : [mengapa diperbaiki]
```

**Clean-code standard** (enforce in both modes): single responsibility · no
`x/temp/data/res/val` names · no hardcoded values (constants/env) · error
handling on every async op · loading/empty/error states handled · comments only
for genuinely complex logic · no dead code/unused imports · complete, accurate
types where applicable.

**Anti-hallucination guard:** don't change already-correct business logic; if
unsure of the right implementation, write `// TODO: Verify this implementation —
[reason]`; for specific library/API calls, `// Note: Verify latest API syntax
before deployment`. Prefer verifying in Phase 2 over guessing.

## 9. Saved `.md` report structure

Write to the repo (e.g. `./<file-or-module>-code-review.md`) and tell the user
the path. Structure (English):

```
# Code Review Report — [File/Module]
**Date** · **Tech Stack** · **Scope** · **Blueprint** · **Verified by**
**Overall Score**: [X/10] — [category]
**Reviewer**: Edho Ferdian Code Review Mode (Skill Edition)

## Summary            (severity table)
## Findings           (full findings with confidence labels + evidence)
## Top 5 Priority Fixes
## Reflection Notes    (Phase 3 — dropped/downgraded/merged + why)
## Critique-Correction Outcome  (Phase 4 — rounds, disputes, resolution)
## Revised Code        (full rewrite) OR ## Patches (patch mode)
## Changelog           (reasons in Bahasa Indonesia)
## Re-review Checklist (re-run when: business logic changed · new dependency ·
                        DB/API structure changed · before merge to main/prod)
```

## 10. Special scenarios

- **Module mode (multi-file):** review each file (each picks its own fix mode by
  length), then add a consolidated report with cross-file findings
  (inconsistencies between files) and one combined `.md`. In an agentic context
  you don't need to pause for confirmation between files — process them, then
  summarize; pause only before writing fixes to disk.
- **Very long file (500+ lines):** review by section (Imports & Constants · Types
  & Interfaces · Main Logic · Helpers · Exports), then present the full result.
- **Incomplete/unparseable code:** if the code is truncated, say exactly what's
  missing and that partial review yields unreliable findings; ask for the
  complete file rather than guessing.
