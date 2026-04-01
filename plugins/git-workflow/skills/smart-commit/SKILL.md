---
name: smart-commit
description: Analyze staged changes and create a well-crafted conventional commit with an accurate, descriptive message
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash(git *)
---

Create a commit for the currently staged changes.

## Steps

1. Run `git diff --cached --stat` to see what files are staged. If nothing is staged, tell the user and stop.
2. Run `git diff --cached` to read the full diff.
3. Run `git log --oneline -10` to understand the project's commit message style.
4. Analyze the changes and determine:
   - The type: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `style`, `ci`, `build`
   - An optional scope (module/component affected)
   - Whether this is a breaking change
5. Draft a commit message following Conventional Commits:
   ```
   type(scope): short description

   Optional body explaining the why, not the what.
   ```
6. Show the proposed message to the user and ask for confirmation.
7. On confirmation, create the commit.

## Rules
- Subject line max 72 characters
- Use imperative mood ("add", not "added" or "adds")
- Body should explain motivation, not repeat the diff
- If changes span multiple concerns, suggest splitting into multiple commits
