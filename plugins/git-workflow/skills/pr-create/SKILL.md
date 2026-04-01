---
name: pr-create
description: Create a GitHub pull request with an auto-generated summary, test plan, and review checklist based on the branch diff
user-invocable: true
disable-model-invocation: true
argument-hint: [base-branch]
allowed-tools: Bash(git *), Bash(gh *)
---

Create a well-structured GitHub pull request for the current branch.

## Steps

1. Determine the base branch: use `$ARGUMENTS` if provided, otherwise detect via `git remote show origin | grep 'HEAD branch'`.
2. Run `git log --oneline $(base)..HEAD` to see all commits on this branch.
3. Run `git diff $(base)...HEAD --stat` for a file-level summary.
4. Run `git diff $(base)...HEAD` for the full diff.
5. Check if the branch has been pushed: `git rev-parse --abbrev-ref --symbolic-full-name @{u}`. If not, push with `git push -u origin HEAD`.

## PR format

```markdown
## Summary
<!-- 2-4 bullet points covering what changed and why -->

## Changes
<!-- Group changes by area/concern -->

## Test plan
- [ ] Specific testing steps

## Review notes
<!-- Anything reviewers should pay attention to -->
```

6. Show the draft PR to the user for approval.
7. On approval, create it with `gh pr create --title "..." --body "..."`.
8. Return the PR URL.

## Rules
- PR title: max 70 chars, no period at end
- Link related issues if commit messages reference them
- Mark as draft if any commit message contains WIP/wip/fixup
