---
name: kb-setup
description: Use when configuring the Kanbanned CLI for the first time, when KB_API_KEY or KB_API_URL are missing, when kb commands return auth errors, or when verifying the CLI is correctly installed and authenticated.
compatibility: Requires curl (or wget) and optionally jq for pretty output. Requires a Kanbanned account with an API key.
---

# kb-setup

The `kb` CLI is a thin shell script that wraps the Kanbanned REST API. It requires three environment variables to function. Get these right before running any other skill.

## Required environment variables

| Variable | Purpose | Example |
|---|---|---|
| `KB_API_KEY` | Authenticates every request | `kb_live_abc123...` |
| `KB_API_URL` | Base URL of the API | `https://kanbanned.xyz/api` |
| `KB_PROJECT` | (Optional) Default project UUID | `xxxxxxxx-xxxx-...` |

`KB_PROJECT` is optional but highly recommended — it lets you run `kb tasks` without specifying a project ID every time.

## Installation

```sh
curl -fsSL https://kanbanned.xyz/install.sh | sh
```

The installer downloads `kb` to `~/.local/bin/kb`, makes it executable, and tells you exactly what to add to your shell profile. It does **not** modify your profile automatically.

## One-time setup

1. Generate an API key at `https://kanbanned.xyz/app/settings/api-keys`
2. Add to your shell profile (`~/.bashrc`, `~/.zshrc`, or `~/.config/fish/config.fish`):

```sh
# bash / zsh
export KB_API_KEY="kb_live_..."
export KB_API_URL="https://kanbanned.xyz/api"
export KB_PROJECT="<your-default-project-uuid>"  # optional but recommended

# fish
set -gx KB_API_KEY "kb_live_..."
set -gx KB_API_URL "https://kanbanned.xyz/api"
set -gx KB_PROJECT "<your-default-project-uuid>"
```

3. Reload: `source ~/.bashrc` (or open a new terminal)

## Verification

```sh
kb --version     # should print version number (e.g. 0.2.0)
kb whoami        # should return your user info
kb projects      # should list your organizations' projects
kb tasks         # lists tasks for $KB_PROJECT (if set)
```

## Gotchas

- `jq` is optional but strongly recommended — without it, `kb` output is raw JSON, hard to read. Install: `brew install jq` or `apt install jq`.
- `KB_API_URL` must NOT have a trailing slash. Use `https://kanbanned.xyz/api`, not `https://kanbanned.xyz/api/`.
- API keys are prefixed `kb_live_`. If your key doesn't start with that, it may be invalid or a test key.
- If `kb` is installed but not found, add `~/.local/bin` to your PATH: `export PATH="$HOME/.local/bin:$PATH"`.
- The installer prints exact `export` lines — copy them verbatim; don't paraphrase.

## Auth errors

If any command returns a 401 or "unauthorized" error:
1. Check `echo $KB_API_KEY` — must be non-empty and prefixed `kb_live_`
2. Check `echo $KB_API_URL` — must be the correct API base URL
3. Verify the key is still active at `https://kanbanned.xyz/app/settings/api-keys`
4. Re-source your profile: `source ~/.bashrc`
