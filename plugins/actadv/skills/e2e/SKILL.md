---
name: e2e
description: Check recent GitHub Actions for e2e test failures, diagnose root causes, and fix them
user-invocable: true
argument-hint: [run-id | pr-number | "all" (default: all recent failures)]
allowed-tools: Bash(gh *), Bash(git *), Bash(npx *), Bash(npm *), Bash(pnpm *), Bash(yarn *), Agent, Read, Grep, Glob
---

You find and fix end-to-end test failures from recent GitHub Actions runs.

## Step 1 — Find failing runs

If `$ARGUMENTS` is a run ID, fetch that run. If it's a PR number, fetch runs for that PR. Otherwise, find all recent failures:

```bash
gh run list --status failure --limit 15 --json databaseId,name,headBranch,event,conclusion,createdAt,url
```

Filter to runs whose name contains `e2e`, `playwright`, `cypress`, `end-to-end`, or `integration` (case-insensitive). If none match, broaden to all failed runs and inspect their job names:

```bash
gh run view $RUN_ID --json jobs --jq '.jobs[] | select(.conclusion == "failure") | {name, conclusion}'
```

Keep only runs with e2e-related job failures. If still nothing, tell the user there are no recent e2e failures and stop.

## Step 2 — Extract failure details

For each failing run (up to 5), pull the logs:

```bash
gh run view $RUN_ID --log-failed 2>&1 | tail -500
```

Parse out:
- **Which tests failed** — test file, test name, line number
- **Error messages** — assertion failures, timeouts, element-not-found, network errors
- **Screenshots/traces** — note artifact references if present

Download test artifacts when available:

```bash
gh run download $RUN_ID --name <artifact-name> --dir /tmp/e2e-artifacts-$RUN_ID 2>/dev/null
```

Read any HTML reports, JSON results, or trace files from downloaded artifacts.

## Step 3 — Group and triage

Classify each failure:

| Category | Examples | Action |
|---|---|---|
| **Test bug** | Flaky selector, bad assertion, race condition, stale fixture | Fix the test |
| **App bug** | Regression, broken feature, API error | Fix the application code |
| **Infra/env** | Timeout, browser crash, missing env var, service unavailable | Escalate |

Group failures by root cause — multiple test failures often share one underlying issue.

## Step 4 — Fix

For each fixable root cause, spawn an **Agent** with `isolation: "worktree"` and `run_in_background: true` (up to 3 agents):

> You are fixing e2e test failure(s) from GitHub Actions run $RUN_ID on branch `$BRANCH`.
>
> **Context:**
> - Failing test(s): $TEST_FILES
> - Error(s): $ERROR_SUMMARY
> - Category: $CATEGORY (test bug | app bug)
> - Branch: `$BRANCH`
>
> **a. Checkout the branch**
>
> ```bash
> git checkout $BRANCH
> git pull origin $BRANCH
> ```
>
> **b. Reproduce locally**
>
> Detect the test runner:
> - `playwright.config.*` → `npx playwright test $TEST_FILE`
> - `cypress.config.*` → `npx cypress run --spec $TEST_FILE`
> - Other → read package.json scripts for the e2e command
>
> Run the failing test(s). Read the output carefully.
>
> **c. Diagnose**
>
> Read the failing test file and the application code it exercises. Use Grep and Glob to trace the code path. Identify the exact root cause — don't guess.
>
> **d. Fix**
>
> - **Test bug**: Fix selectors, add proper waits/retries, update assertions, stabilize fixtures. Do NOT just add arbitrary sleeps.
> - **App bug**: Fix the application code. Write or update unit tests for the fix. Ensure the e2e test now passes.
>
> **e. Verify**
>
> Run the fixed test(s) locally. Then run the full e2e suite to check for regressions:
>
> ```bash
> npx playwright test          # or equivalent
> ```
>
> If the full suite can't run locally (needs CI env), run at minimum the fixed tests.
>
> **f. Commit and push**
>
> ```bash
> git add <specific-files>
> git commit -m "fix(e2e): <what was wrong and why> (#$ISSUE_OR_PR)"
> git push origin $BRANCH
> ```
>
> **g. Verify CI**
>
> ```bash
> gh run list --branch $BRANCH --limit 1 --json databaseId,status
> gh run watch $RUN_ID --exit-status
> ```
>
> If CI fails again on a different issue, report it. If the same test fails, investigate further — do not loop more than twice.
>
> **h. Stuck? Escalate**
>
> If the fix requires env changes, secrets, external service fixes, or is ambiguous:
>
> ```bash
> gh issue create --title "e2e fix needs human: <summary>" --body "<details>" --label "human,bug"
> ```

## Step 5 — Handle infra/env failures

For failures categorized as infra/env, create an issue instead of attempting a fix:

```bash
gh label create "e2e-infra" --description "E2e infrastructure or environment issue" --color "F9D0C4" --force
gh issue create --title "e2e infra: <summary>" --body "## Failure
<error details>

## Run
$RUN_URL

## Likely cause
<analysis>

## Suggested fix
<what to change in CI config, env, or infra>" --label "e2e-infra,bug"
```

## Step 6 — Report

### E2e health summary

| Run | Branch | Failures | Root cause | Action | Status |
|---|---|---|---|---|---|
| #ID | branch | N tests | description | fixed / escalated / infra issue | link |

### Fixes pushed
- List each commit with branch, description, and CI status

### Issues created
- List each with number, title, and link

### Patterns noticed
If the same test or category fails repeatedly across runs, call it out — it may indicate a systemic issue (flaky test, unreliable service, missing test isolation).

## Rules

- Never force-push or push to main
- Never add arbitrary `sleep()` or `waitForTimeout()` to fix flaky tests — use proper waits (`waitForSelector`, `toBeVisible`, network idle, etc.)
- Don't disable or skip failing tests as a fix — fix the underlying issue
- If a test is genuinely obsolete (tests removed feature), delete it and note why in the commit message
- Prefer targeted fixes over large refactors
- Use conventional commits (`fix(e2e): ...`)
- One root cause per agent, one commit per fix
