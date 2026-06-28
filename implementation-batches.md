# Implementation Batch Guidance (Spec Kit)

**Purpose**: Rules for `/speckit-tasks` and `/speckit-implement` so each batch produces a
small, logic-wise reviewable diff suitable for one commit.

**Authority**: Expands constitution **Development Workflow** (review-sized batches,
func-def + call-site rule). Constitution states the MUST; this document states the HOW for
task generation.

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

## Func-def + call-site rule (mandatory)

If an IB **adds**, **moves**, **renames**, or **changes the signature** of a function,
that IB **MUST also update every call site** in the same diff.

| Change | Same IB must include |
|--------|----------------------|
| New helper in shared file | First consumer(s) — adopt the helper in the same batch |
| Move builder/constant to fixtures | All importers updated; definitions removed from source file |
| Rename function | Every reference updated |
| Change signature | Every caller updated |
| Rewrite wrapper to delegate | Wrapper body changed; callers may stay on wrapper name |
| Rename test function (`Test*`) | No in-package call sites (discovered by test runner) |

**Do not**:

- Commit helper or builder **definitions** with zero call-site updates in the same IB
- Split “add `newFoo` in fixtures” and “use `newFoo` in file A” across separate IBs
  unless file A is included in the same IB as the definition

**Progress tracking**: When work is reverted, mark the IB task(s) unchecked and set
Progress status to `pending` in `tasks.md`.

---

## Split principles (smallest logical review unit)

### 1. One review theme per IB

Each IB answers one reviewer question:

- “Is shared setup correct **and wired in**?” (foundation IB)
- “Same behavior, cleaner structure?” (**refactor IB**)
- “New behavior covered?” (**coverage IB**)

### 2. Separate refactor from coverage when both touch the same file

| IB type | User-story mapping | Allowed changes |
|---------|-------------------|-----------------|
| **Refactor IB** | US1 + US2 (structure, dedup, naming) | Behavior-preserving only |
| **Coverage IB** | US3 (new tests, expanded tables) | Add assertions/scenarios only |
| **Single IB** | US1 + US2 (+ tiny US3 doc comment) | When no new tests/subtests |

**Rule**: For the same file, generate **refactor IB before coverage IB** (e.g. IB05 then
IB06). Never combine both in one `/speckit-implement` prompt.

### 3. Prefer one file per IB

- Default: one primary file per IB (~30–150 lines changed).
- Exception: **atomic cross-file changes** (e.g. move shared builders from file A to
  fixtures + update callers B and C) = one IB, 2–3 files, one logical move.
- Exception: **foundation IB01** = shared support file **plus** first adopter file so
  new helpers ship with call sites in one reviewable diff.

### 4. Do not split too fine

Avoid separate IBs for:

- Rename vs subtest vocabulary vs NotFound style in the same file (one refactor IB)
- Client wrapper dedup vs finalizer dedup in the same file (one refactor IB)
- Each helper function in fixtures (one foundation IB — **with** first adopter)
- Helper definition vs first usage (must be same IB per func-def rule)

### 5. Foundation before vertical slices

Typical order:

1. **IB01** — shared helpers in support file **+ first adopter** (defs and call sites
   together; e.g. `fixtures_test.go` + reference unit test file)
2. **IB02** — atomic fixture or module relocation (move defs + update **all** importers)
3. **IB03+** — per-unit refactor and coverage IBs

---

## Per-IB task wording

When an IB introduces or changes functions, task bullets MUST separate **Define** and
**Call sites** (or use explicit “replace every `X` call site” language):

```markdown
- [ ] T001 Add helpers to path/fixtures_test.go and adopt in path/foo_test.go in the **same IB**:
  - **Define** in `fixtures_test.go`: `newFakeClient`, `requireNotFound`, …
  - **Call sites** in `foo_test.go`: rewrite `newFooClient` to delegate; replace inline builders
```

For refactor IBs without new defs, require updating **every** call site of the pattern
being replaced (not only a named wrapper):

- “Replace **every** `fake.NewClientBuilder` call site with `newFakeClient(…)`”
- “Rewrite `newBarClient` to delegate to `newFakeClient`; replace any inline builders”

---

## tasks.md structure (required sections)

Generated `tasks.md` MUST include:

1. **How to use** — table of Preflight / IB / Postflight; **func-def + call-site rule**
2. **Implementation Batches table** — IB | Type | File(s) | Review theme
3. **Progress** (optional but recommended) — IB status; `Next: IBxx`
4. **Per-IB task blocks** — checklist items with exact paths; **Define** / **Call sites**
   where applicable; state “Do **not** …” for out-of-scope work in that IB
5. **Refactor checklist** and **Coverage checklist** (when feature has both)
6. **Dependencies** — strict order; refactor→coverage pairs
7. **`/speckit-implement` prompts** — one IB per request; examples

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
- [ ] **Func-def rule**: no IB with new/moved/renamed/changed functions without call-site updates in the same IB
- [ ] **IB01** pairs shared helpers with first adopter (not helpers-only)
- [ ] Relocation IBs list moved symbols and **all** importer files
- [ ] Refactor and coverage split for any file that gains new tests or subtests
- [ ] Foundation IBs come before per-file IBs
- [ ] IB table lists review theme in plain language
- [ ] MVP path documented (e.g. defer coverage IBs)
- [ ] No multi-file IBs except atomic relocation/setup or foundation IB01
- [ ] Refactor tasks name every call-site pattern to replace (wrappers **and** inline usage)
- [ ] Explicit “implement IBxx” examples; warn against combining refactor+coverage IBs
- [ ] Notes: never commit defs without call sites in the same IB

---

## `/speckit-implement` usage

User prompt pattern:

```text
implement IB01
```

Agent MUST:

1. Complete only that IB’s tasks (including all call-site updates for any defs changed)
2. Run affected tests
3. Stop for user review/commit before next IB
4. Mark IB tasks complete only after user confirms; on revert, uncheck tasks and update Progress

---

## Example (test refactor feature)

| IB | Type | Files | Theme |
|----|------|-------|-------|
| IB01 | foundation | `fixtures_test.go`, `dnsrecord_test.go` | Shared helpers **+ first adopter** |
| IB02 | foundation | fixtures + 2 callers | Relocate shared builders + update all call sites |
| IB03 | single | `certificate_test.go` | Refactor + doc comment |
| IB04 | single | `domainmapping_test.go` | Refactor only |
| IB05 | refactor | `bar_test.go` | Rename, dedup, replace all client construction call sites |
| IB06 | coverage | `bar_test.go` | New subtests only |

**Wrong** (violates func-def rule):

| IB | Files | Problem |
|----|-------|---------|
| IB01 | `fixtures_test.go` only | Helpers defined with no consumers until IB03 |
| IB02 | move builders only | Moved defs without updating importers |

---

## Sync into projects

After editing this file in spec-kit-preset:

1. Copy to project: `.specify/memory/implementation-batches.md` (or `.specify/templates/`)
2. Re-copy `constitution.md` if workflow bullets changed
3. Optionally add to `speckit-tasks` skill: “Read implementation-batches.md when generating tasks”
4. Optionally embed summary in `.specify/templates/tasks-template.md` under Organization

**Version**: 1.1.0 | **Companion to constitution**: 1.4.0+
