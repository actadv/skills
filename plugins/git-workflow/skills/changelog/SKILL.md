---
name: changelog
description: Generate a changelog entry from recent commits, grouped by type (features, fixes, breaking changes)
user-invocable: true
argument-hint: [since-tag-or-ref]
allowed-tools: Bash(git *), Read, Glob
---

Generate a changelog from git history.

## Steps

1. Determine the starting point:
   - If `$ARGUMENTS` is provided, use it as the since-ref.
   - Otherwise, find the latest tag: `git describe --tags --abbrev=0 2>/dev/null`. If no tags exist, use the first commit.
2. Run `git log --oneline $(since)..HEAD` to get all commits.
3. Parse and categorize commits:
   - **Breaking Changes** — commits with `BREAKING CHANGE` or `!` after type
   - **Features** — `feat:` commits
   - **Bug Fixes** — `fix:` commits
   - **Performance** — `perf:` commits
   - **Other** — everything else (group by type)
4. Check if a `CHANGELOG.md` exists. If so, read its format and match it.

## Output format

```markdown
## [version] - YYYY-MM-DD

### Breaking Changes
- description (#PR)

### Features
- description (#PR)

### Bug Fixes
- description (#PR)
```

5. Show the generated entry to the user.
6. If `CHANGELOG.md` exists, offer to prepend the entry. If not, offer to create it.

## Rules
- Use present tense ("add", not "added")
- Include PR/issue numbers when found in commit messages
- Skip merge commits and fixup commits
- Group related changes together
