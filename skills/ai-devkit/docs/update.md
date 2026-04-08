---
name: update
description: Update progressive disclosure docs after code changes. Use when the user wants to update existing docs, refresh docs, or sync docs with recent code changes.
---

# update

Update the existing progressive disclosure documentation to reflect recent code changes. This is for repos that already have `docs/ai/` — use `generate` to create from scratch.

## Prerequisites

Check that `docs/ai/L0_repo_card.md` exists. If it does not, tell the user to run `generate` first and stop.

## Rules

- Only modify docs that are affected by the changes — do not regenerate everything
- Preserve existing content that is still accurate
- Follow the same format and density targets as the original docs
- Update `last_reviewed` in L0 when done

## Workflow

### 1. Identify what changed

Determine the scope of changes to document:

- If `$ARGUMENTS` specifies a scope (e.g., "auth module", "v2 API"), focus there
- Otherwise, compare against the last `last_reviewed` date in `docs/ai/L0_repo_card.md`:
  ```
  git log --oneline --since="[last_reviewed date]"
  ```
- If no date or no L0 exists, fall back to the last 30 days:
  ```
  git log --oneline --since="30 days ago"
  ```

Group changes into: new modules, changed interfaces, new workflows, new gotchas, deprecated features, dependency changes, removed code.

### 2. Read current docs

Read the existing PD docs to understand what's already documented:

1. `docs/ai/L0_repo_card.md`
2. All 8 L1 files in `docs/ai/L1/`
3. `docs/ai/L1/deep_dives/_index.md`

Do NOT read L2 files yet — only read them if a change directly affects a deep dive topic.

### 3. Map changes to docs

For each change identified in step 1, determine which doc files need updating:

| Change Type                          | Update                                                       |
| ------------------------------------ | ------------------------------------------------------------ |
| New module or package                | `03_code_map.md` — add to directory tree and module map      |
| New API endpoint or event            | `06_interfaces.md` — add contract                            |
| Changed API contract                 | `06_interfaces.md` — update contract                         |
| Deprecated API or feature            | `06_interfaces.md` — mark deprecated, add migration note     |
| New workflow (deploy, migrate, etc.) | `05_workflows.md` — add step-by-step                         |
| New environment variable or config   | `01_setup.md` — add to env table                             |
| Architecture change                  | `02_architecture.md` — update component diagram or data flow |
| New convention or pattern            | `04_conventions.md` — add pattern                            |
| New gotcha or incident lesson        | `07_gotchas.md` — add entry                                  |
| Security model change                | `08_security.md` — update trust boundaries                   |
| Dependency added or upgraded         | `01_setup.md` — update dependency list                       |
| Tech stack change                    | `02_architecture.md` — update stack section                  |
| Module removed                       | `03_code_map.md` — remove entry                              |

### 4. Read the changed code

For each affected area, read the actual source files to understand the change. Don't rely on commit messages alone — read the code to get accurate details.

### 5. Update docs

Apply the changes identified in step 3. For each file:

- Add new content in the same style as existing content
- Remove content that is no longer accurate
- Keep each L1 file within 80-200 lines — if a section is growing too large, move detail to a new L2 deep dive

If a change warrants a new L2 deep dive:

1. Create the file in `docs/ai/L1/deep_dives/`
2. Add it to `_index.md`
3. Add a link in the relevant L1 file's `## Related Deep Dives` section

### 6. Update L0

Update `docs/ai/L0_repo_card.md`:

- Set `last_reviewed` to today's date
- Update the L1 index table if any file purposes changed
- Update identity block if repo type, language, or deploy target changed

### 7. Verify

- All cross-references still resolve
- Each modified L1 file is still 80-200 lines
- Any new L2 files start with `> **When to Read This:** ...`
- No stale information remains in modified sections

### 8. Summary

List what was updated:

```
## Docs Updated

- `01_setup.md` — added REDIS_URL env var
- `03_code_map.md` — added auth/ module
- `06_interfaces.md` — updated POST /v1/sessions response shape
- `deep_dives/auth_flow.md` — new deep dive for OAuth2 implementation
- `L0_repo_card.md` — updated last_reviewed
```
