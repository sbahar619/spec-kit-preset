# Stacked IB Git Workflow (Spec Kit)

**Purpose**: Tell coding agents how this project manages git for Spec Kit IB work. User runs all git commands.

**Authority**: Companion to constitution **Development Workflow**. Seeded to `.specify/memory/` in consumer projects.

---

## Stack and naming

```text
main → {spec-id}/ib01-{spec-name} → ib02 → … → ibNN
```

Next spec branches from the prior spec's last IB (e.g. `002/ib01-refactor-resourceclient` from `001/ib05-resourcemanagers-test-consistency`).

**Branch format**: `{spec-id}/ib{NN}-{spec-name}` — `{spec-id}` from `specs/001-…`, `ib{NN}` zero-padded.

Examples: `001/ib01-resourcemanagers-test-consistency`, `002/ib04-refactor-resourceclient`.

One IB = one commit = one PR. Branch each IB from the previous open PR branch.

---

## Merge to main

**Review stacked, merge tip only.** Open a PR per IB for review and CI (stacked: ib02 targets ib01, …). Do **not** merge early stack PRs to `main`. When the spec is approved, merge only the tip branch `{spec-id}/ibNN-{spec-name}` into `main` (base `main`, compare tip).

If `main` moved while the stack was open, rebase the tip branch onto `main` once before merging.

---

## Agent role

When the user mentions IB branches or stacked PRs: read this file, acknowledge the workflow, do not re-explain it. **Do not** run mutating git, commit, push, or open PRs unless explicitly asked.

Conflict recovery is user-operational — see `docs/git-troubleshooting.md` (not part of the preset).
