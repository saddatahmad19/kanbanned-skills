---
name: kb-task-hierarchy
description: Use when working with master tasks and subtasks in Kanbanned — attaching or detaching tasks from a parent, understanding the two-level hierarchy rules, or restructuring task relationships.
---

# kb-task-hierarchy

Kanbanned supports exactly **two levels** of task hierarchy: master tasks and their subtasks. There is no deeper nesting.

## Hierarchy rules (enforced by the database)

| Rule | Detail |
|---|---|
| Max depth | 2 levels only. A subtask cannot have its own subtasks. |
| Subtask → subtask blocked | If a task already has children, it cannot be made a subtask. |
| Master → subtask blocked | If a task already is a master (has children), it cannot become a subtask. |
| No self-parent | A task cannot be its own parent. |
| Sibling prereq blocked | A subtask's `complete_after_id` cannot point to its own parent or a sibling subtask. |

## Making a task a subtask (attach)

```sh
kb attach <TASK_ID> to <MASTER_TASK_ID>
```

- Sets `parent_task_id = <MASTER_TASK_ID>` on the task
- `<MASTER_TASK_ID>` must not itself be a subtask (no nesting)
- `<TASK_ID>` must not already have children of its own

Example: promoting a standalone task into a subtask of a master:
```sh
kb attach task-uuid-456 to task-uuid-123
```

## Promoting a subtask back to standalone (detach)

```sh
kb detach <TASK_ID>
```

- Clears `parent_task_id` (sets to NULL)
- Task becomes standalone again
- Does not affect the former master's other subtasks

## Listing subtasks of a master

```sh
kb subtasks <MASTER_TASK_ID>
```

Returns all direct children in board order. There is no recursive listing because nesting only goes one level deep.

## Filtering by hierarchy

```sh
kb tasks kind=master       # all tasks that have at least one subtask
kb tasks kind=subtask      # all tasks that have a parent
kb tasks kind=standalone   # all tasks with no parent and no children
kb tasks parent=<ID>       # subtasks of a specific master
kb tasks parent=none       # tasks with no parent (masters + standalones combined)
kb tasks parent=any        # tasks that have a parent (all subtasks)
```

## Understanding kind — it is computed, not stored

`kind` is not a column in the database. It is derived:
- A task is a **master** if any other task has `parent_task_id = this task's id`
- A task is a **subtask** if its own `parent_task_id` is not null
- A task is **standalone** if neither condition is true

This means kind can change silently: a standalone task becomes a master the moment you attach a subtask to it, and reverts to standalone if you detach all its subtasks.

## Checking the current state before restructuring

Before attaching or detaching, confirm current state:

```sh
# Does task-A already have children? (Would make it a master, not eligible to become subtask)
kb tasks parent=<TASK_A_ID>

# Is task-A already a subtask?
kb get <TASK_A_ID> | jq .parent_task_id

# Is task-B a valid master candidate? (must not be a subtask itself)
kb get <TASK_B_ID> | jq .parent_task_id   # must be null
```

## Typical restructuring workflow

```sh
# 1. Create a new master task
kb new "Authentication Overhaul" --status todo --priority high

# 2. Attach existing standalone tasks as subtasks
kb attach <task-id-1> to <master-id>
kb attach <task-id-2> to <master-id>

# 3. Verify
kb subtasks <master-id>
```

## Gotchas

- Attaching a task that already has children will fail — the API enforces the 2-level limit.
- `kb detach` does not delete the task — it only clears the parent relationship.
- When a subtask is detached, its `complete_after_id` is preserved. If it previously depended on a sibling, that dependency now points to an unrelated standalone task. Review after detaching.
- There is no bulk-attach command. Attach tasks one at a time.
- The web board groups subtasks visually under their master and lets users collapse/expand them. The CLI has no collapsing concept — `kb subtasks` always shows all children.
