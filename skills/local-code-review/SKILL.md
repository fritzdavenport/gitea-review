---
name: local-code-review
description: Use when user wants local code review with GitHub-like PR interface, or when reviewing branches before merge using Gitea
---

# Local Code Review

Ephemeral GitHub-like PR review via a throwaway local Gitea instance. User reviews in the web UI (line comments, suggestions, squash merge). Agent manages PRs via CLI. Nothing persists after `done`.

## Quick Reference

| Command | Description |
|---------|-------------|
| `gitea-review start <branch> [...]` | Spin up Gitea, push branches, create PRs, open browser |
| `gitea-review comments <pr#>` | Fetch review comments grouped by file:line |
| `gitea-review push <branch>` | Push fixes to existing PR |
| `gitea-review reply <pr#> <path> <line> <msg>` | Reply to a comment |
| `gitea-review merge <pr#>` | Squash merge PR |
| `gitea-review list` | List open PRs |
| `gitea-review open [pr#]` | Open in browser |
| `gitea-review status` | Check if Gitea is running |
| `gitea-review done` | Tear down Gitea, remove remote, clean up |

**AI assistants:** Use full path `~/.claude/skills/local-code-review/bin/gitea-review`.

## When to Use

- User wants to review branches with line comments before merging
- User asks for "local code review" or "local PR review"
- User wants GitHub-like review without pushing to GitHub
- User says "create a PR for review" and context is local/private work

## How It Works

Gitea runs as an ephemeral Docker container with a Caddy reverse proxy that auto-logs the user in (no password needed). The script adds a `gitea` remote alongside `origin`, pushes branches there, and creates PRs. The user reviews at `http://localhost:3000` (override with `GITEA_PORT`). When done, `gitea-review done` removes the container, the remote, and all temp state.

## Workflow

```
gitea-review start branch1 branch2   # Launch Gitea, create PRs, open browser
  [user reviews in browser, leaves line comments]
gitea-review comments 1               # Agent reads comments
  [agent makes code fixes]
gitea-review push branch1             # Push fixes
  [user resolves threads in browser]
gitea-review merge 1                  # Squash merge when approved
gitea-review done                     # Tear down
```

## Responding to Review Comments

When the user says "address review comments" or "check PR comments":

1. `gitea-review comments <pr#>` -- each comment shows `[id] reviewer: message` at `file:line` with diff context
2. Make the requested code changes
3. `gitea-review push <branch>`
4. `gitea-review reply <pr#> <path> <line> "Fixed -- <what you did>"`
5. User resolves threads in the browser UI

## Requirements

- Docker
- curl, python3, git

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `GITEA_PORT` | `3000` | Port for the Gitea web UI |
