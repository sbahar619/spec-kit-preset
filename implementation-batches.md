# Implementation Batch Guidance (Spec Kit)

**Purpose**: Rules for `/speckit-tasks` and `/speckit-implement` so each batch produces a
small, logic-wise reviewable diff suitable for one commit.

**Authority**: Expands constitution **Development Workflow** (review-sized batches,
func-def + call-site rule, separation of concerns). Constitution states the MUST; this
document states the HOW for task generation.

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
| Rename subtest string (`t.Run("…")`) | No in-package call sites |

**Do not**:

- Commit helper or builder **definitions** with zero call-site updates in the same IB
- Split “add `newFoo` in fixtures” and “use `newFoo` in file A” across separate IBs
  unless file A is included in the same IB as the definition

**Progress tracking**: When work is reverted, mark the IB task(s) unchecked and set
Progress status to `pending` in `tasks.md`.

---

## Separation of concerns (split principles)

Each IB MUST target **one concern**. Reviewers should answer one question per diff.

### Concern types

| Concern | What changes | What must NOT change |
|---------|--------------|----------------------|
| **Foundation** | Shared fixtures/helpers in support file **when ≥2 consumers**; first adopter **wiring** in one reference file | Test/subtest names; assertion style beyond what wiring requires; helpers with only one consumer in support file |
| **Naming** | Outer `Test*` renames; recurring subtest string normalization (project vocabulary) | Fake-client construction, helpers, assertion idioms, test logic |
| **Wiring** | Shared helper adoption, client-builder dedup, assertion standardization, setup dedup | Outer/subtest names (done in Naming IB for that file) |
| **Coverage** | New subtests, expanded tables, new scenarios (US3) | Renames or unrelated refactors |

### Order per file

When a file needs more than one concern, generate IBs in this order:

```text
Foundation (once, package-wide) → Naming → Wiring → Coverage
```

- **Naming before Wiring** for the same file — never combine in one IB.
- **Wiring before Coverage** for the same file — never combine in one IB.
- **Foundation IB01** = support file + first adopter **wiring only** (no renames in IB01).
- **Foundation IB02** (typical) = atomic fixture relocation (move defs + update **all** importers).

### One review theme per IB

Each IB answers one reviewer question:

- “Is shared setup correct **and wired in**?” (**Foundation** / wiring portion)
- “Are test labels consistent?” (**Naming**)
- “Same behavior, cleaner structure?” (**Wiring**)
- “New behavior covered?” (**Coverage**)

### Refactor vs coverage (still applies)

| IB type | User-story mapping | Allowed changes |
|---------|-------------------|-----------------|
| **Naming IB** | US1 | Rename-only |
| **Wiring IB** | US2 | Behavior-preserving structure/dedup only |
| **Coverage IB** | US3 | Add assertions/scenarios only |
| **Single IB** | US1 + US2 (+ tiny US3 doc comment) | When file has no split-worthy naming or coverage work |

**Rule**: For the same file, generate **Naming → Wiring → Coverage** when multiple apply.
Never combine Naming + Wiring or Wiring + Coverage in one `/speckit-implement` prompt.

### Prefer one file per IB

- Default: one primary file per IB (~30–150 lines changed).
- Exception: **atomic cross-file changes** (e.g. move shared builders from file A to
  fixtures + update callers B and C) = one IB, 2–3 files, one logical move.
- Exception: **IB01** = first adopter **wiring** in one file (local helpers until a second file needs them; **no renames** in IB01).
- **Locality**: define helpers in the consumer file until ≥2 files use them; promote to support file in the IB that adds the second consumer (move + update all call sites in same diff).

### Do not split too fine (within a concern)

Avoid separate IBs for:

- Client wrapper dedup vs finalizer dedup in the same file (one **Wiring** IB)
- Each helper function in fixtures (one **Foundation** IB — **with** first adopter wiring)
- Helper definition vs first usage (must be same IB per func-def rule)

**Do** split across IBs:

- Outer/subtest renames (**Naming**) vs helper adoption and dedup (**Wiring**)
- Refactor (**Naming** + **Wiring**) vs new subtests (**Coverage**)

### Foundation before vertical slices

Typical order:

1. **IB01** — first adopter **wiring** in one file (local helpers; e.g. reference unit test file; **no renames**)
2. **IB02** — **Naming** for first adopter (if applicable) OR atomic fixture relocation
3. **IB03** — atomic fixture or module relocation (move defs + update **all** importers)
4. **IB04+** — per-unit **Naming → Wiring → Coverage** IBs; promote local helpers to support file when second consumer wires in

Adjust numbering to fit the feature; keep concern order strict.

---

## Per-IB task wording

When an IB introduces or changes functions, task bullets MUST separate **Define** and
**Call sites** (or use explicit “replace every `X` call site” language):

```markdown
## IB01 — foo wiring

- [ ] T001 Adopt wiring in path/foo_test.go:
  - **Define** in `foo_test.go` (local until reused by ≥2 files): `newFakeClient`, …
  - **Wiring**: rewrite `newFooClient` to delegate; replace inline builders
  - **Out of scope**: renames (IB02); moving helpers to support file (IB that adds second consumer)

## IB05 — bar wiring (promotes shared helper)

- [ ] T006 Move `newFakeClient` from `foo_test.go` to `fixtures_test.go`; update foo + bar call sites in the **same IB**

## IB02 — foo naming

- [ ] T002 [US1] Rename tests in `foo_test.go` only:
  - Outer tests per naming convention
  - Recurring subtest strings per vocabulary
  - **Out of scope**: helpers, fake-client wiring, assertions, test logic
```

For **Wiring** IBs without new defs, require updating **every** call site of the pattern
being replaced (not only a named wrapper):

- “Replace **every** `fake.NewClientBuilder` call site with `newFakeClient(…)`”
- “Rewrite `newBarClient` to delegate to `newFakeClient`; replace any inline builders”

For **Naming** IBs, require explicit **Out of scope** lines forbidding wiring changes.

---

## tasks.md structure (required sections)

Generated `tasks.md` MUST include:

1. **How to use** — table of Preflight / IB / Postflight; **func-def + call-site rule**;
   **Split principle** table (Foundation / Naming / Wiring / Coverage)
2. **Implementation Batches table** — IB | Concern | File(s) | Review theme
3. **Progress** (optional but recommended) — IB status; `Next: IBxx`
4. **Per-IB task blocks** — checklist items with exact paths; **Define** / **Wiring** /
   **Out of scope** where applicable; state “Do **not** …” for out-of-scope work
5. **Naming checklist**, **Wiring checklist**, and **Coverage checklist** (when feature has them)
6. **Dependencies** — strict order; Naming→Wiring→Coverage pairs per file
7. **`/speckit-implement` prompts** — one IB per request; examples; warn against combining concerns

Task IDs (T001, T002, …) map to IBs. Multiple tasks per IB are allowed when they are
one atomic change (e.g. move + update call sites).

---

## Mapping user stories to IB types

| Spec priority | Typical IB content |
|---------------|-------------------|
| P1 — layout, naming, conventions | **Naming** IBs |
| P2 — shared fixtures, dedup, assertion style | **Foundation** + **Wiring** IBs |
| P3 — coverage gaps, new scenarios | **Coverage** IBs (after Naming + Wiring for same file) |

User story phases in tasks.md are orthogonal to IBs: IBs are the **commit/review** axis;
story labels ([US1], [US2], [US3]) stay on individual tasks inside an IB.

---

## `/speckit-tasks` generation checklist

When generating tasks.md:

- [ ] Every IB produces a reviewable code diff (except preflight/postflight)
- [ ] **Func-def rule**: no IB with new/moved/renamed/changed functions without call-site updates in the same IB
- [ ] **IB01** is first adopter **wiring** only (local helpers; not support-file helpers with one consumer)
- [ ] **Naming** and **Wiring** are separate IBs when both apply to the same file
- [ ] Relocation IBs list moved symbols and **all** importer files
- [ ] **Coverage** split from **Naming** and **Wiring** for any file that gains new tests or subtests
- [ ] Foundation IBs come before per-file Naming/Wiring IBs
- [ ] IB table lists **Concern** column and review theme in plain language
- [ ] MVP path documented (e.g. defer coverage IBs)
- [ ] No multi-file IBs except atomic relocation/setup or foundation IB01
- [ ] Wiring tasks name every call-site pattern to replace (wrappers **and** inline usage)
- [ ] Naming tasks include **Out of scope** forbidding wiring changes
- [ ] Explicit “implement IBxx” examples; warn against combining Naming+Wiring or Wiring+Coverage
- [ ] Notes: never commit defs without call sites in the same IB

---

## `/speckit-implement` usage

User prompt pattern:

```text
implement IB01
```

Agent MUST:

1. Complete only that IB’s tasks (respect **Out of scope** — do not apply later concerns)
2. Run affected tests
3. Stop for user review/commit before next IB
4. Mark IB tasks complete only after user confirms; on revert, uncheck tasks and update Progress

---

## Example (test refactor feature)

| IB | Concern | Files | Theme |
|----|---------|-------|-------|
| IB01 | wiring | `dnsrecord_test.go` | Local `newFakeClient` + dnsrecord wiring (**no renames**) |
| IB02 | naming | `dnsrecord_test.go` | dnsrecord outer + subtest vocabulary only |
| IB03 | foundation | fixtures + 2 callers | Relocate shared builders + update all call sites |
| IB04 | naming | `certificate_test.go` | Subtest vocabulary only |
| IB05 | wiring | `fixtures_test.go`, `certificate_test.go`, `dnsrecord_test.go` | Promote `newFakeClient` to fixtures + certificate wiring |
| IB06 | naming | `bar_test.go` | Outer + subtest renames |
| IB07 | wiring | `bar_test.go` | Client construction, dedup |
| IB08 | coverage | `bar_test.go` | New subtests only |

**Wrong** (violates func-def rule):

| IB | Files | Problem |
|----|-------|---------|
| IB01 | `fixtures_test.go` only | Helpers defined with no consumers until IB03 |

**Wrong** (violates separation of concerns):

| IB | Files | Problem |
|----|-------|---------|
| IB01 | fixtures + dnsrecord | Wiring **and** test/subtest renames in same IB |
| IB05 | `bar_test.go` | Naming + wiring + new subtests in one IB |

**Right** (retroactive split after combined IB01):

If wiring and naming landed together, revert naming to match IB01 scope, mark Naming IB
pending, and apply renames in the dedicated Naming IB.

---

## Sync into projects

After editing this file in spec-kit-preset:

1. Copy to project: `.specify/memory/implementation-batches.md` (or `.specify/templates/`)
2. Re-copy `constitution.md` if workflow bullets changed
3. Optionally add to `speckit-tasks` skill: “Read implementation-batches.md when generating tasks”
4. Optionally embed summary in `.specify/templates/tasks-template.md` under Organization

**Version**: 1.2.0 | **Companion to constitution**: 1.5.0+
