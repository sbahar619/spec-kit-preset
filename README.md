# spec-kit-preset

Shared Spec Kit constitution and task-batch guidance for reuse across projects.

## Files

| File | Purpose |
|------|---------|
| `constitution.md` | Core principles and workflow MUSTs (v1.4.0+) |
| `implementation-batches.md` | How `/speckit-tasks` splits IB01+ for review-sized commits; func-def + call-site rule |

## Use in a project

After `specify init . --force --integration cursor-agent`:

```bash
cp /home/fedora/spec-kit-preset/constitution.md .specify/memory/constitution.md
cp /home/fedora/spec-kit-preset/implementation-batches.md .specify/memory/implementation-batches.md
```

Edit placeholders in the constitution copy: `[PROJECT_NAME]`, `[RATIFICATION_DATE]`,
`[LAST_AMENDED_DATE]`.

### Optional: wire into task generation

So `/speckit-tasks` picks up batch rules automatically, add to the project’s
`.cursor/skills/speckit-tasks/SKILL.md` (Outline step 2):

```markdown
- **IF EXISTS**: Load `.specify/memory/implementation-batches.md` when generating tasks.md
```

Or embed the “Implementation Batches” section from `implementation-batches.md` into
`.specify/templates/tasks-template.md`.

## Edit the base

1. Change files in this repo.
2. Re-copy into projects to refresh.
3. Bump constitution **Version** when workflow/principles change.
4. Bump `implementation-batches.md` version when batch rules change.

## Constitution vs implementation-batches

| Document | Level |
|----------|--------|
| **constitution** | MUST principles — review-sized IBs, func-def + call-site rule, refactor/coverage split |
| **implementation-batches** | HOW for prompts — preflight/IB/postflight tables, define+call-site pairing, checklists, examples |

Keep detailed split rules out of the constitution; reference the companion doc instead.
