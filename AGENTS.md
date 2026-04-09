# Agent Conventions

ai-devkit provides git conventions and documentation generation as skills for AI coding agents.

## Repo Structure

| Path                         | Purpose                                       |
| ---------------------------- | --------------------------------------------- |
| `skills/ai-devkit/SKILL.md` | entry point — conventions + skill directory   |
| `skills/ai-devkit/git/`     | git workflow skills (ship, pr, sync)          |
| `skills/ai-devkit/docs/`    | documentation skills (generate, update, test) |
| `hooks/`                     | session-start hook, platform wrappers         |
| `docs/`                      | documentation standard + orchestration guide  |
| `.claude-plugin/`            | Claude Code plugin config                     |
| `.cursor-plugin/`            | Cursor plugin config                          |
| `.codex/`                    | Codex install guide                           |
| `gemini-extension.json`      | Gemini extension config                       |

## Architecture

- **Ambient conventions** — `hooks/session-start` reads `skills/ai-devkit/SKILL.md` and injects git conventions and the skill directory into the session context at startup. No user action needed.
- **On-demand skills** — detailed workflows (ship, pr, sync, docs) are loaded via the Skill tool when invoked.

## Conventions

1. **Commit messages:** lowercase start, no AI tool names, present tense.
2. **Spec before plan, plan before code.** Separate WHAT from HOW.
3. **Test driven development.** Write the test first, verify it fails, implement, verify it passes.
4. **Review before commit.** AI review checks spec compliance and code quality.

## AGENTS.md as Primary Delivery

The AGENTS.md template defined in [section 4.7 of the progressive disclosure standard](docs/progressive-disclosure-standard.md#47-agentsmd-and-claudemd-integration) is the primary way repos adopt git conventions and doc commands. Any repo that creates an AGENTS.md from that template gets conventions without installing ai-devkit as a plugin. The plugin (skills + hooks) is a convenience for tools that support it.

## Complementary Skills

- [Superpowers](https://github.com/obra/superpowers) — spec, plan, TDD, review workflow
