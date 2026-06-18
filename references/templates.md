# Templates

Use these templates only when the user asks to create or revise commit messages, PR descriptions, AGENT.md rules, branch protection guidance, or review checklists.

## Agent Commit Message

```text
<type>(<scope>): <summary>

<body: explain the motivation, context, and any non-obvious implementation choice>

Agent-Task: <original task, ticket, or concise task summary>
Agent-Model: <model name if known; otherwise "unspecified">
Agent-Decision: <key design decision and rationale>
Agent-Limitation: <known limitation, risk, or "none known">
```

## Checkpoint Commit Message

```text
[WIP] <scope>: <checkpoint summary>

Checkpoint after <completed milestone>. Remaining work: <next milestone or risk>.

Agent-Task: <original task, ticket, or concise task summary>
Agent-Model: <model name if known; otherwise "unspecified">
Agent-Decision: <why this checkpoint boundary is meaningful>
Agent-Limitation: <known limitation, risk, or "temporary checkpoint">
```

## PR Description

```markdown
## Summary

- <change 1>
- <change 2>

## Agent Context

- Task: <original task or ticket>
- Model: <model name if known; otherwise "unspecified">
- Branch: <branch name>
- Base: <base branch>

## Key Decisions

- <decision and rationale>

## Validation

- [ ] <test, lint, typecheck, build, or manual check>

## Risks and Limitations

- <known limitation, rollback concern, migration concern, or "none known">

## Review Notes

- <files, modules, or commits that deserve focused review>
```

## AGENT.md Git Rules

```markdown
## Git Rules for Agent Work

- Never work directly on `main` or `master`.
- Start each new task from an up-to-date feature branch unless the user says otherwise.
- Run `git status --short --branch` before editing and before finishing.
- Treat existing uncommitted changes as user work. Do not overwrite, revert, or stage them unless explicitly instructed.
- Use checkpoint commits for long tasks, prefixed with `[WIP]`.
- Before PR creation, clean checkpoint commits into atomic commits.
- Each final commit must be buildable, testable, and limited to one logical change.
- Use Conventional Commits plus `Agent-Task`, `Agent-Model`, `Agent-Decision`, and `Agent-Limitation` trailers.
- Do not force-push, rewrite published history, merge PRs, or delete branches without explicit approval.
- For parallel agent tasks, use separate `git worktree` directories or the repository-approved equivalent.
```

## Branch Protection Checklist

```text
- Require pull request before merging.
- Require at least one human approval.
- Dismiss stale approvals when new commits are pushed.
- Require status checks to pass before merging.
- Restrict direct pushes to protected branches.
- Require secret scanning or equivalent checks when available.
- Require PR description sections for summary, agent context, validation, and limitations.
```

## History Cleanup Prompt

```text
Prepare this branch for PR.

1. Run `git log --oneline <base>..HEAD`.
2. Run `git diff --stat <base>...HEAD`.
3. Group commits into logical changes.
4. Propose which commits to keep, squash, fixup, reword, or drop.
5. Wait for confirmation before rewriting history.
6. After cleanup, show `git log --oneline <base>..HEAD` and validation results.

Every retained commit must follow Conventional Commits and include Agent-Task, Agent-Decision, and Agent-Limitation trailers.
```
