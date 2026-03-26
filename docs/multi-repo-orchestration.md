# Multi-Repo Orchestration

## Coordination Across Repositories with AI Agents

> **Prerequisite:** Repos must have L0/L1 docs per the [Progressive Disclosure Documentation Standard](progressive-disclosure-standard.md). Set those up first.

> **For single-repo practices, start with the [README](../README.md).** This document covers multi-repo coordination.

This is forward-thinking guidance for when your AI-assisted development reaches the point where features span multiple repositories and you need agents working across them. The practices here are suggestions — adapt them as tooling matures.

---

## Table of Contents

- [1. Summary](#1-summary)
- [2. Architecture](#2-architecture)
- [3. The System Card](#3-the-system-card)
- [4. Epic Lifecycle](#4-epic-lifecycle)
- [5. Testing and Completion](#5-testing-and-completion)
- [6. Agent Transport Layer](#6-agent-transport-layer)
- [7. Open Questions](#7-open-questions)

---

## 1. Summary

When a feature spans multiple repositories, AI coding agents working in isolation produce locally correct but globally inconsistent changes. An agent modifying the API may rename a field that the SDK agent doesn't know about.

This guide describes a coordination model: a System Agent plans cross-repo work, Repo Agents implement within their own boundaries, and human review gates keep things aligned.

### Agent Tiers

| Tier   | Agent         | Scope                          | Writes Code?                    |
| ------ | ------------- | ------------------------------ | ------------------------------- |
| **T0** | System Agent  | Cross-repo orchestration       | No — plans and coordinates only |
| **T1** | Repo Agent    | Single repository              | Yes — sole writer for its repo  |
| **T2** | Sub-Agent     | Single task within a repo      | Yes — delegated by Repo Agent   |
| **T3** | E2E Validator | Integrated system (browser/UI) | No — tests and validates only   |

### Design Principles

- **Repo sovereignty** — each repo has exactly one Repo Agent. No external agent writes to a repo it doesn't own.
- **Interface-driven coordination** — cross-repo dependencies are expressed as contracts agreed before implementation.
- **Human-in-the-loop** — review gates at plan, contract, and integration phases.

---

## 2. Architecture

```
    ┌─────────────────────────────────────────────────────────────┐
    │                      SYSTEM AGENT (T0)                      │
    │         Reads System Card · Plans epics · Coordinates       │
    │              Never writes code · Human review gates          │
    └──────────┬──────────────┬──────────────┬────────────────────┘
               │              │              │
          Task Spec      Task Spec      Task Spec
               │              │              │
    ┌──────────▼──────┐ ┌─────▼────────┐ ┌──▼───────────────┐
    │  REPO AGENT (T1) │ │ REPO AGENT   │ │  REPO AGENT      │
    │  api-service     │ │ sdk-library  │ │  frontend-app    │
    │  Sole writer     │ │ Sole writer  │ │  Sole writer     │
    └─────────────────┘ └──────────────┘ └──────────────────┘
```

The System Agent's power is coordination, not implementation. Separating it from Repo Agents prevents a class of bugs where an orchestrator makes locally-reasonable but globally-inconsistent changes.

This architecture depends on the [Progressive Disclosure Documentation Standard](progressive-disclosure-standard.md): the System Agent reads L0 Identity Blocks to build its repo registry, and L1 `06_interfaces.md` files to understand cross-repo contracts.

---

## 3. The System Card

The System Card is the system-level equivalent of a repo's L0 Repo Card — which repos exist, what they do, how they connect.

```markdown
# [System Name] — System Card

> [One-line description]

## Identity

| Field       | Value                               |
| ----------- | ----------------------------------- |
| System      | [system-name]                       |
| Description | [What the system does — 1 sentence] |
| Owner       | [Team or org]                       |
| Repos       | [Count]                             |

## Repo Registry

| Repo           | Type         | Language   | Description                 |
| -------------- | ------------ | ---------- | --------------------------- |
| `org/user-api` | api-service  | TypeScript | User management REST API    |
| `org/user-sdk` | sdk-library  | TypeScript | TypeScript SDK for User API |
| `org/web-app`  | frontend-app | TypeScript | Customer-facing web app     |

## Dependency Map

user-api (provider) → user-sdk (consumer) → web-app (consumer)

## Shared Conventions

| Convention     | Value                                  |
| -------------- | -------------------------------------- |
| API versioning | URL path prefix (`/v1/`)               |
| Auth mechanism | JWT                                    |
| Error format   | `{ error: { code, message } }`         |
```

The repo registry can be populated by scraping L0 Identity Blocks from each repo's PD docs. Shared conventions are maintained manually.

---

## 4. Epic Lifecycle

An "epic" is a cross-repo feature requiring coordinated changes across two or more repositories.

### Phases

| Phase | Name                | Who                        | What Happens                                                          |
| ----- | ------------------- | -------------------------- | --------------------------------------------------------------------- |
| 1     | Discovery           | System Agent               | Load System Card, identify affected repos, read their L0/L1 docs     |
| 2     | Planning            | System Agent               | Produce an Epic Plan: per-repo tasks, execution order, interfaces     |
| 3     | Interface Agreement | System Agent + Repo Agents | Review and agree on interface contracts before code is written        |
| 4     | Implementation      | Repo Agents (parallel)     | Each agent implements its task using TDD, constrained by contracts    |
| 5     | Integration         | System Agent + Repo Agents | Cross-repo contract tests, integration verification                   |
| 6     | E2E Validation      | E2E Validator              | Test the integrated system through browser/UI                         |
| 7     | Deploy              | Human                      | Production deployment per existing team process                       |

### Review Gates

| Gate               | When          | What the Human Reviews                                      |
| ------------------ | ------------- | ----------------------------------------------------------- |
| Plan Review        | After Phase 2 | Epic Plan — affected repos, task breakdown, execution order |
| Contract Review    | After Phase 3 | Interface contracts between repos                           |
| Code Review        | After Phase 4 | Pull requests in each repo                                  |
| Integration Review | After Phase 5 | Cross-repo test results, contract alignment                 |
| E2E Review         | After Phase 6 | E2E Validator test report                                   |

Front-load reviews — a wrong interface contract discovered during implementation wastes work in multiple repos. Teams can relax gates as they build trust.

### Sovereignty in Practice

The System Agent specifies _what_ needs to happen. The Repo Agent decides _how_.

| System Agent Says                                          | Repo Agent Decides                                        |
| ---------------------------------------------------------- | --------------------------------------------------------- |
| "Add a `/users` endpoint returning `{ id, name, email }`"  | Route structure, controller design, model layer           |
| "Expose a `getUser(id)` method in the SDK"                 | Method signature, error handling, internal implementation |

### Cross-Repo Code Review

When one agent's changes affect another repo's interface, the affected Repo Agent reviews the change from the consumer's perspective. The System Agent passes the relevant diff, and the reviewer reports back: **compatible**, **update needed**, or **conflict**.

This complements contract testing — contract tests verify mechanically, cross-repo review adds judgment.

---

## 5. Testing and Completion

### Test Layers

| Layer           | Scope                             | Who                    | When              |
| --------------- | --------------------------------- | ---------------------- | ----------------- |
| Unit            | Single function/module            | Repo Agent             | Implementation    |
| Integration     | Single repo, external deps mocked | Repo Agent             | Implementation    |
| Contract        | Interface between two repos       | Both Repo Agents       | Integration phase |
| E2E (automated) | Running system, scripted flows    | Repo Agent (test code) | Integration phase |
| E2E (validated) | Running system, exploratory       | E2E Validator          | Validation phase  |

### Completion Rule

A Repo Agent should not report a task as complete if any test is failing or skipped. The System Agent should not advance the epic to the next phase while any repo has failing tests.

### Contract Testing

The provider and consumer each write tests against the same contract:

- **Provider test:** "My endpoint returns this shape"
- **Consumer test:** "When I call the endpoint, I expect this shape"

If both pass, the contract is upheld. If either fails, fix before merging.

---

## 6. Agent Transport Layer

Sections 2-5 describe what agents say to each other. This section covers how they say it.

### The Problem

AI coding agents today are interactive sessions — a human types, the agent responds. They don't listen for incoming requests or push results to other agents. For agents on different machines, you need a network transport.

### What's Needed

The System Agent is just an AI agent session with a prompt that tells it how to coordinate. Every phase of the epic lifecycle breaks down to:

- **Prompting** — give an agent context and instructions (discovery, planning, interface agreement)
- **HTTP calls** — send a task to a Repo Agent, get a result back (implementation, cross-repo review)
- **Shell commands** — run tests (integration, E2E validation)

No special orchestration framework needs to be built. You write the System Agent prompt, point it at a transport layer, and run it. The transport layer already exists.

### Existing Projects

| Project | Agents | Protocol | Multi-Agent | Stars | Repo |
| --- | --- | --- | --- | --- | --- |
| **coder/agentapi** | 12+ (Claude, Goose, Aider, Gemini, Copilot, etc.) | REST + SSE | Multiple instances | ~1,300 | [coder/agentapi](https://github.com/coder/agentapi) |
| **claude-code-by-agents** | Claude Code | REST | First-class orchestration | ~800 | [baryhuang/claude-code-by-agents](https://github.com/baryhuang/claude-code-by-agents) |
| **claude-a2a** | Claude Code | Google A2A + MCP | Config-based + MCP client | Small | [jcwatson11/claude-a2a](https://github.com/jcwatson11/claude-a2a) |
| **CLIProxyAPI** | Multiple CLIs | OpenAI-compat | Load balancing only | ~20,000 | [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) |
| **claude-code-openai-wrapper** | Claude Code | OpenAI + Anthropic | None | ~460 | [RichardAtCT/claude-code-openai-wrapper](https://github.com/RichardAtCT/claude-code-openai-wrapper) |
| **claude-code-api** | Claude Code | OpenAI-compat | None | ~280 | [codingworkflow/claude-code-api](https://github.com/codingworkflow/claude-code-api) |
| **claude-code-api-rs** | Claude Code | OpenAI + WebSocket | None | ~140 | [ZhangHanDong/claude-code-api-rs](https://github.com/ZhangHanDong/claude-code-api-rs) |

### Which to Use

**Same-machine PoC:** `coder/agentapi` — install the binary, start one instance per repo on different ports, call their REST APIs. Supports 12+ agents, built-in `/chat` UI for debugging.

**Claude-only multi-machine:** `claude-a2a` — auth, budgets, session continuity, MCP client for agent-to-agent calls. A2A protocol provides agent discovery via Agent Cards.

**Multi-agent orchestration out of the box:** `claude-code-by-agents` — built-in task decomposition and dependency management, closest to the System Agent concept in this guide.

---

## 7. Open Questions

These are suggestions for teams reaching this stage — not blockers.

**System Card location** — Dedicated orchestration repo, auto-generated from L0 scraping, or a hybrid? Start with a manually maintained markdown file; automate the repo registry when it becomes tedious.

**Epic scope** — How many repos per epic? 3-5 keeps coordination manageable. Beyond that, consider splitting into sub-epics.

**Contract versioning** — Can a contract change mid-epic? Start with amendments and re-approval. Move toward semantic versioning for production.

**Agent state and resumability** — If a System Agent session times out mid-epic, file-based state (plans, contracts, partial results in git branches) enables any new session to pick up where the last one left off.

**Scaling** — At 50+ repos, a flat System Card may need to become hierarchical — a "System of Systems" with sub-system cards.
