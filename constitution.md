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

### V. High-Signal Testing

Add tests only when they justify their cost: new coverage, a distinct behavior,
edge case, or regression lock. Test project-owned logic, contracts, and wiring —
not stdlib, framework, or third-party behavior. One clear behavior per test;
shortest setup and assertions needed. Do not duplicate assertions already covered
elsewhere in the suite.

### VI. Test Structure & Naming

Tests MUST describe observable behavior and contracts, not internals. Put
scenario wording in nested or sub-test names, not in overloaded top-level test
identifiers. Use table-driven or parameterized tests when many cases share the
same shape; use separate sub-tests when setup or assertions differ. Prefer
fail-fast checks so later asserts do not run on bad state. Shared fixtures
belong in dedicated test helper files, marked as helpers per language convention.

### VII. Validation Before Done

After code changes, review the diff for duplication and scope creep, then run
lint and the affected tests (repo Makefile or CI targets when available). Do not
commit unless explicitly requested.

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
- Tasks MUST not introduce constitution violations; justify any exception in the
  plan Complexity Tracking table.
- Implementation MUST not expand scope beyond approved tasks without a spec/plan
  update.

## Governance

This constitution supersedes ad-hoc guidance for Spec Kit–driven work in this
repository. Amendments require an explicit `/speckit-constitution` update,
version bump, and sync of dependent templates. All plans and reviews MUST verify
compliance with MUST principles. Complexity that violates minimal-change or
simplicity principles MUST be documented with rejected simpler alternatives.

**Version**: 1.0.0 | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
