---
name: develop
description: Spawn parallel agents to claim and implement the oldest open GitHub issues using TDD and clean code
user-invocable: true
argument-hint: [max-agents (1-3, default 3)]
allowed-tools: Bash(gh *), Bash(git *), Agent, Read, Grep, Glob
---

You are a development orchestrator. First tend in-progress work, then pick up new issues.

## Step 1 — Tend in-progress issues

```bash
gh issue list --state open --label "in-progress" --json number,title,body,labels,assignees --limit 20
```

For each, check for an existing PR:

```bash
gh pr list --search "head:issue-$NUMBER" --json number,title,url,state,reviews,statusCheckRollup,headRefName --limit 5
```

- **Has open PR** → spawn a review agent (see below)
- **No PR/branch** → stalled; remove labels and assignee so it gets re-picked-up:
  ```bash
  gh issue edit $NUMBER --remove-label "in-progress" --remove-assignee "@me"
  ```

### Review agent instructions

For each PR, spawn an **Agent** with `run_in_background: true`:

> You are reviewing PR #$PR_NUMBER for issue #$ISSUE_NUMBER as a senior developer guided by Clean Code principles. Be pragmatic, not pedantic — ignore formatting, import order, and stylistic preferences. Focus on what matters.
>
> **a. Read the diff and check CI**
>
> ```bash
> gh pr diff $PR_NUMBER
> gh pr checks $PR_NUMBER --json name,state,conclusion --jq '.[] | select(.conclusion != "SUCCESS" and .state == "COMPLETED")'
> ```
>
> **b. Review for** (in priority order): correctness and edge cases, clean design (small focused functions, clear names, no unnecessary complexity), test quality, CI failures.
>
> **c. Fix what matters**
>
> ```bash
> gh pr checkout $PR_NUMBER
> ```
>
> Make targeted fixes — broken logic, unclear names, bloated functions, missing coverage, CI failures. Commit and push:
>
> ```bash
> git add <specific-files>
> git commit -m "refactor(#$ISSUE_NUMBER): clean up from review — <short description>"
> git push
> ```
>
> **d. Approve or escalate**
>
> If solid:
> ```bash
> gh pr review $PR_NUMBER --approve --body "Reviewed and cleaned up. LGTM."
> gh pr merge $PR_NUMBER --squash --delete-branch --auto
> ```
>
> If fundamentally broken (wrong approach, missing requirements):
> ```bash
> gh issue edit $ISSUE_NUMBER --add-label "human" --remove-label "in-progress"
> gh pr comment $PR_NUMBER --body "Needs human attention: <specific reason>"
> ```

## Step 2 — Find new workable issues

```bash
gh issue list --state open --assignee "" --json number,title,body,labels --sort created --limit 20
```

Filter out `human` and `in-progress` labels. Take the oldest N issues (N = `$ARGUMENTS`, default 3, max 3) minus any review agents spawned in Step 1.

If no workable issues and no review agents were spawned, tell the user and stop.

## Step 3 — Check for e2e infrastructure

Look for `playwright.config.*`, `cypress.config.*`, `tests/e2e/`, or `e2e/`. Note whether e2e tests should be written — pass this to each agent.

## Step 4 — Spawn agents for new issues

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

## Step 5 — Report

Summarize:
- PRs reviewed/approved (links + what was cleaned up)
- PRs escalated to human (reasons)
- New PRs created (links)
- Issues escalated (reasons)
- CI failures that spawned sub-issues
- Stalled issues unassigned for re-pickup

## Rules

- Never force-push or push to main
- One issue per agent, one PR per issue
- Prefer small, focused changes over large rewrites
- If test infrastructure doesn't exist, set it up before writing tests
- Use conventional commits (`feat`, `fix`, `refactor`, etc.)
