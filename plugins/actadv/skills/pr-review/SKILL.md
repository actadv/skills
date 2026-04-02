---
name: pr-review
description: Review and tend in-progress PRs — check CI, fix issues, approve/merge, or escalate to human
user-invocable: true
argument-hint: [max-agents (1-5, default 5)]
allowed-tools: Bash(gh *), Bash(git *), Agent, Read, Grep, Glob
---

You tend all open PRs linked to in-progress issues. Review, fix, merge, or escalate each one.

## Step 1 — Find in-progress issues with PRs

```bash
gh issue list --state open --label "in-progress" --json number,title,body,labels,assignees --limit 20
```

For each, check for an existing PR:

```bash
gh pr list --search "head:issue-$NUMBER" --json number,title,url,state,reviews,statusCheckRollup,headRefName --limit 5
```

- **Has open PR** → spawn a review agent (see below)
- **No PR/branch** → stalled; remove labels and assignee so it gets re-picked up:
  ```bash
  gh issue edit $NUMBER --remove-label "in-progress" --remove-assignee "@me"
  ```

Spawn up to N review agents (N = `$ARGUMENTS`, default 5, max 5).

## Step 2 — Review agents

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

## Step 3 — Report

Summarize:
- PRs reviewed/approved (links + what was cleaned up)
- PRs merged
- PRs escalated to human (reasons)
- Stalled issues unassigned for re-pickup

## Rules

- Never force-push or push to main
- One PR per agent
- Prefer targeted fixes over large rewrites
- Use conventional commits (`feat`, `fix`, `refactor`, etc.)
