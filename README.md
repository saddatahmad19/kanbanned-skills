# kanbanned-skills

Agent skills for working with [Kanbanned](https://kanbanned.xyz) — a kanban project management app with a CLI and REST API.

Install these skills into any compatible AI agent (Claude Code, Cursor, GitHub Copilot, etc.) to give it the knowledge to navigate projects, read and understand tasks deeply, manage task status, and work with the master/subtask hierarchy.

## Install

```sh
npx skills add saddatahmad19/kanbanned-skills
```

To install for a specific agent only:

```sh
npx skills add saddatahmad19/kanbanned-skills -a claude-code
```

To preview available skills without installing:

```sh
npx skills add saddatahmad19/kanbanned-skills --list
```

## Skills

| Skill | Purpose |
|---|---|
| [`kb-setup`](skills/kb-setup/SKILL.md) | Install and configure the `kb` CLI, set required environment variables, verify authentication |
| [`kb-projects`](skills/kb-projects/SKILL.md) | List available projects, get project UUIDs, set a default project |
| [`kb-tasks-list`](skills/kb-tasks-list/SKILL.md) | List and filter tasks by status, priority, kind (standalone/master/subtask), assignee, and tag |
| [`kb-task-understand`](skills/kb-task-understand/SKILL.md) | Deep-read a single task: all fields, hierarchy position, prerequisite chain, description |
| [`kb-task-start`](skills/kb-task-start/SKILL.md) | Claim a task, understand it fully, and drive it through to completion or failure |
| [`kb-task-hierarchy`](skills/kb-task-hierarchy/SKILL.md) | Work with master/subtask relationships — attach, detach, understand the 2-level nesting rules |
| [`kb-task-create-edit`](skills/kb-task-create-edit/SKILL.md) | Create tasks, edit fields, manage tags, set assignees, link prerequisites, delete |

## Task hierarchy quick reference

```
standalone  — no parent, no children
master      — no parent, has subtasks (max 1 level deep)
subtask     — has a parent_task_id pointing to a master
```

Subtasks cannot be nested. A master that loses all subtasks reverts to standalone.

## Prerequisites

- `curl` or `wget`
- `jq` (optional but recommended for readable output)
- A Kanbanned account with an API key from `https://kanbanned.xyz/app/settings/api-keys`
