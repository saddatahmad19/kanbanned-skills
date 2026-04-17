---
name: kb-task-start
description: Use when picking up a Kanbanned task to work on — claiming it, marking it in progress, understanding what it requires, and keeping its status accurate as work proceeds through to completion or failure.
---

# kb-task-start

Taking ownership of a task is a three-part responsibility: claim it so others know it's taken, understand it fully before touching any code, and keep its status honest as you work.

## Step 1 — Claim the task

```sh
kb claim <TASK_ID>   # sets status → in_progress, records started_at
# or equivalently:
kb start <TASK_ID>   # same as claim
```

Both commands set `status = in_progress` and record `started_at`. Do this **before** beginning work, not after.

Also assign it to yourself if unassigned:

```sh
kb assign <TASK_ID> me
```

## Step 2 — Understand the task completely

Before writing a single line of code, read the full task:

```sh
kb get <TASK_ID> | jq .
```

Work through these questions in order:

**What kind of task is this?**
```sh
# Is it a subtask? (has a parent)
kb get <TASK_ID> | jq .parent_task_id

# Is it a master? (has children)
kb tasks parent=<TASK_ID>
```

If it's a **subtask**, read the master first for full context:
```sh
kb get <PARENT_TASK_ID> | jq '{title, description, status}'
kb subtasks <PARENT_TASK_ID>   # see sibling subtasks, understand scope
```

If it's a **master**, understand that your work affects multiple subtasks:
```sh
kb subtasks <TASK_ID>          # what subtasks exist?
kb tasks parent=<TASK_ID> status=backlog,todo,in_progress   # which are still open?
```

**Does it have a prerequisite?**
```sh
kb get <TASK_ID> | jq .complete_after_id
```
If non-null, check whether the prerequisite is already completed before proceeding:
```sh
kb get <PREREQ_ID> | jq '{title, status}'
```
If the prerequisite is not `completed`, note it. You can still work, but be aware of the dependency — the system will warn you at completion time.

**Read the description fully.** It may contain acceptance criteria, spec links, branch names, or explicit instructions for you as an AI agent.

## Step 3 — Do the work, then update status

When the work is done:

```sh
kb done <TASK_ID>     # status → completed, records completed_at
kb fail <TASK_ID>     # status → failed, records failed_at (use when blocked/impossible)
```

If you need to step back without completing:
```sh
kb reopen <TASK_ID>   # status → todo
kb backlog <TASK_ID>  # status → backlog
```

### Prerequisite warning on completion

If `complete_after_id` points to an unfinished task, `kb done` succeeds but returns a warning:

```
X-Kanbanned-Warning: Prerequisite "<title>" isn't completed yet
```

This is **not** an error — the task is marked complete. But note it so a human can review the ordering.

## Status lifecycle reference

```
backlog → todo → in_progress → completed
                             → failed
         todo ← in_progress  (reopen)
backlog ← todo               (backlog)
```

Every status transition is recorded in the task's status history (visible in the web app), including the source (`cli`) and timestamp.

## Subtask completion and master progress

Completing a subtask does **not** automatically complete the master. Check sibling progress after each subtask:

```sh
kb tasks parent=<MASTER_ID> status=backlog,todo,in_progress
```

When all subtasks are done, complete the master explicitly:

```sh
kb done <MASTER_ID>
```

## Gotchas

- `claim` and `start` are identical — either works. Use whichever reads more naturally in context.
- Always claim **before** starting work. If someone else claims it first, coordinate rather than double-claiming.
- `kb done` on a master with open subtasks succeeds with no warning — the system trusts you. Always check subtask status manually before marking a master complete.
- `failed_at` and `completed_at` are set by the server on the corresponding transition. You cannot set them independently.
- Reopening a task (`kb reopen`) clears `completed_at` / `failed_at` — the task returns to `todo` cleanly.
- If a task description says "do not mark complete until PR is merged" — follow that instruction. The CLI will let you mark it done regardless.
