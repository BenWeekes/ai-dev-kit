# ai-devkit for Codex

## Install

```bash
git clone https://github.com/AgoraIO-Community/ai-devkit.git
```

Point Codex at `skills/ai-devkit/SKILL.md` as the entry point.

## What it provides

- **Git conventions** — injected at session start (lowercase commits, no AI tool names, present tense)
- **Git skills** — ship (commit+push), pr (create PR), sync (rebase onto main)
- **Docs skills** — generate (create docs from scratch), update (refresh after changes), test (verify docs)

## Files

- `skills/ai-devkit/SKILL.md` — main conventions and skill directory
- `skills/ai-devkit/git/` — git workflow skills
- `skills/ai-devkit/docs/` — documentation skills
