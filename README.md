# gitea-review

> [!CAUTION]
> I did not write this code - though was involved in it's creation process and have fully reviewed all lines.
> Make sure you review all scripts and skills prior to installation

Ephemeral local code review with a GitHub-like PR interface. Spins up a throwaway [Gitea](https://gitea.io) instance, creates PRs from your branches, and opens the browser for review. Tear it down when you're done -- nothing persists.

## Install

```bash
npx skills add fritzdavenport/gitea-review
```

Or manually: copy `bin/gitea-review` to your PATH and `skills/local-code-review/SKILL.md` to `~/.claude/skills/local-code-review/`.

## Requirements

- Docker
- curl, python3, git

## Usage

```bash
# Start a review session (launches Gitea, creates PRs, opens browser)
gitea-review start feature-branch

# Review in the browser (line comments, suggestions, etc.)

# Agent reads comments
gitea-review comments 1

# Push fixes after addressing comments
gitea-review push feature-branch

# Squash merge when approved
gitea-review merge 1

# Tear down
gitea-review done
```

## How It Works

1. `start` launches an ephemeral Gitea Docker container with a Caddy reverse proxy that auto-logs you in (no password needed)
2. Creates a `gitea` git remote, pushes your branches, and opens PRs with auto-generated descriptions
3. You review in the browser at `http://localhost:3000` with full GitHub-like UI (line comments, suggestions, squash merge button)
4. The agent reads comments via the Gitea API and pushes fixes
5. `done` removes the container, git remote, and all temp state

### Concurrent Sessions

Each repository gets its own isolated Gitea instance:
- Container name: `gitea-review-${REPO}` (allows concurrent containers)
- Port: auto-assigned from repo name hash (3000-3999 range), override with `GITEA_PORT`
- State file: per-repo in `$TMPDIR`

This prevents multiple sessions from interfering with each other when reviewing different repositories simultaneously.

### PR Descriptions

Each PR is automatically created with a thorough description that analyzes your commits and changes:

- Conventional commit type breakdown (feat, fix, test, etc.)
- File modification count and line change statistics
- Changed files list (up to 10)
- Commit history (up to 10)

This gives reviewers immediate context about what changed and why, with no manual work required.

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `GITEA_PORT` | Auto-assigned (3000-3999) | Port for the Gitea web UI |

## Commands

| Command | Description |
|---------|-------------|
| `start <branch> [...]` | Spin up Gitea, push branches, create PRs, open browser |
| `comments <pr#>` | Show review comments grouped by file:line |
| `push <branch>` | Push updated branch |
| `reply <pr#> <path> <line> <msg>` | Reply to a comment |
| `merge <pr#> [style]` | Squash merge (styles: squash, merge, rebase) |
| `list [state]` | List PRs (default: open) |
| `open [pr#]` | Open in browser |
| `status` | Check if Gitea is running |
| `done` | Tear down Gitea, remove remote, clean up |

## License

MIT
