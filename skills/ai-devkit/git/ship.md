---
name: ship
description: Commit staged changes and push to remote. Use when the user says "ship it", "commit and push", or wants to send changes to the remote.
---

# ship

Commit the currently staged changes and push them to the remote.

## Rules

- Do NOT add a Co-Authored-By trailer
- Do NOT modify git config (user.name, user.email)
- Do NOT skip hooks (no --no-verify)
- NEVER use --force or --force-with-lease
- NEVER amend commits after hook failures (always create a new commit)
- Do NOT create commits on detached HEAD

## Workflow

### 1. Pre-flight checks

Run `git status --porcelain -b` to verify:

- On a branch (not detached HEAD) — if detached, stop and tell user
- Staged changes exist — if none, stop and tell user to stage files first

### 2. Generate a commit message

If `$ARGUMENTS` is provided, use it as the commit message.

Otherwise, generate a commit message by:

1. Run `git diff --cached` to read the staged changes
2. Run `git log --oneline -5` to see recent commit style
3. Check for project conventions in CLAUDE.md, .gitmessage, or similar
4. Generate a message following these rules:
   - lowercase start, present tense, no AI tool names
   - one concise line focusing on "why" not just "what"
   - follow the project's existing commit style

### 3. Commit

Run `git commit -m "<message>"`.

If the commit fails due to hooks:

- Fix the issue the hook identified
- Re-stage fixed files
- Create a NEW commit (do NOT use --amend)

### 4. Push

Attempt to push to the remote tracking branch:

- If tracking branch exists: `git push`
- If no tracking branch: `git push -u origin <current-branch>`

If push fails:

- **Non-fast-forward / behind remote:** Run `git pull --rebase`, resolve any conflicts, then retry push
- **No remote configured:** Stop and tell user to configure a remote
- **Network failure:** Stop and report the error
- **Rejected by remote hooks:** Stop and report the error

After successful push, confirm the commit SHA and remote branch.
