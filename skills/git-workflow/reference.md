# PS-Tech workflow — reference

## GitHub CLI: reading branches and PRs

Run from the repo root after `gh auth status` succeeds.

| Goal | Command |
|------|---------|
| Current repo | `gh repo view --json nameWithOwner,defaultBranchRef` |
| PR for current branch | `gh pr status` |
| List PRs | `gh pr list` (add `--state open` / `--author @me` as needed) |
| PR metadata (JSON) | `gh pr view <N> --json title,state,isDraft,baseRefName,headRefName,url,author,commits` |
| Checks | `gh pr checks <N>` |
| Branch list (remote) | `git branch -a` |
| Remote branches via API | `gh api repos/{owner}/{repo}/branches --paginate` (substitute owner/repo from `gh repo view`) |

Resolve `{owner}/{repo}` once:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

## Flow cheat sheets (training slides)

Replace `<staging>`, `<production>`, `<dev>` with your branch names.

### Pull staging into a dev branch

```bash
git fetch
git checkout <dev>
git rebase origin/<staging>
```

### Merge dev into staging (safe, linear)

```bash
git fetch
git checkout <dev>
git rebase origin/<staging>
git checkout <staging>
git pull
git rebase <dev>
git push
```

### Merge staging into production

```bash
git fetch
git checkout <production>
git rebase origin/<staging>
git push
```

### Production only up to a given commit

Takes commits between current `production` and `<commit>`, inclusive:

```bash
git fetch
git checkout <production>
git rebase <commit-hash>
git push
```

### Exclude a “middle” commit when promoting

When staging has commits you do not want all of in production, and an unwanted commit sits **in the middle** of the history: revert it on `staging`, rebase production onto staging, push, then **revert the revert** on `staging` so staging stays consistent (full sequence is delicate—confirm with the team before running).

```bash
git fetch
git checkout <staging>
git revert <commit-to-exclude>
git checkout <production>
git rebase <staging>
git push
git checkout <staging>
git revert <revert-commit-sha>
git push
```

### Odoo.sh out of sync with GitHub

Wait **10–15 minutes**. If still stale, trigger with an **empty commit**:

```bash
git commit --allow-empty -m "[REL] Trigger build"
git push
```

Use **`[REL]`** and messaging consistent with [commit-message-format](../commit-message-format/SKILL.md).

### Wrong commit merged into staging

Avoid rewriting shared history. **Revert** instead:

```bash
git checkout <staging>
git revert <commit>
git push
```

### Bad commit in production

Revert on production, then **re-align staging** onto production:

```bash
git checkout <production>
git revert <commit>
git push
git checkout <staging>
git rebase <production>
git push
```

### Force push

Routine work should **not** need `git push --force`. Legitimate cases from training: rebasing a **personal dev** branch on staging, or **repairing** a repo that did not follow this process. If force-push is truly required:

```bash
git push --force-with-lease
```

Never prefer `--force` over `--force-with-lease`.

## Other commands (training)

| Situation | Command |
|-----------|---------|
| Save WIP, switch branches | `git stash save -u "message"` then later `git stash pop` |
| Bring one commit from another branch | `git cherry-pick <commit>` |
| Move HEAD (careful) | `git reset <commit>` |
| Discard local work to match a branch | `git reset --hard <branch>` |
