# spec-kit-preset

Shared Spec Kit constitution and IB batch guidance for reuse across projects.

[Spec Kit preset](https://github.github.io/spec-kit/reference/presets.html): `preset.yml` + templates + memory companions.

Preset path (adjust if yours differs): `/home/fedora/spec-kit-preset`

## Layout

| Path | Seeded? | Role |
|------|---------|------|
| `preset.yml` | — | Preset manifest; overrides core `constitution-template` |
| `templates/constitution-template.md` | yes | Governance MUSTs |
| `memory/implementation-batches.md` | yes | IB rules HOW |
| `memory/git-workflow.md` | yes | Stacked IB git workflow (agents ack only) |
| `docs/git-troubleshooting.md` | no | User ops when stacked git breaks |

Constitution principles derive from [sbahar619/ai-agent-config](https://github.com/sbahar619/ai-agent-config). IB and git workflow evolve in this repo only.

## Versions (what to bump)

| Required | When |
|----------|------|
| `preset.yml` — `schema_version`, `preset.version`, `requires.speckit_version` | Spec Kit CLI enforces on `preset add` |
| `templates/constitution-template.md` — `**Version**` | `/speckit-constitution` reads and bumps it |

Memory docs and `docs/` have no version fields — use git history for rollout tracking.

## New project init

Run in the **target repo** after clone or create.

### Step 0 — Spec Kit scaffold

```bash
cd /path/to/your-project
specify init . --force --integration cursor-agent
```

### Step 1 — Install preset and copy memory files

```bash
specify preset add --dev /home/fedora/spec-kit-preset --priority 5
P=.specify/presets/spec-kit-preset
cp "$P/templates/constitution-template.md" .specify/memory/constitution.md
cp "$P/memory/"*.md .specify/memory/
```

`cp` copies content only — not placeholders or template sync.

### Step 2 — `/speckit-constitution` (required)

```text
Adopt spec-kit-preset constitution (Version from the template). Project name: <your-project>.
Fill all placeholders, set ratification/amended dates, and sync dependent templates
(plan, spec, tasks) with the constitution.
```

### Step 3 — First feature

```text
/speckit-specify → /speckit-plan → /speckit-tasks → /speckit-analyze → /speckit-implement
```

## Update an existing project

1. Edit files here (see **Edit the base**); bump constitution `**Version**` or `preset.version` when applicable.
2. In the target project: **re-run Step 1** from above.
3. Run **`/speckit-constitution`** for constitution changes (merge/update — avoid blind overwrite if the project has local governance edits).
4. Do **not** hand-edit project skills or existing `specs/*` as part of preset rollout.

## Edit the base (this repo)

| Change | File | Bump |
|--------|------|------|
| Principles / coding standards | `templates/constitution-template.md` | constitution `**Version**` |
| IB / tasks / implement workflow | `memory/implementation-batches.md` | — |
| Stacked-branch workflow | `memory/git-workflow.md` | — |
| Git conflict playbook | `docs/git-troubleshooting.md` | — |
| Preset manifest or layout | `preset.yml` | `preset.version` |
| Init or rollout steps | `README.md` | — |

Do **not** add project-specific specs, skills, or `.specify/` trees here.
