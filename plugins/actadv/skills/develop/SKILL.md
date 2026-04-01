---
name: develop
description: Spawn parallel agents to claim and implement the oldest open GitHub issues using TDD and clean code
user-invocable: true
argument-hint: [max-agents (1-5, default 5)]
allowed-tools: Bash(gh *), Bash(git *), Agent, Read, Grep, Glob
---

You are a development orchestrator. Pick up open GitHub issues and deliver working PRs in parallel.

## Step 1 — Find workable issues

```bash
gh issue list --state open --assignee "" --json number,title,body,labels --sort created --limit 20
```

Filter out issues that have the `human` or `in-progress` label. Take the oldest N issues where N is `$ARGUMENTS` (default 5, max 5).

If no workable issues exist, tell the user and stop.

## Step 2 — Check for e2e infrastructure

Look for `playwright.config.*`, `cypress.config.*`, a `tests/e2e/` or `e2e/` directory. Note whether e2e tests should be written — pass this to each agent.

## Step 3 — Spawn agents

For each issue, launch an **Agent** with `isolation: "worktree"` and `run_in_background: true`. Include the issue number, title, full body, and whether e2e tests are expected. Each agent receives the instructions below.

---

## Agent instructions

You are implementing a single GitHub issue end-to-end. Work autonomously. Write clean, minimal code.

### a. Claim

```bash
gh issue edit $NUMBER --add-label "in-progress" --add-assignee "@me"
```

### b. Branch

```bash
git checkout -b issue-$NUMBER-<short-slug>
```

### c. Understand

Read the issue carefully. Use Grep, Glob, and Read to explore relevant code. Understand the domain before writing anything.

### d. TDD cycle

1. **Red** — Write a failing test that encodes the requirement.
2. **Green** — Write the minimum code to pass it.
3. **Refactor** — Clean up. Small functions, clear names, no duplication, no dead code.
4. Run the full test suite to confirm nothing is broken.

Repeat for each behavior the issue requires.

### e. E2e tests

If the orchestrator indicated e2e infrastructure exists, write e2e tests covering the new behavior using the project's existing e2e framework.

### f. Clean code gate

Before committing, review your diff. Verify:
- Every function does one thing
- No function exceeds ~20 lines
- No nesting deeper than 2 levels
- Names reveal intent
- No commented-out code or TODOs

### g. Commit, push, PR

```bash
git add <specific-files>
git commit -m "feat(#$NUMBER): <short description>"
git push -u origin HEAD
gh pr create --title "<title>" --body "Closes #$NUMBER

## Summary
<what and why>

## Test plan
- <how to verify>"
```

### h. CI

Run `gh pr checks <PR_NUMBER> --watch` (timeout 10 min). If checks fail:
1. Read the failure output.
2. Fix and push. If the fix fails a second time, create a sub-issue:
   ```bash
   gh issue create --title "CI fix needed for #$NUMBER: <failure summary>" --body "Parent: #$NUMBER\n\n<details>" --label "bug"
   ```

### i. Stuck? Escalate

If the issue is ambiguous, needs credentials, requires infrastructure changes, or is otherwise unworkable after a good-faith attempt:

```bash
gh issue edit $NUMBER --add-label "human" --remove-label "in-progress"
gh issue comment $NUMBER --body "Needs human intervention: <specific reason>"
```

Do NOT guess. Escalate early.

---

## Step 4 — Report

As agents complete, summarize:
- Issues that got PRs (with PR links)
- Issues escalated to human (with reasons)
- CI failures that spawned sub-issues

## Rules

- Never force-push or push to main
- One issue per agent, one PR per issue
- Prefer small, focused changes over large rewrites
- If test infrastructure doesn't exist, set it up before writing tests
- Use conventional commits (`feat`, `fix`, `refactor`, etc.)
