---
name: git-workflow-ps-tech
description: >-
  Defines the PS-Tech Git/GitHub workflow: linear history, rebase-first integration, draft PRs early,
  promote flows, revert (not rewrite) on shared history, staging/production incidents, Odoo.sh sync.
  Inspects branches, PRs, and checks with GitHub CLI (gh) when available.
  Use when discussing team branching, rebasing, promoting, reverting, Odoo.sh, incidents, or reading repo state via gh.
alwaysApply: false
---

# Ps-Tech Git / GitHub workflow

## Scope

- Encode **PS-Tech** defaults: linear history, **rebase** over merge for integrating work, **draft PRs early**, clean history before review, **revert** instead of rewriting shared history.
- **Read GitHub state** with **`gh`** when available (schemas first—see [reference.md](reference.md)).
- **Mutating git** (rebase, push, revert): only when the user **explicitly** asks to run commands or recover from a described situation. Do not push, force-push, merge on GitHub, approve, or comment on PRs unless the user requests it.

## Inspecting branches and PRs

1. Run from the **repository root** (or `cd` there first). If not a git repo, say so and stop there.
2. Prefer **`gh`**: `gh auth status`; then list/view PRs and checks (see [reference.md](reference.md)).
3. **Branches:** `git branch -a`, `git status`; optional `gh api repos/{owner}/{repo}/branches` if JSON is needed (`gh repo view --json nameWithOwner` for owner/repo).

## Linear history and branch roles

- **`production`** and **`staging`**: staging tracks production, often with a few extra commits. Updating production means moving it **forward** to staging (conceptually), using the flows below—not merge-button shortcuts that scatter merge commits.
- **Dev branches** sit **on top of `staging`**. If `staging` advances before the feature is ready, **rebase** the dev branch onto `staging`.

## Merge vs rebase

- **Goal:** land commits onto another branch. **`git rebase`** moves your branch on top of the target and keeps history **linear**—this is the team preference.
- Avoid **`git merge`**.

## GitHub merges

- Merge buttons or `gh` actions often behave like **non–fast-forward** merges and **break linearity**. Prefer **CLI** (`git checkout`, `git rebase`, `git push`) for control.

## Rebase conflicts

1. Find conflict markers (`<<<<<<<`).
2. Fix the files.
3. `git add <file>`
4. `git rebase --continue`
5. Escape hatch: `git rebase --abort`

## Pull request etiquette

- Open a **draft PR early** so others see what you are doing.
- **Push regularly** (e.g. daily or when switching tasks) to reduce lost work.
- Mark **ready for review** and assign a reviewer when appropriate.
- Before review, **clean up** history: interactive rebase—squash, fixup, reword (`git rebase -i HEAD~X`). Aim for **one commit per feature** (often one task per feature).

## Staying current and merging (commands)

Use **`git fetch`** first; rebase onto **`origin/<branch>`** so the base is the latest **remote** tip. Full command sequences for updating dev from staging, merging dev → staging, staging → production, promotion up to a commit, excluding a middle commit, Odoo.sh trigger, reverts, and force-push policy are in [reference.md](reference.md).

## After you push: treat commits as immutable

Once others may have your commits, treat them as **immutable**. Follow-ups are **new commits** with clear **why** in the body—not silent history rewrites on shared branches.

## Commit messages

The training deck stresses a short first line and a **verbose body focused on why**. For bracketed **`[TAG]`**, **`[TASK]`**, module prefix, and the **80-character** header rule, use the project skill [commit-message-format](../commit-message-format/SKILL.md).

## Related skills

- **Commit headers and tags:** [commit-message-format](../commit-message-format/SKILL.md).
- **Full read-only PR review report** (severity, confidence, workspace validation): use the **`review-github`** skill when the user wants a production-style review; this skill focuses on **workflow and discovery/read**.

## Additional reference

- Command cheat sheets and **`gh`** read patterns: [reference.md](reference.md).
