---
name: pr
description: Create a pull request from the current branch. Use when the user wants to open a PR or send changes for review.
---

# pr

Create a pull request from the current branch to the base branch.

## Workflow

### 1. Determine branches

- Current branch: `git branch --show-current`
- Base branch: default to `main`. If `$ARGUMENTS` contains a branch name (e.g., "pr develop"), use that as the base.

If the current branch is the base branch, stop and tell the user to switch to a feature branch first.

### 2. Check for existing PR

Run `gh pr list --head <current-branch>` to check if a PR already exists for this branch. If it does, return the PR URL and stop.

### 3. Verify commits exist

Run `git log <base>..HEAD --oneline` to check if there are commits ahead of the base branch. If there are none, stop and tell the user there's nothing to create a PR for.

### 4. Ensure branch is pushed

Check if the current branch has a remote tracking branch:

- If no remote branch exists: push with `git push -u origin <current-branch>`
- If remote exists but local has unpushed commits: push with `git push`
- If push fails due to divergence, stop and tell the user to resolve the conflict (do not force push)

### 5. Analyze changes

Run these commands in parallel to understand ALL changes in this PR:

- `git log <base>..HEAD --oneline` — see all commit messages
- `git diff <base>...HEAD --stat` — see file changes summary

Analyze all commits (not just the latest) to understand the full scope of changes since branching from base.

### 6. Generate PR title and body

- **Title:** short (under 70 characters), lowercase start, present tense, describes the overall change (not just one commit)
- **Body:**
  ```
  ## Summary
  <1-3 bullet points summarizing the change>

  ## Test plan
  <bulleted markdown checklist of how to verify the change>
  ```

If `$ARGUMENTS` includes text beyond the base branch (e.g., "pr main fix auth bug"), incorporate that context into the title or summary.

### 7. Create the PR

Use `gh pr create --title "..." --body "$(cat <<'EOF' ... EOF)"` with a HEREDOC to create the pull request. This ensures proper formatting.

Return the PR URL when done.

## Error Handling

If `gh` is not installed or not authenticated, stop and tell the user to:

1. Install GitHub CLI: `brew install gh` (macOS) or visit https://cli.github.com
2. Authenticate: `gh auth login`
