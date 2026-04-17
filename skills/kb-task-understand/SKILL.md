---
name: kb-task-understand
description: Use when you need to deeply understand a specific Kanbanned task before working on it — reading all its fields, understanding its place in the hierarchy, tracing its prerequisite chain, and knowing what completion actually means for this task.
---

# kb-task-understand

Before touching a task, read it completely. A task contains more information than its title suggests.

## Fetch the full task

```sh
kb get <TASK_ID>
```

This returns raw JSON with every field. Pipe through `jq` for readability:

```sh
kb get <TASK_ID> | jq .
```

## Full field reference

| Field | Type | Meaning |
|---|---|---|
| `id` | UUID | Unique task identifier |
| `project_id` | UUID | Which project this belongs to |
| `organization_id` | UUID | Which org owns it |
| `title` | string | Task name |
| `description` | string \| null | Full description — read this entirely |
| `status` | enum | Current state: `backlog` `todo` `in_progress` `completed` `failed` |
| `priority` | enum | `low` `medium` `high` `urgent` |
| `assignee_id` | UUID \| null | Who is assigned (null = unassigned) |
| `created_by` | UUID \| null | Who created the task |
| `created_at` | timestamp | When created |
| `updated_at` | timestamp | Last modification |
| `started_at` | timestamp \| null | Set when status → `in_progress` |
| `completed_at` | timestamp \| null | Set when status → `completed` |
| `failed_at` | timestamp \| null | Set when status → `failed` |
| `tags` | string[] | Labels for filtering/grouping |
| `external_ref` | string \| null | External reference (e.g. `PR-1234`, `JIRA-42`) |
| `board_order` | number | Position within its status column |
| `parent_task_id` | UUID \| null | If set, this task is a subtask of that master |
| `complete_after_id` | UUID \| null | If set, this task has a prerequisite dependency |

## Determine task kind

```
parent_task_id is NULL AND no other task has parent_task_id = this id  →  standalone
parent_task_id is NULL AND some tasks have parent_task_id = this id    →  master
parent_task_id is NOT NULL                                              →  subtask
```

```sh
# Check if master: does anything point to this task as parent?
kb tasks parent=<TASK_ID>

# Check if subtask: does this task have a parent?
kb get <TASK_ID> | jq .parent_task_id

# If subtask, fetch the master:
kb get <PARENT_TASK_ID>
```

## Trace the prerequisite chain

`complete_after_id` means "this task should not be completed before the prerequisite is done." The check is a **soft warning** — it does not block completion, but it signals that the order matters.

```sh
# Does this task have a prerequisite?
kb get <TASK_ID> | jq .complete_after_id

# If so, check that prerequisite's status:
kb get <PREREQ_TASK_ID> | jq '{title, status, completed_at}'
```

If the prerequisite is not yet `completed`, completing this task will return a warning header. Decide consciously whether to proceed.

## Understand the full context of a master task

```sh
# 1. Read the master itself
kb get <MASTER_ID> | jq .

# 2. Read all subtasks
kb subtasks <MASTER_ID>

# 3. Check which subtasks are still open
kb tasks parent=<MASTER_ID> status=backlog,todo,in_progress
```

Completing a master task when subtasks are still open is allowed but may indicate incomplete work. Always check.

## Read the description

`description` is free-form markdown. It often contains:
- Acceptance criteria
- Links to specs or PRs
- Context an `external_ref` points to
- Instructions for the agent (you) on how to complete the work

**Always read the full description before starting a task.**

## Gotchas

- `started_at`, `completed_at`, `failed_at` are set automatically by the API on status transitions — you cannot set them manually via the CLI.
- `board_order` is managed by the web app's drag-and-drop. Ignore it in CLI workflows.
- A null `assignee_id` does not mean the task is available — check `status` too.
- `external_ref` is free text — conventions vary by team (`PR-123`, `#456`, `feat/branch-name`).
- There is no status history endpoint in the CLI. Status change history is visible only in the web app's task drawer.
