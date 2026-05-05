---
name: parallel-dev-worktrees
description: Use an existing repo's parallel feature development workflow with git worktrees, Portless named local URLs, isolated local state, and safe merge/cleanup habits. Use this skill any time worktrees are mentioned or when a user is juggling multiple feature branches locally. Use when creating, using, inspecting, handing off, finishing, or cleaning feature worktrees in a repo that supports or should support parallel local development.
---

# Parallel Dev Worktrees

Use this skill for day-to-day parallel development in one repo. It is operational: create a feature worktree, run it at a named Portless URL, keep mutable state isolated, finish safely, and never damage unrelated work.

## First Look

Before creating or merging worktrees:

1. Read repo instructions such as `AGENTS.md`, `.agents/skills/*worktree*/SKILL.md`, `package.json` scripts, and existing worktree docs.
2. Prefer repo-local scripts over generic commands. Look for `wt:create`, `wt:list`, `wt:doctor`, `wt:runtime`, `wt:overlap`, `wt:finish`, `wt:clean`, and `wt:prune`. If a repo-local worktree script exists for the operation, use it instead of the generic commands below unless the script is broken or the user explicitly asks for a manual fallback.
3. Check the current checkout status. Respect dirty files and assume they belong to the user or another agent.
4. If the repo lacks a worktree workflow, use the fallback guidance in [generic-worktree-commands.md](references/generic-worktree-commands.md): create a sibling worktree from the explicit default integration branch, derive a safe branch slug, run setup/env isolation checks, use Portless for the runtime URL, and finish only from clean checkouts after overlap checks. Recommend adding project scripts later.

## Project Onboarding

Skill installation and project initialization are separate steps. Installing this skill makes the workflow available to the agent; it does not automatically modify every repo or install project-local scripts.

Portless is a required companion for the intended workflow. During project initialization, check both:

- The Portless skill is available, or the environment can run `npx skills add https://github.com/vercel-labs/portless --skill portless`.
- The `portless` CLI is available or the project has a documented way to run Portless.

If the Portless skill is missing and skill installation is available, install it as part of the initialization. If installation is not available or fails, stop before adding scripts that depend on Portless and report the exact install command. If only the CLI is missing, defer to the Portless skill for CLI setup and troubleshooting.

When the user explicitly asks to initialize, onboard, configure, or install the workflow in a repo, inspect the project and then add a repo-specific workflow when appropriate. Prefer concrete project scripts and documentation over generic advice. A minimal workflow usually includes:

- `wt:doctor`: check git status, worktree list, Portless availability, and state isolation settings.
- `wt:create <branch>`: create a sibling worktree from the default integration branch.
- `wt:list`: show active worktrees, branches, URLs, and dirty status.
- `wt:finish <branch>`: verify clean checkouts, fast-forward integration, check overlap, merge, remove the worktree, and prune stale metadata.
- `wt:clean` / `wt:prune`: remove only safe stale worktree metadata, routes, and generated local state.

For projects with shared backend risk, add or suggest a small runtime guard that reads a worktree identifier such as `WORKTREE_SLUG` and refuses to start if the app would connect to shared databases, caches, queues, object storage, or webhook endpoints without an explicit isolation setting.

Also update or propose updating `AGENTS.md` with the project's worktree path convention, Portless URL pattern, state isolation rules, and finish/cleanup commands.

If a repo has no worktree workflow and the user asked for an unrelated feature task, do not invent project-specific automation as part of that task. Use the generic fallback for the current work, then propose adding the minimal workflow later.

## Core Rules

- Use one worktree per feature branch.
- Keep `main` or the repo's default branch as the integration checkout.
- Prefer sibling worktree directories: `../<repo>.worktrees/<branch-slug>`.
- Use consistent slugs. Branch names may include namespaces such as `feature/my-task`; derive `branch-slug` as a lowercase kebab-case value safe for paths and hostnames, such as `my-task` or `feature-my-task`. Use the raw branch name for git commands and the slug for worktree directories, Portless names, browser profiles, and local state identifiers.
- Use named Portless URLs, not fixed localhost ports.
- Main should use `https://<project>.localhost`; feature worktrees should use `https://<branch-slug>.<project>.localhost`.
- Use project scripts for creation, doctor checks, overlap checks, finish/merge, and cleanup whenever available.
- Never delete, reset, checkout over, or force-clean a dirty worktree without explicit user approval.
- Do not merge when the integration checkout or feature worktree is dirty.
- For schema, migration, seed, queue, upload, or backend-state work, isolate the database/backend/cache for the worktree.

## Create A Worktree

Use the project command if present:

```bash
pnpm wt:doctor
pnpm wt:create feature/my-task
```

After creating the worktree, run the repo's documented setup step if the create script did not already do it. Check whether dependencies, env files, generated clients, and local state are worktree-local. Common setup includes `pnpm install` or equivalent, copying `.env.example` to `.env`, and setting worktree-specific values for ports, URLs, database names, cache prefixes, and storage prefixes. Never copy secrets blindly between worktrees; preserve required values but update anything that identifies mutable local state.

If the work changes canonical app state, schemas, migrations, seeds, queues, or shared backend data, look for an isolation flag or documented state setup. If no isolation exists, stop and explain the state collision risk before proceeding. Name the concrete shared resources involved; common risks include schema or migration conflicts, seed data overwrites, queue/job duplication, cache poisoning, object storage collisions, webhook/OAuth callback mixups, and local SQLite/database files being reused across worktrees.

After creation, report:

- worktree path
- branch name
- Portless URL
- any env/backend/database setup still required
- commands to start the app

## Run And Test

- Start the app through the repo's Portless script, usually `pnpm dev`, `pnpm next:dev`, or equivalent.
- For Portless installation, setup, and troubleshooting, defer to the official Portless skill: https://github.com/vercel-labs/portless/blob/main/skills/portless/SKILL.md.
- If Portless is unavailable, strongly recommend setting it up before running multiple worktrees. Use numeric ports only as a temporary fallback for local-only tasks, document the exact port in the handoff, and avoid numeric-port fallbacks for OAuth, webhooks, browser automation handoffs, or anything requiring stable callback URLs.
- Share the Portless URL with the user and browser tools.
- Use worktree-local browser profiles/sessions when available.
- Do not run two worktrees against the same mutable backend unless the repo explicitly supports it.
- If a runtime guard refuses to start a shared backend, inspect the guard output, then stop the stale process or create an isolated backend. Do not bypass the guard.

## Finish A Worktree

Prefer the project finish command:

```bash
pnpm wt:finish feature/my-task
```

A safe finish workflow should:

1. Confirm feature worktree is committed and clean.
2. Confirm integration checkout is clean.
3. Fast-forward pull the integration branch unless the user approves otherwise.
4. Run overlap checks against active worktrees.
5. Merge the feature branch.
6. Remove the linked worktree.
7. Delete only the merged feature branch.
8. Prune stale metadata/routes when supported.

After finishing, verify status in the integration checkout and run the relevant tests/build if the project expects it.

## Cleanup

- Use project cleanup scripts first.
- `wt:clean` should refuse dirty worktrees by default.
- `wt:prune` should remove stale metadata, not active work.
- Portless cleanup should target orphaned routes/processes only.
- Browser automation cleanup should target only the project's generated profiles/sessions.

## Handoff Format

When handing a worktree to a user or another agent, include:

```text
Branch: feature/my-task
Path: /path/to/repo.worktrees/feature-my-task
URL: https://feature-my-task.project.localhost
Start: pnpm dev
Doctor: pnpm wt:doctor
Finish: pnpm wt:finish feature/my-task
State: shared or isolated backend/database details
```

Keep guidance specific to the repo in front of you. This skill should not override project-local rules.
