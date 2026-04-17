---
name: kb-projects
description: Use when listing available projects in Kanbanned, when selecting which project to work in, when you need a project UUID for other kb commands, or when orienting yourself to what work exists across an organization.
---

# kb-projects

Projects in Kanbanned are the top-level containers for tasks. Each project belongs to an organization. You must have a project UUID to list or create tasks.

## List projects

```sh
kb projects
```

Returns all active projects across every organization you belong to, ordered newest first. Each row shows:

| Field | Meaning |
|---|---|
| `id` | UUID — use this in other commands as `PROJECT_ID` |
| `name` | Human-readable project name |
| `description` | Optional description |
| `is_active` | Whether the project is active |
| `created_at` | Creation timestamp |

## Set a default project

Once you know which project to work in, set `KB_PROJECT` so you don't need to pass the UUID every time:

```sh
export KB_PROJECT="<project-uuid>"
```

After this, `kb tasks` and `kb new` work without explicitly passing a project ID.

## Whoami

```sh
kb whoami
```

Shows your authenticated user details. Useful for confirming who you're acting as, especially when using `assign me` or filtering by `assignee=me`.

## Workflow: orient yourself to a new project

1. `kb projects` — find the project you want; copy its UUID
2. `export KB_PROJECT="<uuid>"` — set default
3. `kb tasks` — see all tasks in the project
4. Use `kb-tasks-list` skill to filter by status, priority, or kind

## Gotchas

- If `kb projects` returns an empty list, check that your API key belongs to a user who is a member of at least one organization.
- Project IDs are UUIDs (e.g. `a1b2c3d4-...`). Always copy-paste; never type by hand.
- There is no `kb project create` command in the CLI — projects are created via the web app.
- `kb whoami` calls the same `/projects` endpoint — it shows user context alongside projects.
