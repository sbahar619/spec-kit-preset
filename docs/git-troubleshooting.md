# Git Troubleshooting — Stacked IB Workflow

**Purpose**: Playbook when stacked IB git goes wrong. User-operational — **not** part of the preset; not seeded to `.specify/memory/`.

**Companion to**: `../git-workflow.md` (agent-ack workflow at preset root).

**Version**: 1.0.0

---

## When things break

Usually caused by merging an early stack PR to `main` while later PRs were still open, or rebasing full history onto a sibling.

**Symptoms**: `skipped previously applied commit`; conflicts on an older IB when the target already has later IB work; import-only conflicts (HEAD = later IB, incoming = earlier IB).

**Prevention**: follow tip-only merge in `git-workflow.md`.

---

## Recovery: reset + cherry-pick deltas

Do not rebase tip onto sibling with full stack ancestry.

```bash
git log <parent-branch>..<tip-branch> --oneline   # delta commits only
git checkout <tip-branch>
git reset --hard <parent-branch>
git cherry-pick <commit-sha>                       # each delta, in order
```

---

## Conflict resolution

| Situation | Keep |
|-----------|------|
| Later IB vs earlier IB on same file | **Later IB** (parent you reset onto, or HEAD during cherry-pick) |
| Import-only conflict | Imports matching the **IB being applied** + bodies already in file |
| Skipped commits + one conflict | Skipped = duplicate (safe); resolve the one conflict with later-IB rule |

After partial merge to `main`: rebase the next unmerged branch onto `main`, not a stale sibling.

```bash
git checkout 001/ib02-resourcemanagers-test-consistency
git rebase main
```

**Version**: 1.0.0
