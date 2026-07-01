# spec-kit-preset

Shared Spec Kit constitution and IB batch guidance for reuse across projects.

Preset path (adjust if yours differs): `/home/fedora/spec-kit-preset`

## Sources

**Constitution** (`constitution.md`) — core coding principles (Minimal Change, Simplicity, Testing, Validation, etc.) are derived from the coding-standard rules in [sbahar619/ai-agent-config](https://github.com/sbahar619/ai-agent-config). When those rules change upstream, refresh the constitution here and bump its version before rolling out to projects.

**Implementation batches** (`implementation-batches.md`) — Spec Kit–specific IB workflow; not sourced from ai-agent-config. Evolves in this repo only.

**Git workflow** (`git-workflow.md`) — stacked IB branches, tip-only merge; agent ack. Seeded to consumer projects.

**Git troubleshooting** (`docs/git-troubleshooting.md`) — conflict recovery; user ops, not preset.

## Files

| File | Seeded to consumer? | Purpose |
|------|---------------------|---------|
| `constitution.md` | yes (via `/speckit-constitution`) | Core principles and workflow MUSTs (v1.6.2+) |
| `implementation-batches.md` | yes (`cp`) | IB rules (v1.4.0+) |
| `git-workflow.md` | yes (`cp`) | Agent-ack git workflow (v1.3.0+) |
| `docs/` | no | User-operational notes (not part of preset) |

## New project init

Run these steps **in the target repo** after cloning or creating the project.

### Step 0 — Spec Kit scaffold

```bash
cd /path/to/your-project
specify init . --force --integration cursor-agent
```

Creates `.specify/`, `.cursor/skills/speckit-*`, and default templates.

### Step 1 — Seed preset files

```bash
cp /home/fedora/spec-kit-preset/constitution.md .specify/memory/constitution.md
cp /home/fedora/spec-kit-preset/implementation-batches.md .specify/memory/implementation-batches.md
cp /home/fedora/spec-kit-preset/git-workflow.md .specify/memory/git-workflow.md
```

`cp` copies content only. It does **not** fill project placeholders or sync templates.

### Step 2 — `/speckit-constitution` (required for constitution)

In Cursor, run **`/speckit-constitution`** with a prompt like:

```text
Adopt spec-kit-preset constitution v1.6.2. Project name: <your-project>.
Fill all placeholders, set ratification/amended dates, and sync dependent templates
(plan, spec, tasks) with the constitution.
```

This step:

- Replaces `[PROJECT_NAME]`, `[RATIFICATION_DATE]`, `[LAST_AMENDED_DATE]`
- Aligns `.specify/templates/*` with constitution MUSTs
- Writes a Sync Impact Report on the constitution file

**Do not skip** Step 2 if you only `cp` constitution — placeholders and templates will drift.

### Step 3 — First feature

Use the normal Spec Kit flow in the project:

```text
/speckit-specify → /speckit-plan → /speckit-tasks → /speckit-analyze → /speckit-implement
```

Feature folders under `specs/` are project-local; they are not copied from this preset.

---

## Update an existing project (preset rollout)

1. Change files **here** first; bump version in the edited preset file.
2. In the target project:
   - **`implementation-batches.md`**, **`git-workflow.md`**: `cp` from preset → `.specify/memory/`.
   - **`constitution.md`**: run **`/speckit-constitution`** with the new preset version in the prompt (merge/update — avoid blind overwrite if the project has local governance edits).
3. Do **not** hand-edit project skills or existing `specs/*` folders as part of preset rollout.

---

## `cp` vs `/speckit-constitution`

| Artifact | `cp` | `/speckit-constitution` |
|----------|------|-------------------------|
| `constitution.md` | Seed content | **Required** — placeholders + template sync |
| `implementation-batches.md` | **Required** — only delivery path today | Not handled (future: init hook) |
| `git-workflow.md` | **Required** — only delivery path today | Not handled |

---

## Edit the base (this repo)

1. **Principles changed in [ai-agent-config](https://github.com/sbahar619/ai-agent-config)?** Update `constitution.md` to match, then bump its version.
2. **IB / task workflow changed?** Update `implementation-batches.md` and bump its version.
3. **Stacked-branch workflow changed?** Update `git-workflow.md` and bump its version.
4. **Git conflict playbook changed?** Update `docs/git-troubleshooting.md` and bump its version.
5. Roll out to projects per **Update an existing project** above.

## Companion docs

| Document | Seeded? | Level |
|----------|---------|--------|
| **constitution** | yes | MUST — review-sized IBs, in-scope call sites, concern separation |
| **implementation-batches** | yes | HOW — consumer inventory, wiring completeness, matrix-audit |
| **git-workflow** | yes | HOW — branch naming, stack, tip-only merge; agents ack only |
| **docs/git-troubleshooting** | no | User ops — not part of preset |

Keep IB rules in `implementation-batches.md` and agent git context in `git-workflow.md`.
