# AI-Dev Kit

A practical guide and toolkit for consistent AI-assisted development. Works with
any AI coding agent — the practices are about how your team works, not which
tool you use.

This repo provides two things: a guide to the practices that make AI-assisted
development reliable across a team, and installable skills that teach AI agents
to follow those practices automatically.

## Table of Contents

- [1. AI Documentation Standard](#1-ai-documentation-standard)
- [2. Skills in This Repo](#2-skills-in-this-repo)
  - [Install](#install)
  - [/git — Git workflow conventions](#git--git-workflow-conventions)
  - [/docs — Documentation generation](#docs--documentation-generation)
- [3. Additional Skills](#3-additional-skills)
  - [Superpowers — Development workflow](#superpowers--development-workflow)
  - [Spec Kit — Executable specifications](#spec-kit--executable-specifications)
  - [Using skills together](#using-skills-together)
- [4. Multi-Repo Orchestration](#4-multi-repo-orchestration)

---

## 1. AI Documentation Standard

Every repo should be self-describing for AI agents. Without structured
documentation, agents either load the entire codebase (expensive, slow,
error-prone) or hallucinate from incomplete context.

The [Progressive Disclosure Documentation Standard](https://github.com/BenWeekes/ai-dev/blob/main/progressive-disclosure-standard.md)
defines a three-level architecture that gives agents exactly the context they
need:

| Level  | Name       | What It Is                                              | Token Budget |
| ------ | ---------- | ------------------------------------------------------- | ------------ |
| **L0** | Repo Card  | Identity + L1 index. Always read first. Single file.    | 300-500      |
| **L1** | Summaries  | Structured summaries for standard work. 8 fixed files.  | 300-600 each |
| **L2** | Deep Dives | Full specs and subsystem docs. Loaded only when needed. | No ceiling   |

Docs live in `docs/ai/` in each repository. Start at L0, load all 8 L1 files
at session start (they're small), go to L2 only when L1 isn't detailed enough.

The `/docs` skill in this repo generates these docs automatically — see below.

---

## 2. Skills in This Repo

Skills are plain markdown files that teach AI agents specific workflows and
conventions. The agent follows the rules because the skill instructions say so —
no hooks, no interceptors, no build steps.

### Install

Add as a skill dependency for your AI coding tool. For Claude Code:

```bash
claude mcp add-skill https://github.com/BenWeekes/ai-dev-kit
```

Or clone and reference locally:

```bash
git clone https://github.com/BenWeekes/ai-dev-kit.git
```

### /git — Git workflow conventions

Commit, push, and PR commands that enforce consistent conventions across your
team:

- **lowercase start** — commit messages and PR titles start lowercase
- **present tense** — "add feature" not "added feature"
- **no AI tool names** — no mentions of claude, cursor, copilot, etc. in commits
- **no Co-Authored-By** — clean commit authorship

| Command      | What It Does                                                             |
| ------------ | ------------------------------------------------------------------------ |
| `/git:ship`  | Commit staged changes and push (enforces conventions above)              |
| `/git:pr`    | Create a PR from current branch to main with generated title and summary |
| `/git:sync`  | Pull latest from main, rebase current branch on top                      |

### /docs — Documentation generation

Generates progressive disclosure documentation (L0/L1/L2) for any repository,
following the [Progressive Disclosure Documentation Standard](https://github.com/BenWeekes/ai-dev/blob/main/progressive-disclosure-standard.md).

The agent reads the standard, maps your codebase, and generates the full
`docs/ai/` structure — repo card, 8 L1 summaries, deep dives, and the
`AGENTS.md`/`CLAUDE.md` entry points.

| Command | What It Does                                                   |
| ------- | -------------------------------------------------------------- |
| `/docs` | Generate L0/L1/L2 docs for the repo following the PD standard |

---

## 3. Additional Skills

Skills are independent — install what you need. These complement ai-dev-kit
well.

### Superpowers — Development workflow

A complete development workflow: spec, plan, TDD, review. Provides
subagent-per-task dispatch, two-stage review, systematic debugging, and model
selection by task complexity. Use this when you want a structured process for
turning requirements into tested, reviewed code.

Install: [github.com/obra/superpowers](https://github.com/obra/superpowers)

**What it provides:**
- `/spec` — capture requirements before planning
- `/plan` — plan implementation approach for a spec
- `/tdd` — implement using test-driven development
- `/review` — review changes before committing

### Spec Kit — Executable specifications

Spec-driven development with executable specifications and multi-step
refinement. Use this when you want specs that can be validated automatically
against the implementation — particularly useful for API contracts and data
schemas.

Install: [github.com/github/spec-kit](https://github.com/github/spec-kit)

### Using skills together

A typical workflow combining ai-dev-kit + Superpowers:

1. `/spec` — capture what you want to build
2. `/plan` — plan how to build it
3. `/tdd` — implement with tests
4. `/review` — review the changes
5. `/git:ship` — commit and push (conventions enforced)
6. `/git:pr` — create a PR
7. `/docs` — update repo docs if needed

---

## 4. Multi-Repo Orchestration

When features span multiple repositories, you need coordination across agents.
A new API endpoint may require backend changes, SDK updates, frontend
integration, and infrastructure provisioning. Agents working in isolation
produce locally correct but globally inconsistent changes.

The [Multi-Repo Orchestration Guide](https://github.com/BenWeekes/ai-dev/blob/main/multi-repo-orchestration.md)
describes a multi-agent architecture for this:

| Tier   | Agent         | Scope                          | Writes Code?                    |
| ------ | ------------- | ------------------------------ | ------------------------------- |
| **T0** | System Agent  | Cross-repo orchestration       | No — plans and coordinates only |
| **T1** | Repo Agent    | Single repository              | Yes — sole writer for its repo  |
| **T2** | Sub-Agent     | Single task within a repo      | Yes — delegated by Repo Agent   |
| **T3** | E2E Validator | Integrated system (browser/UI) | No — tests and validates only   |

The guide covers the full epic lifecycle — discovery, planning, interface
agreement, parallel implementation, integration testing, and E2E validation —
with human review gates at each stage.

**Prerequisite:** Repos must have at least L0 and L1 docs (see
[section 1](#1-ai-documentation-standard)) for the System Agent to read.
