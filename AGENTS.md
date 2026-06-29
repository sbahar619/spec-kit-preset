# Agent context — spec-kit-preset

Shared **base** for Spec Kit projects. Not an application repo — no features, tests, or app code here.

## What lives here

| File | Role |
|------|------|
| `constitution.md` | Governance MUSTs — principles from [ai-agent-config](https://github.com/sbahar619/ai-agent-config) coding standards + Spec Kit workflow |
| `implementation-batches.md` | HOW for IB batches (separation, wiring completeness, matrix-audit) |
| `README.md` | New-project init and preset rollout — **read before changing consumer projects** |

## When editing this repo

- **Principle / coding-standard changes** → update `constitution.md` (sync from ai-agent-config when applicable); bump version.
- **IB / tasks / implement workflow** → update `implementation-batches.md`; bump version.
- **Init or rollout steps** → update `README.md` only.
- Do **not** add project-specific specs, skills, or `.specify/` trees here.

## Consumer projects

Rollout is **per target repo** (see README): seed files with `cp`, then `/speckit-constitution` in that project. Do not implement feature work in this preset repo when the user meant a consumer project.
