# Implementation Batch Guidance (Spec Kit)

**Purpose**: Rules for `/speckit-tasks` and `/speckit-implement` so each batch produces a
small, logic-wise reviewable diff suitable for one commit.

**Authority**: Expands constitution **Development Workflow** (review-sized batches).
Constitution states the MUST; this document states the HOW for task generation.

---

## Batch kinds

| Kind | ID pattern | Produces code diff? | Use |
|------|------------|---------------------|-----|
| **Preflight** | P1, P2, … | No | Baseline, verify green — run once before IB01 |
| **Implementation batch** | IB01, IB02, … | **Always yes** | One `/speckit-implement` request per IB |
| **Postflight** | PF1, PF2, … | No | Full validation — run once after last IB |

Do **not** label preflight/postflight as IB. Do **not** create IBs that only run greps or
record metrics unless paired with code in the same feature (put validation in postflight).

---

## Split principles (smallest logical review unit)

### 1. One review theme per IB

Each IB answers one reviewer question:

- “Is shared setup correct?” (foundation IB)
- “Same behavior, cleaner structure?” (**refactor IB**)
- “New behavior covered?” (**coverage IB**)

### 2. Separate refactor from coverage when both touch the same file

| IB type | User-story mapping | Allowed changes |
|---------|-------------------|-----------------|
| **Refactor IB** | US1 + US2 (structure, dedup, naming) | Behavior-preserving only |
| **Coverage IB** | US3 (new tests, expanded tables) | Add assertions/scenarios only |
| **Single IB** | US1 + US2 (+ tiny US3 doc comment) | When no new tests/subtests |

**Rule**: For the same file, generate **refactor IB before coverage IB** (e.g. IB06 then
IB07). Never combine both in one `/speckit-implement` prompt.

### 3. Prefer one file per IB

- Default: one primary file per IB (~30–150 lines changed).
- Exception: **atomic cross-file changes** (e.g. move shared builders from file A to
  fixtures + update callers B and C) = one IB, 2–3 files, one logical move.

### 4. Do not split too fine

Avoid separate IBs for:

- Rename vs subtest vocabulary vs NotFound style in the same file (one refactor IB)
- Client wrapper dedup vs finalizer dedup in the same file (one refactor IB)
- Each helper function in fixtures (one foundation IB)

### 5. Foundation before vertical slices

Typical order:

1. IB01 — shared helpers / infrastructure (single support file)
2. IB02 — atomic fixture or module relocation (if needed)
3. IB03+ — per-unit refactor and coverage IBs

---

## tasks.md structure (required sections)

Generated `tasks.md` MUST include:

1. **How to use** — table of Preflight / IB / Postflight
2. **Implementation Batches table** — IB | Type | File(s) | Review theme
3. **Per-IB task blocks** — checklist items with exact paths; state “Do **not** …” for
   out-of-scope work in that IB
4. **Refactor checklist** and **Coverage checklist** (when feature has both)
5. **Dependencies** — strict order; refactor→coverage pairs
6. **`/speckit-implement` prompts** — one IB per request; examples

Task IDs (T001, T002, …) map to IBs. Multiple tasks per IB are allowed when they are
one atomic change (e.g. move + update call sites).

---

## Mapping user stories to IB types

| Spec priority | Typical IB content |
|---------------|-------------------|
| P1 — layout, naming, conventions | Refactor IBs |
| P2 — shared fixtures, dedup | Foundation IBs + refactor IBs |
| P3 — coverage gaps, new scenarios | Coverage IBs (after refactor IB for same file) |

User story phases in tasks.md are orthogonal to IBs: IBs are the **commit/review** axis;
story labels ([US1], [US2], [US3]) stay on individual tasks inside an IB.

---

## `/speckit-tasks` generation checklist

When generating tasks.md:

- [ ] Every IB produces a reviewable code diff (except preflight/postflight)
- [ ] Refactor and coverage split for any file that gains new tests or subtests
- [ ] Foundation IBs come before per-file IBs
- [ ] IB table lists review theme in plain language
- [ ] MVP path documented (e.g. defer coverage IBs)
- [ ] No multi-file IBs except atomic relocation/setup
- [ ] Explicit “implement IBxx” examples; warn against combining refactor+coverage IBs

---

## `/speckit-implement` usage

User prompt pattern:

```text
implement IB03
```

Agent MUST:

1. Complete only that IB’s tasks
2. Run affected tests
3. Stop for user review/commit before next IB

---

## Example (test refactor feature)

| IB | Type | Files | Theme |
|----|------|-------|-------|
| IB01 | foundation | `fixtures_test.go` | Add shared helpers |
| IB02 | foundation | fixtures + 2 callers | Relocate shared builders |
| IB03 | single | `foo_test.go` | Refactor only |
| IB04 | refactor | `bar_test.go` | Rename, dedup |
| IB05 | coverage | `bar_test.go` | New subtests only |

---

## Sync into projects

After editing this file in spec-kit-preset:

1. Copy to project: `.specify/memory/implementation-batches.md` (or `.specify/templates/`)
2. Re-copy `constitution.md` if workflow bullets changed
3. Optionally add to `speckit-tasks` skill: “Read implementation-batches.md when generating tasks”
4. Optionally embed summary in `.specify/templates/tasks-template.md` under Organization

**Version**: 1.0.0 | **Companion to constitution**: 1.3.0+
