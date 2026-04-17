---
name: kb-tasks-list
description: Use when listing, searching, or filtering tasks in a Kanbanned project, when you need to find tasks by status, priority, type, assignee, or tag, or when you need to understand what tasks exist before starting work.
---

# kb-tasks-list

## List all tasks in a project

```sh
kb tasks                          # uses $KB_PROJECT
kb tasks <PROJECT_ID>             # explicit project
```

Output is a table with columns: ID (short), status emoji, priority dot, title, tags, assignee.

## Filters

Append filters as `key=value` pairs after the project ID. Multiple filters are ANDed.

```sh
kb tasks status=todo
kb tasks status=in_progress,todo          # comma-separate for OR within a field
kb tasks priority=urgent,high
kb tasks kind=master                      # only master tasks
kb tasks kind=subtask                     # only subtasks
kb tasks kind=standalone                  # tasks with no parent and no children
kb tasks assignee=me                      # assigned to you
kb tasks assignee=none                    # unassigned
kb tasks assignee=<USER_ID>
kb tasks tag=backend                      # has this tag
kb tasks parent=<TASK_ID>                 # direct subtasks of a master
kb tasks parent=none                      # tasks with no parent
kb tasks parent=any                       # tasks that have a parent (all subtasks)
```

Combine filters:

```sh
kb tasks status=todo priority=urgent kind=standalone
kb tasks assignee=me status=in_progress
```

## Task kinds — critical concept

Every task is exactly one of three kinds. `kind` is not a stored field — it is computed from `parent_task_id` and whether the task has children:

| Kind | `parent_task_id` | Has children? | Meaning |
|---|---|---|---|
| `standalone` | NULL | No | Independent task, lives alone on the board |
| `master` | NULL | Yes | Parent task that owns one or more subtasks |
| `subtask` | UUID of master | — | Child task nested under a master |

**Hierarchy rules:**
- Max 2 levels: master → subtask. Subtasks cannot have their own subtasks.
- A task that starts standalone becomes a master the moment it gets its first subtask.
- A master that loses all subtasks reverts to standalone.

## List subtasks of a specific master

```sh
kb subtasks <MASTER_TASK_ID>
```

Returns only direct children of that master, in board order.

## Get a single task

```sh
kb get <TASK_ID>
```

Returns the raw JSON for one task — all fields. Use this before editing, or when you need fields not shown in the table view (e.g. `complete_after_id`, `external_ref`, full `description`).

## Status reference

| Status | Meaning |
|---|---|
| `backlog` | Not yet prioritized |
| `todo` | Prioritized, ready to start |
| `in_progress` | Actively being worked on |
| `completed` | Finished successfully |
| `failed` | Could not be completed |

## Gotchas

- `kind` filtering is computed client-side by the API — it is not a database column. Filtering by `kind=master` fetches all tasks and filters in memory; large projects may be slower.
- Comma-separated values within a filter field mean OR (`status=todo,in_progress` = todo OR in_progress).
- Tags are case-sensitive. `tag=Backend` ≠ `tag=backend`.
- `kb tasks` with no filters shows ALL tasks in all statuses — including completed and failed. Filter by status if you want only active work.
- Subtasks are NOT shown in the default board view when their master is collapsed. Use `kb subtasks <MASTER_ID>` to always see them.
