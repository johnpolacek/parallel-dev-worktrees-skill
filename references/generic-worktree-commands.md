# Generic Worktree Commands

Use this only when a repo does not provide its own worktree scripts. Prefer project scripts whenever they exist.

## Create

Use the raw branch name for git commands, such as `feature/my-task`, and a lowercase kebab-case `branch-slug` for paths, hostnames, profiles, and local state identifiers.

Concrete example:

```bash
branch="feature/login-rate-limit"
slug="login-rate-limit"
git worktree add ../myapp.worktrees/$slug -b "$branch" origin/main
```

Do not use the raw branch name where a path or hostname-safe slug is required.

```bash
git status --short --branch
git remote show origin
git fetch origin --prune
git worktree add ../<repo>.worktrees/<branch-slug> -b feature/<branch-name> origin/<default-branch>
```

Use the default branch reported by `git remote show origin`. If the repo is offline or has no remote, confirm the current branch is the integration branch before creating the worktree from the current checkout.

```bash
git branch --show-current
git status --short --branch
git worktree add ../<repo>.worktrees/<branch-slug> -b feature/<branch-name>
```

After creation:

- copy required local env files carefully
- install dependencies if needed
- create isolated DB/cache/storage identifiers
- add a worktree-specific Portless name in env or package config

## Doctor Checklist

When no `wt:doctor` script exists, inspect before creating, running, finishing, or cleaning worktrees.

```bash
git status --short --branch
git worktree list --porcelain
if command -v portless >/dev/null 2>&1; then
  portless list
else
  echo "portless unavailable"
fi
```

If `portless` is unavailable, strongly recommend installing or configuring Portless before running multiple worktrees. Use numeric ports only as a temporary fallback for local-only tasks.

For each active worktree, check dirty state:

```bash
git -C <worktree-path> status --short --branch
```

Also check:

- stale or missing worktree paths reported by Git
- overlapping dirty files between active worktrees
- Portless routes that point to removed worktrees or stopped processes
- numeric dev ports only if the repo uses them as a temporary fallback
- env files and local state identifiers such as database names, cache prefixes, storage prefixes, and `WORKTREE_SLUG`
- runtime guard output before bypassing or stopping any process

## Run With Portless

Prefer a package script. If none exists:

```bash
portless run --name <branch-slug>.<project> <dev-command>
```

For main:

```bash
portless run --name <project> <dev-command>
```

Do not hand off fixed numeric ports when a named Portless route is available.

## Finish Manually

Only use this if there is no project finish script:

If the integration branch has no upstream or the repo is offline, skip `git pull --ff-only` only after confirming the integration checkout is the intended base.

Before merging, check active worktrees for overlapping changed files. Run this only after confirming the feature worktree and integration checkout are clean; it is intended to find overlap with other active dirty worktrees.

```bash
feature_files="$(mktemp)"
dirty_files="$(mktemp)"
trap 'rm -f "$feature_files" "$dirty_files"' EXIT

git diff --name-only <integration-branch>...<feature-branch> | sort -u > "$feature_files"
git worktree list --porcelain | awk '/^worktree / {print substr($0, 10)}' |
  while read -r path; do
    git -C "$path" diff --name-only
    git -C "$path" diff --name-only --cached
  done | sort -u > "$dirty_files"
comm -12 "$feature_files" "$dirty_files"
```

If the final command prints files, inspect the overlap before merging.

```bash
# In feature worktree
git status --short --branch
git add <intended-files>
git commit -m "<message>"

# In integration checkout
git status --short --branch
git pull --ff-only
git merge --no-ff feature/<branch-name>
git worktree remove ../<repo>.worktrees/<branch-slug>
git branch -d feature/<branch-name>
git worktree prune
```

## State Isolation Checklist

- SQLite file unique to worktree.
- Postgres database/schema unique to worktree.
- Docker Compose `COMPOSE_PROJECT_NAME` unique to worktree.
- Redis/cache key prefix or container unique to worktree.
- Object storage prefix unique to worktree.
- OAuth/webhook URLs use the worktree Portless URL.
- Browser automation profile/session unique to worktree.
