# [PROJECT_NAME] Constitution

## Core Principles

### I. Minimal Change

Every change MUST be the smallest correct diff that solves the stated problem.
Drive-by refactors, unrelated formatting, and scope creep are forbidden unless
explicitly requested. Match existing package patterns, naming, imports, and
documentation level in the files you touch.

### II. Simplicity Over Abstraction

Do not over-engineer. Avoid new helpers, layers, or abstractions for one-off or
single-use cases. Prefer inline logic when extraction adds no reuse or clarity.
Use existing repo utilities, stdlib, and already-imported dependencies before
adding new ones.

### III. Single Responsibility & Locality

Functions, types, and modules MUST have one clear purpose. Define helpers as
close as possible to their use; widen scope only when reused or required by the
API. Public surfaces stay minimal; keep implementation details private.

### IV. Respect Boundaries

Extend through existing layers, interfaces, and package boundaries. Do not
duplicate logic, bypass abstractions, or reach into unrelated internals.
Before hardcoding enums, formats, limits, or well-known strings, check stdlib
and existing dependencies for exported constants.

### V. Test-First (Default)

TDD is the default for project-owned logic that introduces or changes behavior:
write the test first, confirm it fails for the right reason, then implement until
it passes (Red-Green-Refactor). Bug fixes SHOULD follow the same pattern with a
regression test first.

TDD is not required for small, low-risk tasks where test-first adds no signal,
including:

- Test-only edits (refactoring tests, renaming helpers, tightening assertions)
- Cosmetic or behavior-preserving refactors with existing coverage
- Documentation, configuration, or chore tasks with no observable behavior change

When TDD is skipped on exempt tasks, existing tests MUST still pass and
Principle VIII validation still applies.

### VI. High-Signal Testing

Add tests only when they justify their cost: new coverage, a distinct behavior,
edge case, or regression lock. Test project-owned logic, contracts, and wiring —
not stdlib, framework, or third-party behavior. One clear behavior per test;
shortest setup and assertions needed. Do not duplicate assertions already covered
elsewhere in the suite.

### VII. Test Structure & Naming

Tests MUST describe observable behavior and contracts, not internals. Put
scenario wording in nested or sub-test names, not in overloaded top-level test
identifiers. Use table-driven or parameterized tests when many cases share the
same shape; use separate sub-tests when setup or assertions differ. Prefer
fail-fast checks so later asserts do not run on bad state. Shared fixtures
belong in dedicated test helper files, marked as helpers per language convention.

### VIII. Validation Before Done

After code changes, review the diff for duplication and scope creep, then run
lint and the affected tests (repo Makefile or CI targets when available). Do not
commit unless explicitly requested. Local validation uses unit and integration
tests only. End-to-end tests require live infrastructure and run in CI; do not
run them locally unless there is an explicit, task-specific reason to do so.

## Additional Constraints

- Comments explain non-obvious intent, constraints, or tradeoffs — not obvious
  code narration. Document public API only; skip boilerplate on internals.
- Top-down file order: public/caller code first, helpers below.
- Error and human-readable messages: assert stable contract fields exactly;
  treat message text loosely (non-empty or stable substring) unless a stable
  error code exists.

## Development Workflow

- Spec Kit phases (`/speckit-specify`, `/speckit-plan`, `/speckit-tasks`,
  `/speckit-implement`) MUST honor this constitution.
- Plans MUST pass the Constitution Check gates before Phase 0 research and again
  after Phase 1 design.
- Tasks that add or change behavior SHOULD order test tasks before implementation
  (Red-Green-Refactor). Exempt tasks (Principle V) omit test-first ordering.
- Tasks MUST not introduce constitution violations; justify any exception in the
  plan Complexity Tracking table.
- Implementation MUST not expand scope beyond approved tasks without a spec/plan
  update.
- `/speckit-tasks` MUST group work into **implementation batches (IB01, IB02, …)**
  sized for a single review and commit. Each IB MUST produce a concrete code diff
  except optional preflight/postflight steps (baseline, final validation).
- If an IB adds, moves, renames, or changes a function signature, that IB MUST
  also update every **in-scope** call site in the same diff. Do not land definitions
  without consumers in the same batch. Partial wiring (one consumer while others
  remain on the old pattern) is forbidden unless tasks explicitly scope a single file.
- Work MUST be split into implementation batches by concern to preserve
  reviewability and testability. See `implementation-batches.md` for batch
  definitions and ordering.
- If an IB spans more than one Feature Requirement **and** the changes involve
  different reviewer mental models (structural refactoring, mechanical safety
  fixes, naming/decomposition), split into separate IBs — one per distinct
  concern. Uniform mirror changes across symmetric files may share one IB
  (horizontal IB pattern).
- Spec Kit workflow SHOULD run `/speckit-analyze` after `/speckit-tasks` and before
  the first `/speckit-implement` to catch IB scope, wiring completeness, and concern-mixing issues.
- Detailed batch rules (consumer inventory, wiring completeness, matrix-audit) live
  in `implementation-batches.md` (companion to this constitution); projects SHOULD
  copy it into `.specify/memory/` or `.specify/templates/`.
- Stacked IB git workflow (branch naming, stack order, tip-only merge) lives in
  `git-workflow.md` (companion to this constitution); projects SHOULD copy it into
  `.specify/memory/`. Coding agents SHOULD acknowledge that workflow without running
  git unless the user explicitly requests it.

## Governance

This constitution supersedes ad-hoc guidance for Spec Kit–driven work in this
repository. Amendments require an explicit `/speckit-constitution` update,
version bump, and sync of dependent templates. All plans and reviews MUST verify
compliance with MUST principles. Complexity that violates minimal-change or
simplicity principles MUST be documented with rejected simpler alternatives.

**Version**: 1.6.5 | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
