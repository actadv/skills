---
name: issues
description: List current open GitHub issues with their labels, organized by tag for a quick project overview
user-invocable: true
argument-hint: [label-filter]
allowed-tools: Bash(gh *), AskUserQuestion
---

You list open GitHub issues and their labels in a concise, scannable format.

## Step 1 — Fetch issues

If `$ARGUMENTS` is provided, use it as a label filter:

```bash
gh issue list --state open --label "$ARGUMENTS" --json number,title,labels,assignees --limit 10
```

Otherwise, fetch all open issues:

```bash
gh issue list --state open --json number,title,labels,assignees --limit 10
```

## Step 2 — Display

Present the issues in two views:

### By label

Group issues by their labels. For each label, list the issues under it. Issues with multiple labels appear under each label. Show untagged issues in a separate "unlabeled" group.

Format each issue as: `#number — title` (with assignee if assigned).

### Full list

A flat table of all issues sorted by issue number (oldest first):

```
#number  title                          labels              assignee
```

## Step 3 — Summary

End with a one-line summary: total open issues shown, count of distinct labels, and count of unassigned issues.

## Step 4 — Prompt for more

If exactly 10 issues were returned (meaning there are likely more), ask the user if they want to see more using AskUserQuestion. If they do, fetch the next batch with `--limit 50` and display the full set.
