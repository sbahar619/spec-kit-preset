# Implementation Batch Guidance (Spec Kit)

**Purpose**: Rules for `/speckit-tasks` and `/speckit-implement` — one concern per batch, complete wiring, reviewable diffs.

**Authority**: Expands constitution **Development Workflow**. Constitution = MUST; this doc = HOW.

---

## IB sequence

An **IB sequence** is the full unit of delivery: Preflight → IB01…IBn → Postflight.
Preflight and Postflight are mandatory gates — not optional follow-on work.

```
Preflight  →  IB01 → IB02 → … → IBn  →  Postflight
  (open)           delivery batches          (close)
```

**Preflight is the mandatory open** — do not start IB01 until all Preflight gates pass.
**Postflight is the mandatory close** — the sequence is not complete until all Postflight gates pass.

### Phase reference

| Phase | ID pattern | Produces code diff? | Role |
|-------|------------|---------------------|------|
| **Preflight** | P1, P2, … | No | Baseline, verify green — once, before IB01 |
| **Implementation batch** | IB01, IB02, … | **Always yes** | One `/speckit-implement` request per IB |
| **Postflight** | PF1, PF2, … | No | Full validation — once, after last IB |

Do **not** label Preflight/Postflight as IB. Validation-only work belongs in Postflight unless paired with code in the same feature.

---

## Complete wiring (func-def + call-site)

If an IB **adds**, **moves**, **renames**, or **changes the signature** of a function, that IB **MUST** update **every in-scope call site** in the same diff — not the smallest diff that compiles.

| Requirement | Rule |
|-------------|------|
| **Consumer inventory** | List every file or pattern in scope (e.g. all `*_test.go` in package X; all importers of `newFoo`). |
| **Same diff** | Define/move symbol **and** wire every inventory consumer; remove old defs on relocation. |
| **No dead code** | No new/moved symbol without callers in the diff; zero in-scope usages of the replaced pattern remain. |
| **Done when** | Explicit: no orphan defs; no unwired in-scope sites; tests green. |

| Change | Same IB must include |
|--------|----------------------|
| New/moved helper | All consumers in inventory |
| Rename / signature change | Every reference updated |
| `Test*` / subtest rename | No in-package call sites (test runner) |

**Do not**: defs-only; wire one file while others stay on the old pattern; split define vs use across IBs unless all consumers are in the define IB.

**Horizontal IB**: When a concern is uniform package-wide (shared client, naming vocabulary), one IB MAY touch many files if concern is single, inventory lists every file, and Done when requires all in diff.

**Progress**: On revert, uncheck IB tasks and set status `pending` in `tasks.md`.

---

## Separation of concerns

Each IB = **one concern**, one review question, one commit. User story labels ([US1]–[US3]) tag tasks inside IBs; IBs are the review axis.

When multiple concerns apply to the same files, behavior-preserving work **MUST** split into
separate IBs — one concern per batch. Behavior-adding work (new tests, new features) **MUST**
stay out of wiring IBs when both touch those files.

| Concern | Changes | Must NOT change |
|---------|---------|-----------------|
| **Foundation** | Shared fixtures/helpers (≥2 consumers); wiring in inventory | Names, assertion style, single-consumer helpers in support file |
| **Naming** | Outer `Test*` + subtest vocabulary | Helpers, wiring, assertions, logic |
| **Wiring** | Helper adoption, dedup, assertion/setup standardization | Outer/subtest names |
| **Coverage** | New scenarios/subtests (US3) | Renames, wiring, unrelated refactors |

**Concern order** (canonical — package and per-file):

```text
Foundation → Wiring → Naming → Coverage
```

Wiring before Naming: package-wide helper/symbol adoption (horizontal IB) lands before rename passes so consumers are on the new pattern first. Per-file Wiring IBs follow the same order when a file still needs both.

Never combine **Naming + Wiring**, **Wiring + Coverage**, or **removals + adds** in one IB. Run removal/anti-redundancy IBs before coverage adds when both apply.

**File scope**: Default one primary file (~30–150 lines). Multi-file only for atomic relocation or horizontal IB (inventory required). Within one concern, do not split wrapper dedup vs finalizer dedup; do split naming vs wiring vs coverage.

**Typical sequence**: IB01 Foundation or package-wide Wiring → IB02 Naming → IB03 relocation (if not in IB01) → IB04+ per-file Wiring/Coverage gaps. Adjust numbering; keep concern order above.

| Spec priority | Typical IBs |
|---------------|-------------|
| P1 — layout, naming | Naming |
| P2 — fixtures, dedup | Foundation + Wiring |
| P3 — coverage gaps | Coverage (after Wiring + Naming for same file) |

---

## Coverage IBs (scenario-contract)

Add tests only for a **distinct scenario contract** (preconditions + outcome) — not matrix symmetry or coverage % alone.

1. **Audit** same file before adding a matrix row subtest.
2. Satisfy row with subtest **or** documented exception — not both for the same contract.
3. Each new subtest must assert a contract not already in the same file.

---

## tasks.md requirements

Generated `tasks.md` MUST include:

1. **How to use** — Preflight / IB / Postflight; wiring + separation rules (brief)
2. **Implementation Batches table** — IB | Concern | Review theme | Consumer inventory (summary)
3. **Progress** — status; `Next: IBxx` (recommended)
4. **Per-IB blocks** — Define, Consumer inventory, Wiring, Out of scope, Done when
5. **Dependencies** — concern order; removal before coverage when applicable
6. **`/speckit-implement` prompts** — one IB per request
7. **Gates** in quickstart.md when applicable
8. **IB Traceability table** — one row per IB with markdown links to `spec.md` anchors:

   | IB | User Stories | Functional Requirements | Success Criteria |
   |----|-------------|------------------------|-----------------|
   | **IBxx** | [USn — Title](spec.md#anchor) | [FR-nnn](spec.md#functional-requirements) · … | [SC-nnn](spec.md#measurable-outcomes) … |

   - Link targets: `spec.md#user-story-N-...`, `spec.md#functional-requirements`, `spec.md#measurable-outcomes`
   - Each cell lists every US / FR / SC the IB directly advances, separated by ` · `
   - Place this table after the existing User Story Coverage table

**Per-IB example**:

```markdown
## IB01 — package-wide foo wiring

**Consumer inventory**: fixtures + every `*_test.go` in package (all `OldBuilder` call sites)

- [ ] T001 [US2] Wire fake client:
  - **Define**: `newFakeClient` in fixtures
  - **Wiring**: replace every `OldBuilder` call site
  - **Out of scope**: renames (IB02)
  - **Done when**: no `OldBuilder` in package; all inventory files in diff; tests green
```

### Generation checklist (`/speckit-tasks`)

- [ ] Every IB produces a code diff (except preflight/postflight)
- [ ] Func-def + complete consumer inventory on wiring/foundation IBs
- [ ] One concern per IB; separation order respected
- [ ] Relocation = move + all importers + remove old defs
- [ ] Coverage: matrix-audit; no blind adds; no mix with naming/wiring/removal
- [ ] Recommend `/speckit-analyze` after tasks, before first implement
- [ ] IB Traceability table present: one row per IB; every US/FR/SC linked to `spec.md` anchors

---

## Workflow

**`/speckit-implement IBxx`**: Complete only that IB; verify inventory + separation; run tests/gates; stop for review; mark `[X]` only after gates pass.

**`/speckit-analyze`** (before first implement): Flag **CRITICAL** for defs without in-scope call sites, missing Consumer inventory/Done when, concern mixing, or coverage adds without audit.

---

## Anti-patterns

| IB | Problem |
|----|---------|
| Support file defs only | Dead code |
| Def + one file wired | Partial wiring |
| Wiring + renames | Concern mix |
| Wiring + new subtests | Concern mix |
| Removals + adds | Concern mix |
| Matrix add without audit | Redundant subtests |

If a combined IB landed, revert out-of-scope portions, mark the dedicated IB pending, re-apply in the correct IB.
