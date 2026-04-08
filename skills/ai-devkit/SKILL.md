---
name: ai-devkit
description: Git conventions and documentation generation for AI-assisted development. Enforces lowercase commits, no AI tool names, present tense. Use when committing, pushing, creating PRs, rebasing, or generating repo documentation.
---

# ai-devkit

Git conventions and progressive disclosure documentation for AI-assisted development.

## Git Conventions (always active)

These rules apply to every commit in repos that install ai-devkit:

- **Lowercase start** — commit messages begin with a lowercase letter
- **No AI tool names** — never mention claude, cursor, copilot, cody, aider, gemini, codex, chatgpt, or gpt-3/4
- **Present tense** — "add feature", not "added feature"
- **No Co-Authored-By trailers** — omit AI attribution lines
- **No --no-verify** — let git hooks run normally
- **No git config changes** — do not modify user.name or user.email

## Available Skills

### git

Git workflow skills for committing, pushing, PRs, and rebasing. For detailed workflows, read the skill file before executing.

| Skill  | Description                                   | Workflow                                        |
| ------ | --------------------------------------------- | ----------------------------------------------- |
| `ship` | commit staged changes and push to remote      | Read `skills/ai-devkit/git/ship.md`            |
| `pr`   | create a pull request from the current branch | Read `skills/ai-devkit/git/pr.md`              |
| `sync` | rebase current branch onto latest main        | Read `skills/ai-devkit/git/sync.md`            |

### docs

Documentation generation following the progressive disclosure standard. For detailed workflows, read the skill file before executing.

| Skill      | Description                                                 | Workflow                                        |
| ---------- | ----------------------------------------------------------- | ----------------------------------------------- |
| `generate` | generate L0/L1/L2 docs for the repo from scratch            | Read `skills/ai-devkit/docs/generate.md`       |
| `update`   | update existing docs after code changes — only what changed | Read `skills/ai-devkit/docs/update.md`         |
| `test`     | verify generated docs meet the standard                     | Read `skills/ai-devkit/docs/test.md`           |
