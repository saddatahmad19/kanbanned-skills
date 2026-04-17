---
name: kb-task-create-edit
description: Use when creating new tasks in Kanbanned, editing existing task fields, managing tags, changing assignees or priority, setting prerequisite dependencies, attaching external references, or deleting tasks.
---

# kb-task-create-edit

## Create a task

```sh
kb new "Task title" [PROJECT_ID] [options]
```

`PROJECT_ID` is optional if `$KB_PROJECT` is set.

### Creation options

| Option | Values | Default |
|---|---|---|
| `desc=` | Free text description | (empty) |
| `priority=` | `low` `medium` `high` `urgent` | `medium` |
| `status=` | `backlog` `todo` `in_progress` | `backlog` |
| `parent=<TASK_ID>` | Makes this a subtask of that master | (none) |
| `after=<TASK_ID>` | Sets `complete_after_id` (prerequisite) | (none) |
| `assignee=<USER_ID\|me>` | Assigns immediately | (none) |
| `tags=tag1,tag2` | Comma-separated tags | (none) |
| `ref=<string>` | External reference (PR, ticket ID, etc.) | (none) |

```sh
# Standalone task
kb new "Fix login redirect bug" priority=high status=todo

# Subtask under a master
kb new "Write migration script" parent=<MASTER_ID> priority=medium

# With all fields
kb new "Refactor auth middleware" \
  desc="Remove deprecated session handling, see RFC-42" \
  priority=urgent \
  status=todo \
  assignee=me \
  tags=auth,backend \
  ref=PR-1234 \
  after=<PREREQ_TASK_ID>
```

## Edit a task

```sh
kb edit <TASK_ID> [options]
```

Uses the same option syntax as `new`. Only fields you specify are changed.

```sh
kb edit <TASK_ID> priority=urgent
kb edit <TASK_ID> desc="Updated acceptance criteria..."
kb edit <TASK_ID> ref=PR-9999
kb edit <TASK_ID> assignee=me
kb edit <TASK_ID> status=todo        # also valid; prefer dedicated commands
```

## Dedicated status commands (prefer these over `edit status=`)

```sh
kb claim <TASK_ID>    # → in_progress (also use: kb start)
kb done <TASK_ID>     # → completed
kb fail <TASK_ID>     # → failed
kb reopen <TASK_ID>   # → todo
kb backlog <TASK_ID>  # → backlog
```

These also set the corresponding timestamp (`started_at`, `completed_at`, `failed_at`) — `kb edit status=` does not set timestamps.

## Priority

```sh
kb priority <TASK_ID> urgent
kb priority <TASK_ID> high
kb priority <TASK_ID> medium
kb priority <TASK_ID> low
```

## Assignment

```sh
kb assign <TASK_ID> me          # assign to yourself
kb assign <TASK_ID> <USER_ID>   # assign to specific user
kb assign <TASK_ID> none        # unassign
```

## Tags

Tags are additive by default — `tag` merges with existing tags.

```sh
kb tag <TASK_ID> backend auth          # adds tags (merges)
kb untag <TASK_ID> auth                # removes a specific tag
kb edit <TASK_ID> tags=backend,api     # replaces ALL tags
```

Use `kb edit tags=...` when you want to set the exact tag list, replacing whatever was there.

## Prerequisites (complete-after)

```sh
kb link <TASK_ID> after <PREREQ_ID>   # set dependency
kb unlink <TASK_ID>                   # clear dependency
```

Restrictions:
- A task cannot depend on itself
- A subtask cannot depend on its parent or a sibling subtask

## External references

```sh
kb edit <TASK_ID> ref=PR-1234
kb edit <TASK_ID> ref=JIRA-88
kb edit <TASK_ID> ref=feat/my-branch
```

`ref` is free text — no format is enforced. Teams establish their own conventions.

## Comments

```sh
kb comment <TASK_ID> "Progress update: auth middleware extracted to separate module"
```

Comments are appended and cannot be edited or deleted via CLI.

## Delete a task

```sh
kb rm <TASK_ID>        # prompts for confirmation
kb rm <TASK_ID> -y     # skip confirmation
```

**Deletion is permanent.** Deleting a master task does not delete its subtasks — their `parent_task_id` is set to NULL (they become standalone). Delete subtasks individually if needed.

## Gotchas

- `kb tag` **merges** tags; `kb edit tags=` **replaces** tags. Easy to confuse.
- Status-changing via `kb edit status=` works but skips setting `started_at`/`completed_at`/`failed_at`. Always use the dedicated status commands for status changes.
- `kb new` requires a title as the first positional argument, quoted if it contains spaces.
- `assignee=me` resolves to your user ID from the API key's authentication context — it always means "the key owner," regardless of environment.
- There is no `kb edit tags+=` syntax for merging via edit. Use `kb tag` to add, `kb untag` to remove, or `kb edit tags=` to replace entirely.
- Description (`desc=`) does not support newlines in the CLI. Write long descriptions in the web app's task drawer.
