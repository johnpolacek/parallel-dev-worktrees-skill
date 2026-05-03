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

To integrate the workflow into an existing project, install the skill project-locally and add repo-specific `wt:*` scripts such as `wt:doctor`, `wt:create`, `wt:list`, `wt:finish`, and `wt:clean` / `wt:prune`. Document the project's worktree path convention, Portless URL pattern, state isolation rules, and finish/cleanup commands in `AGENTS.md`.

## Works Best With

- Project-specific scripts or documented conventions for isolated databases, caches, browser profiles, and backend state.

## Release

Before publishing, run through `references/release-checklist.md`.

## License

MIT
