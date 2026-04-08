---
name: sync
description: Rebase current branch onto the latest main. Use when the user wants to sync, rebase, or pull latest changes from main.
---

# sync

Pull the latest changes from the main branch and rebase the current branch on top.

## Workflow

### 1. Check working tree status

Run `git status --porcelain`.

If there are uncommitted changes:

- Report the status
- Ask the user if they want to stash changes before proceeding
- If yes, run `git stash` with a clear message
- If no, abort and tell them to commit or stash first

### 2. Determine base branch

Check which base branch exists (prefer the first found):

- `origin/main`
- `origin/master`
- `origin/develop`

If none exist, report error and stop.

### 3. Check if already on base branch

Run `git rev-parse --abbrev-ref HEAD`.

If on main/master/develop, switch strategy:

- Run `git pull origin <branch>`
- Report number of new commits if any
- Skip rebase steps

### 4. Fetch latest

Run `git fetch origin`.

### 5. Check if already up to date

Run `git merge-base HEAD origin/<base>` and `git rev-parse origin/<base>`.

If identical, report "Already up to date" and stop.

### 6. Rebase

Run `git rebase origin/<base>`.

### 7. Report outcome

If successful:

- Count commits rebased: `git rev-list --count origin/<base>..HEAD`
- Report: "Rebased N commit(s) onto <base>"

### 8. Handle conflicts

If the rebase encounters conflicts:

- Run `git status` to identify conflicting files
- Report which files have conflicts
- Tell the user: "Resolve conflicts, then run: git rebase --continue"
- Do NOT force resolve or use `--skip`/`--abort` without the user asking

### 9. Restore stash (if stashed)

If changes were stashed in step 1, run `git stash pop`.

If stash pop causes conflicts, report them and stop.
