---
name: generate
description: Generate progressive disclosure documentation for a repo from scratch. Use when the user wants to create docs, generate docs, or run the docs generator on a repo that has no existing docs/ai/ directory.
---

# generate

Generate progressive disclosure documentation for this repository following the progressive disclosure standard.

## Workflow

### 1. Read the standard

Read `docs/progressive-disclosure-standard.md` to understand the L0/L1/L2 architecture, file naming rules, token budgets, and content density targets.

### 2. Read existing project context

Read all markdown files and config/setup files in the repo root first — these contain project decisions, conventions, and context that should inform the generated docs:

- All `*.md` files (README, CLAUDE.md, AGENTS.md, CONTRIBUTING, CHANGELOG, etc.)
- Config files: package.json, Cargo.toml, go.mod, pyproject.toml, Dockerfile, docker-compose.yml, Makefile, CI config
- Setup scripts: init.sh, bootstrap.sh, setup.py, etc.

### 3. Map and deep-read the codebase

1. List the directory structure (top 3 levels)
2. Identify repo type: api-service, frontend-app, sdk-library, infrastructure, distributed-system, data-pipeline, ml-model
3. Identify the major modules, packages, or service boundaries

**Module definition:** A module is a top-level directory with cohesive functionality: a service in a distributed system, a package in a monorepo, a major feature grouping in an app (e.g., auth/, api/, storage/), or a namespace in an SDK.

For each major module:

- Read all source files (or a representative set if the module is very large)
- Summarize: purpose, key abstractions, public interfaces, internal patterns, external dependencies, gotchas (TODO/FIXME/HACK, complex conditionals, retry logic, environment-specific behavior, race conditions, error swallowing, hard-coded values, implicit assumptions)

**Delegate to sub-agents if:** The module has >15 source files OR reading all files would exceed 50k tokens. Create a task for each module, collect summaries, then proceed.

### 4. Synthesize and plan

Using the module summaries:

- Map the primary data flow across the whole system
- Identify coding conventions consistent across modules
- Identify all external interfaces (APIs, databases, queues, caches)
- Compile gotchas from all modules
- List key topics for each L1 file (3-5 per file)
- Identify 2-4 L2 deep dive topics

**L2 criteria:** Create a deep dive if the topic is >100 lines in L1 alone, OR it has complex multi-step sequences, OR it's frequently modified and needs isolated maintenance, OR it requires code examples and diagrams.

### 5. Generate docs

Create files in this order:

1. `mkdir -p docs/ai/L1/deep_dives`
2. `docs/ai/L0_repo_card.md` — Identity Block + L1 Index
3. L1 files 01 through 08 in `docs/ai/L1/`
4. `docs/ai/L1/deep_dives/_index.md`
5. L2 deep dive files (2-4 minimum)

Also create or update:

- `AGENTS.md` at repo root — use the expanded template from section 4.7 of the progressive disclosure standard. It must include all three sections: **How to Load**, **Git Conventions**, and **Doc Commands**. This makes the repo self-contained — agents get conventions without needing a plugin install.
- `CLAUDE.md` at repo root referencing @AGENTS.md (add reference if file exists, create if not)

### 6. Verify

**Structural checks:**

- All cross-references resolve (relative links between L0 → L1 → L2)
- L0 is under 50 lines
- Each L1 file is 80-200 lines
- Each L1 file starts with a one-line purpose statement
- Each L1 file ends with `## Related Deep Dives`
- Total L1 is under 1,600 lines
- L2 files start with `> **When to Read This:** ...`
- AGENTS.md has How to Load, Git Conventions, and Doc Commands sections
- CLAUDE.md references @AGENTS.md

**Content self-test:**

- Run a spot-check query against the generated docs: "How do I [common task]?" Verify the answer path is clear from L0 → L1 → L2.
- If the query requires jumping between >3 files or the answer is unclear, revise cross-references or add an L2 guide.
