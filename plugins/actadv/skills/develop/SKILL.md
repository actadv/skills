---
name: develop
description: Spawn parallel agents to claim and implement the oldest open GitHub issues using TDD and clean code
user-invocable: true
argument-hint: [max-agents (1-3, default 3)]
allowed-tools: Bash(gh *), Bash(git *), Bash(bash *), Agent, Read, Grep, Glob
---

You are a development orchestrator. Pick up new issues and implement them.

## Step 0 — Check global agent capacity

Before doing anything, check if there are available agent slots:

```bash
bash ~/.claude/agent-registry.sh can-spawn 1
```

If the answer is "no" (exit code 1), tell the user the global agent limit is reached and stop. Show the current count and limit.

## Step 1 — Find workable issues

```bash
gh api repos/{owner}/{repo}/issues --cache 0s --paginate -q '[.[] | select(.assignee == null and (.pull_request | not)) | {number, title, body, labels: [.labels[] | {name}]}]'
```

Filter out `human` and `in-progress` labels. Take the oldest N issues (N = `$ARGUMENTS`, default 3, max 3).

**Cap N** to the number of available agent slots (from Step 0). If 2 slots are free and N=3, only take 2 issues.

If no workable issues, tell the user and stop.

## Step 1.5 — Register agents

Before spawning each agent, register it:

```bash
bash ~/.claude/agent-registry.sh register "develop-issue-$NUMBER" "Implementing #$NUMBER: $TITLE"
```

When the agent completes (task notification), deregister it:

```bash
bash ~/.claude/agent-registry.sh deregister "develop-issue-$NUMBER"
```

## Step 2 — Check for e2e infrastructure

Look for `playwright.config.*`, `cypress.config.*`, `tests/e2e/`, or `e2e/`. Note whether e2e tests should be written — pass this to each agent.

## Step 3 — Spawn agents

For each issue, launch an **Agent** with `isolation: "worktree"` and `run_in_background: true`. Include the issue number, title, full body, and whether e2e tests are expected.

---

## Agent instructions

Implement a single GitHub issue end-to-end. Work autonomously. Write clean, minimal code.

### a. Claim

```bash
gh issue edit $NUMBER --add-label "in-progress" --add-assignee "@me"
```

### b. Branch

```bash
git checkout -b issue-$NUMBER-<short-slug>
```

### c. Understand

Read the issue. Use Grep, Glob, and Read to explore relevant code. Understand the domain before writing anything.

### d. TDD cycle

1. **Red** — Failing test encoding the requirement.
2. **Green** — Minimum code to pass.
3. **Refactor** — Small functions, clear names, no duplication, no dead code.
4. Run the full test suite.

Repeat for each behavior the issue requires.

### e. E2e tests

If e2e infrastructure exists, write e2e tests covering the new behavior.

### f. Clean code gate

Before committing, verify: every function does one thing, no function exceeds ~20 lines, no nesting deeper than 2 levels, names reveal intent, no commented-out code or TODOs.

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

Run `gh pr checks <PR_NUMBER> --watch` (timeout 10 min). If checks fail, fix and push. If the fix fails again:

```bash
gh issue create --title "CI fix needed for #$NUMBER: <failure summary>" --body "Parent: #$NUMBER\n\n<details>" --label "bug"
```

### i. Stuck? Escalate

If ambiguous, needs credentials, or requires infrastructure changes:

```bash
gh issue edit $NUMBER --add-label "human" --remove-label "in-progress"
gh issue comment $NUMBER --body "Needs human intervention: <specific reason>"
```

Do NOT guess. Escalate early.

---

## Step 4 — Report

Summarize:
- New PRs created (links)
- Issues escalated (reasons)
- CI failures that spawned sub-issues

## Rules

- Never force-push or push to main
- One issue per agent, one PR per issue
- Prefer small, focused changes over large rewrites
- If test infrastructure doesn't exist, set it up before writing tests
- Use conventional commits (`feat`, `fix`, `refactor`, etc.)
