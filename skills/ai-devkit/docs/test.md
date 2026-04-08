---
name: test
description: Verify that generated progressive disclosure docs meet the standard. Use when the user wants to check doc quality or validate documentation.
---

# test

Verify that the progressive disclosure documentation in this repo gives AI agents the right context at the right level — L0+L1 for common tasks, L2 only when depth is needed.

## Prerequisites

Check that these exist before running tests:

- `docs/ai/L0_repo_card.md`
- `docs/ai/L1/` directory with at least files 01-08
- `docs/ai/L1/deep_dives/_index.md`

If any are missing, stop and tell the user to run `generate` first.

## Workflow

### 1. Structural checks

Run these quick validations first — fix any failures before proceeding to the expensive sub-agent tests:

- **L0:** exists and is under 50 lines
- **L1 files:** all 8 exist (01_setup through 08_security), each is 80-200 lines
- **L1 format:** each starts with a one-line purpose statement, each ends with `## Related Deep Dives`
- **L1 total:** combined line count is under 1,600
- **L2 index:** `deep_dives/_index.md` exists and lists all L2 files
- **L2 format:** each starts with `> **When to Read This:** ...`
- **Cross-references:** all relative links between L0 → L1 → L2 resolve to existing files
- **AGENTS.md:** exists at repo root with loading instructions
- **CLAUDE.md:** exists at repo root and references @AGENTS.md

If structural checks fail, report all failures and stop. Structural issues must be fixed before content testing.

### 2. Generate test questions

Create questions in these categories by reading the repo's actual codebase and recent git history:

**Setup & Build** — can an agent set up and build this project from docs alone?

- How do I install dependencies and build this project?
- What environment variables are required?
- What's the minimum system requirement?

**Test & Run** — can an agent run and test the project?

- How do I run the test suite?
- How do I start the project locally?

**Conventions** — does the agent know the project's patterns?

- What naming conventions does this project use?
- How should I handle errors in this codebase?

**Development** — can an agent implement a feature?

- Read `git log --oneline --since="3 months ago"` to find 2-3 recent features
- For each, write a question like: "How would I add a similar feature to [area]?"
- These should require understanding architecture, not just setup

**Deep Dive** — questions that should require L2

- Pick a complex subsystem and ask about its internals
- Ask about an edge case or non-obvious behavior
- These should only be answerable with L2 detail

If `$ARGUMENTS` specifies a focus area, weight questions toward that area.

### 3. Run tests

For each question, launch a fresh sub-agent with access to the repo:

1. Give the sub-agent only the question — do NOT tell it which docs to read
2. Let it navigate naturally (starting from AGENTS.md or L0)
3. Record: the answer, which files were read, whether the answer was correct
4. Classify: L0+L1 sufficient, or L2 needed

Expect each sub-agent to take 30-60 seconds. Total test run: 5-10 minutes for ~10 questions.

### 4. Analyze results

For each test, determine:

| Result                                              | Meaning              | Action                                  |
| --------------------------------------------------- | -------------------- | --------------------------------------- |
| Correct answer, right level loaded                  | PD docs working      | None                                    |
| Correct answer, L2 loaded unnecessarily             | L1 summary too thin  | Expand the relevant L1 section          |
| Wrong/incomplete answer, L2 exists but wasn't found | Bad cross-references | Fix `## Related Deep Dives` links in L1 |
| Wrong/incomplete answer, no L2 exists               | Missing deep dive    | Create the L2 file                      |
| Agent couldn't find answer at all                   | Missing L1 coverage  | Add to the relevant L1 file             |

### 5. Write results

Save results to `docs/ai/test-results.md`:

```markdown
# PD Documentation Test Results

Tested: [date]
Agent: [which agent/model]
Repo: [repo name]

## Summary

- Total questions: N
- Passed: N (correct answer, right level)
- L1 gaps: N (needed L2 but L1 should have sufficed)
- L2 gaps: N (needed L2 that doesn't exist)
- Cross-ref issues: N (L2 exists but wasn't found)

## Results

### Setup & Build

| #   | Question        | Answer Correct? | Files Read   | Level Loaded | Result |
| --- | --------------- | --------------- | ------------ | ------------ | ------ |
| 1   | How do I build? | Yes             | L0, 01_setup | L0+L1        | Pass   |

### Test & Run

...

### Conventions

...

### Development

...

### Deep Dive

...

## Recommended Fixes

- [ ] Expand 01_setup.md section on [topic] (questions N, N failed)
- [ ] Add deep dive: [topic].md (question N needed detail not in L1)
- [ ] Fix Related Deep Dives link in 02_architecture.md (question N)
```

### 6. Fix and retest

If any tests failed, fix the docs and rerun the failing questions only. Update test-results.md with the retest.
