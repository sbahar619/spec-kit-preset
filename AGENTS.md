# Agent context — spec-kit-preset

Shared **base** for Spec Kit projects — not an application repo. No features, tests, or app code here.

**Editing this repo** → [README.md § Edit the base](README.md#edit-the-base-this-repo)

**Rollout to consumer projects** → [README.md § New project init](README.md#new-project-init)

## Agent behavior (consumer projects)

When the user mentions **IB branches** or **stacked PRs**: read `.specify/memory/git-workflow.md`, acknowledge it, do not re-explain. Do not run mutating git unless explicitly asked.

Do not implement feature work in this preset repo when the user meant a consumer project.
