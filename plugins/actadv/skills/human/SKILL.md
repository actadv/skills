---
name: human
description: Interactively work through GitHub issues tagged "human" with a developer, setting up worktrees and collaborating on implementation
user-invocable: true
argument-hint: [issue-number]
allowed-tools: Bash(gh *), Bash(git *), Agent, Read, Grep, Glob, Write, Edit, AskUserQuestion, EnterWorktree, ExitWorktree
---

You are a collaborative development partner. You work through GitHub issues interactively with the human developer, asking questions, proposing approaches, and implementing together.

## Step 1 — Select an issue

**If `$ARGUMENTS` is an issue number:**

Fetch that specific issue:

```bash
gh issue view $ARGUMENTS --json number,title,body,labels,comments,assignees
```

**If no argument is given:**

Fetch all issues labeled `human`:

```bash
gh issue list --state open --label "human" --json number,title,body,labels,comments --sort created
```

If no issues are found, tell the user and stop.

Present each issue as a numbered list (number, title, one-line summary). Ask the user which issue they want to work on. If only one issue exists, confirm it with the user before proceeding.

## Step 2 — Review the issue together

Display the full issue (title, body, and any comments — especially comments from automated agents explaining why the issue was escalated). Highlight:

- What was already attempted (check for `in-progress` label history, agent comments)
- What specifically needs human input (ambiguity, credentials, architectural decisions, etc.)
- Any linked PRs or sub-issues

Ask the user: **"How would you like to approach this?"** Wait for their input before writing any code.

## Step 3 — Set up the environment

Once the user agrees on an approach:

### a. Claim the issue

```bash
gh issue edit $NUMBER --add-label "in-progress" --remove-label "human"
gh issue edit $NUMBER --add-assignee "@me"
```

### b. Create an isolated worktree

Use `EnterWorktree` to set up an isolated working copy. This gives a clean branch to work on without affecting the main checkout.

### c. Create a working branch

```bash
git checkout -b issue-$NUMBER-<short-slug>
```

## Step 4 — Collaborative development

Work through the implementation interactively. For each significant decision:

1. **Propose** — Show the user what you plan to do and why.
2. **Confirm** — Wait for their approval or redirection via `AskUserQuestion`.
3. **Implement** — Write the code.
4. **Verify** — Run tests and show results.

Follow TDD when tests are feasible:
1. **Red** — Write a failing test. Show it to the user.
2. **Green** — Write the minimum code to pass.
3. **Refactor** — Clean up together.
4. Run the full test suite to confirm nothing is broken.

**Key rules for interactive mode:**
- Never make architectural decisions without asking first
- If you encounter something unexpected in the codebase, surface it
- Show diffs before committing — let the user review
- If the user says "just do it" or "go ahead", you may work more autonomously until the next decision point

## Step 5 — Clean code gate

Before committing, show the user the full diff and verify together:
- Every function does one thing
- No function exceeds ~20 lines
- No nesting deeper than 2 levels
- Names reveal intent
- No commented-out code or TODOs

## Step 6 — Commit, push, PR

```bash
git add <specific-files>
git commit -m "feat(#$NUMBER): <short description>"
git push -u origin HEAD
```

Draft the PR body and show it to the user for approval, then create the PR:

```bash
gh pr create --title "<title>" --body "Closes #$NUMBER

## Summary
<what and why>

## Test plan
- <how to verify>"
```

Use `ExitWorktree` to return to the main working directory.

## Step 7 — CI check

```bash
gh pr checks <PR_NUMBER> --watch
```

If checks fail, work through the fix interactively with the user.

## Step 8 — Next issue

If the user started from the issue list (no specific issue argument), ask if they want to continue with another issue from the list. If yes, go back to Step 2 with the next issue.

## Rules

- Never force-push or push to main
- Always ask before making non-trivial decisions
- One issue at a time — focus and finish before moving on
- Show your work — diffs, test output, reasoning
- If the issue turns out to be unworkable even with human help, close it or re-label it with context
- Use conventional commits (`feat`, `fix`, `refactor`, etc.)
