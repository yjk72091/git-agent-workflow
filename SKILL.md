---
name: git-agent-workflow
description: Agent-aware Git workflow guidance for coding tasks, repository hygiene, commit planning, history cleanup, PR preparation, and multi-agent isolation. Use when Codex is asked to make code changes with Git discipline, design or audit an AI-agent version-control process, split large diffs into atomic commits, prepare agent commits or PRs, manage checkpoint commits, use worktrees for concurrent agents, write AGENT.md Git rules, or choose between Git, jj, and GitButler for agentic coding.
---

# Git Agent Workflow

## Core Rule

Treat Git history as the durable record of agent intent. Keep changes isolated, reviewable, reversible, and traceable before merging.

Do not rewrite published history, force-push, merge, or delete branches unless the user explicitly asks or the repository's instructions allow it. Never overwrite user WIP.

## Start-of-Task Checklist

Before editing a repository:

1. Inspect state with `git status --short --branch`.
2. Identify the base branch and current branch with `git branch --show-current` and project docs if available.
3. If user changes are present, treat them as off-limits unless they are clearly part of the requested task.
4. For a new task, prefer a feature branch from an up-to-date base branch.
5. For concurrent or risky work, prefer `git worktree` isolation over sharing one working tree.
6. Record the user task, constraints, planned validation, and any risky assumptions in notes for the final PR or commit message.

## Branch and Worktree Discipline

Use feature branches for agent work. Avoid working directly on `main` or `master`.

Prefer branch names like:

```text
agent/<ticket-or-topic>
feat/<topic>
fix/<topic>
```

For parallel agents or independent subtasks, use separate worktrees:

```bash
git fetch origin
git worktree add ../repo-task-auth -b agent/auth-refresh origin/main
git worktree list
```

Remove a completed worktree only after confirming no uncommitted work remains:

```bash
git status --short
git worktree remove ../repo-task-auth
```

## Commit Strategy

Use two commit modes:

- Checkpoint commits: temporary progress saves for long-running work; prefix with `[WIP]`.
- Atomic commits: final reviewable commits; each commit expresses one logical, buildable, testable change.

Create checkpoint commits at meaningful boundaries such as data model/API shape, core implementation, tests, docs, or migration scripts. Before opening a PR, clean them into atomic commits with rebase or another history tool.

An atomic commit must:

- Represent one logical change, not one file and not an entire project.
- Leave the codebase buildable and reasonably testable.
- Avoid mixing feature work, refactors, formatting, dependency changes, and generated files unless they are inseparable.
- Be reversible without removing unrelated work.
- Include tests or explain why validation is not possible.

## Commit Messages

Use Conventional Commits plus agent trailers:

```text
<type>(<scope>): <summary>

<body explaining why the change exists and any important context>

Agent-Task: <original task, ticket, or concise task summary>
Agent-Model: <model name if known; otherwise "unspecified">
Agent-Decision: <key design decision and rationale>
Agent-Limitation: <known limitation, risk, or "none known">
```

Use `git interpret-trailers`-compatible `Key: Value` lines. Do not invent vague trailers that repeat the summary.

Useful checks:

```bash
git log --format='%(trailers:key=Agent-Task,valueonly)' main..HEAD
git log --grep='^Agent-Task:' --all
```

## Dirty Worktree Control

Before committing:

1. Inspect `git diff --stat`, `git diff`, and `git diff --staged`.
2. Separate unrelated formatting, generated files, dependency lockfiles, fixture updates, and business logic.
3. Stage intentionally with `git add -p` when the diff contains mixed concerns.
4. Search for accidental secrets or debug code when touching config, env, auth, logging, or API clients.
5. If a file contains user WIP plus agent changes, isolate only the requested hunks or stop and ask.

## History Cleanup

Before PR creation, inspect the branch:

```bash
git log --oneline main..HEAD
git diff --stat main...HEAD
```

Propose a cleanup plan before rewriting history:

- Which commits to keep as independent atomic commits.
- Which `[WIP]` commits to squash or fixup.
- Which messages need `reword`.
- Which unrelated commits should be moved, dropped, or deferred.

Use `git rebase -i main` only when it is safe for the branch. After cleanup, show the final `git log --oneline main..HEAD` and run relevant validation.

## PR Preparation

An agent PR should make reviewer context explicit:

- What changed.
- Why it changed.
- Original task or ticket.
- Key agent decisions and tradeoffs.
- Test/validation evidence.
- Known limitations and follow-ups.
- Risk areas, especially public APIs, auth, data migrations, concurrency, dependencies, generated code, or shared packages.

Read `references/templates.md` when drafting commit messages, PR descriptions, AGENT.md Git rules, or branch protection guidance.

## Monorepo Guidance

In monorepos, define atomicity by semantic completeness. A shared API change plus required consumers may belong in one commit if splitting would leave the repo broken. Prefer stacked PRs when a large task has reviewable layers, such as:

1. Shared package or schema change.
2. Service implementation.
3. Client integration.
4. Tests and docs.

Use dependency graph tooling when available (`nx affected`, `turbo`, package manager workspaces, Bazel, etc.) to scope validation.

## Tool Selection

Default to plain Git when the repository already uses it and the task is small.

Consider `git worktree` when multiple agents or risky tasks need filesystem isolation.

Consider `jj` when the user or repo already uses it, or when repeated split/squash/rebase operations are central to the task. Favor `jj split`, `jj absorb`, and `jj op undo` for mixed agent diffs and history repair.

Consider GitButler when the user explicitly uses it or asks for virtual branches, hunk assignment, stacked branches, or GUI-assisted multi-agent workflows.

Do not introduce `jj`, GitButler, gh-stack, or new branch protection automation into a repo without user agreement.

## Validation Gate

Before declaring the Git work ready:

1. Confirm `git status --short` is expected.
2. Confirm each commit is reviewable and traceable.
3. Run relevant tests, linters, or affected-package checks.
4. Report what was not validated.
5. If preparing a PR, verify the PR body includes decisions, limitations, and validation evidence.
