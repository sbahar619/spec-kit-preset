# spec-kit-preset

Shared Spec Kit constitution and IB batch guidance for reuse across projects.

Preset path (adjust if yours differs): `/home/fedora/spec-kit-preset`

## Sources

**Constitution** (`constitution.md`) — core coding principles (Minimal Change, Simplicity, Testing, Validation, etc.) are derived from the coding-standard rules in [sbahar619/ai-agent-config](https://github.com/sbahar619/ai-agent-config). When those rules change upstream, refresh the constitution here and bump its version before rolling out to projects.

**Implementation batches** (`implementation-batches.md`) — Spec Kit–specific IB workflow; not sourced from ai-agent-config. Evolves in this repo only.

## Files

| File | Purpose |
|------|---------|
| `constitution.md` | Core principles and workflow MUSTs (v1.6.0+) |
| `implementation-batches.md` | IB rules — separation, wiring completeness, matrix-audit (v1.4.0+) |

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
```

`cp` copies content only. It does **not** fill project placeholders or sync templates.

### Step 2 — `/speckit-constitution` (required for constitution)

In Cursor, run **`/speckit-constitution`** with a prompt like:

```text
Adopt spec-kit-preset constitution v1.6.0. Project name: <your-project>.
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
   - **`implementation-batches.md`**: `cp` from preset → `.specify/memory/` (no slash command yet).
   - **`constitution.md`**: run **`/speckit-constitution`** with the new preset version in the prompt (merge/update — avoid blind overwrite if the project has local governance edits).
3. Do **not** hand-edit project skills or existing `specs/*` folders as part of preset rollout.

---

## `cp` vs `/speckit-constitution`

| Artifact | `cp` | `/speckit-constitution` |
|----------|------|-------------------------|
| `constitution.md` | Seed content | **Required** — placeholders + template sync |
| `implementation-batches.md` | **Required** — only delivery path today | Not handled (future: init hook) |

---

## Edit the base (this repo)

1. **Principles changed in [ai-agent-config](https://github.com/sbahar619/ai-agent-config)?** Update `constitution.md` to match, then bump its version.
2. **IB / task workflow changed?** Update `implementation-batches.md` and bump its version.
3. Roll out to projects per **Update an existing project** above.

## Constitution vs implementation-batches

| Document | Level |
|----------|--------|
| **constitution** | MUST — review-sized IBs, in-scope call sites, concern separation, analyze before implement |
| **implementation-batches** | HOW — consumer inventory, wiring completeness, horizontal IB, matrix-audit, Done when |

Keep detailed IB rules in `implementation-batches.md`; the constitution references that companion doc.
