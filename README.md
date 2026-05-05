# Parallel Dev Worktrees Skill

A skill for AI coding agents working safely in parallel feature branches using git worktrees, Portless named local URLs, and isolated local state.

## Install

Install globally or from your project directory:

```bash
npx skills add johnpolacek/parallel-dev-worktrees-skill
```

This skill uses Portless for branch-specific `.localhost` URLs. We recommend also installing the official Portless skill:

```bash
npx skills add https://github.com/vercel-labs/portless --skill portless
```

Installing the skill teaches the agent the workflow. It does not automatically
modify every project. To add repo-local scripts, Portless naming, and project
documentation, run an explicit project initialization prompt from the target
repo.

## Requirements

- Git worktrees.
- Portless for named local dev URLs. Numeric ports are only a temporary fallback for local-only tasks.

## What It Helps With

- Creating one worktree per feature branch.
- Preferring repo-local worktree scripts when available.
- Running worktrees through stable Portless URLs.
- Avoiding fixed localhost ports in handoffs and browser tests.
- Protecting dirty worktrees from accidental cleanup.
- Finishing feature branches through safe merge and cleanup checks.
- Keeping mutable state isolated across worktrees.

## When To Use

Use this skill when worktrees are mentioned, when juggling multiple local feature branches, or when asking an AI coding agent to create, inspect, run, hand off, finish, or clean up a feature worktree.

Example prompts:

```text
Use $parallel-dev-worktrees to create a safe feature worktree for this task.
Use $parallel-dev-worktrees to inspect active worktrees before I merge.
Use $parallel-dev-worktrees to finish and clean up this feature branch.
```

More examples are in `examples/prompts.md`.

## What The Agent Does

The agent reads repo-local instructions first, prefers existing `wt:*` scripts, checks dirty worktrees, uses Portless URLs for handoffs, verifies env/state isolation, and finishes only from clean checkouts. If no project workflow exists, it falls back to `references/generic-worktree-commands.md` and recommends adding project scripts later.

## Use In A Project

For a project without a worktree workflow, install the skill globally or invoke it explicitly from the repo. The agent will use generic fallback commands, create a safe branch slug, check env/state isolation, and recommend project scripts when useful.

### Initialize A Repo

From the repo you want to configure, ask the agent:

```text
Use $parallel-dev-worktrees to initialize this repo for parallel worktrees. Add repo-local wt:* scripts, configure Portless named URLs, document the workflow in AGENTS.md, and add checks for worktree-local env/state isolation.
```

The agent should inspect the project before editing. A good setup usually adds:

- `wt:doctor`: check git status, active worktrees, Portless availability, and env/state isolation.
- `wt:create <branch>`: create `../<repo>.worktrees/<branch-slug>` from the integration branch.
- `wt:list`: show active worktrees, branches, URLs, and dirty status.
- `wt:finish <branch>`: verify clean checkouts, pull/merge safely, remove the worktree, and prune stale metadata.
- `wt:clean` / `wt:prune`: remove only safe stale worktree metadata, routes, and generated local state.
- `AGENTS.md` notes for the worktree path convention, Portless URL pattern, state isolation rules, and finish/cleanup commands.

If the project uses databases, caches, queues, object storage, webhooks, OAuth callbacks, or local SQLite files, the setup should also document or enforce worktree-specific state identifiers before multiple worktrees run at the same time.

## Works Best With

- Project-specific scripts or documented conventions for isolated databases, caches, browser profiles, and backend state.

## Release

Before publishing, run through `references/release-checklist.md`.

## License

MIT
