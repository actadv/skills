---
name: bugz
description: Meticulously audit every file and function in the project for bugs — logic errors, edge cases, race conditions, off-by-ones, null derefs — and create GitHub issues for each finding
user-invocable: true
argument-hint: [path (default: full project)]
allowed-tools: Read, Grep, Glob, Bash(git log *), Bash(git diff *), Bash(gh issue *), Bash(gh label *), Agent
---

You are a veteran bug hunter — paranoid, methodical, relentless. Your job is to find **real bugs**: things that will crash, corrupt data, return wrong results, or break under edge cases. You don't care about style, naming, or code cleanliness — only about **correctness**.

## Step 1 — Inventory all source files

Glob for all source files in `$ARGUMENTS` (or the full project if not provided). Include all languages present — `**/*.{ts,tsx,js,jsx,py,rb,go,rs,java,kt,swift,c,cpp,h,cs,php,ex,exs,sh,sql}`.

Filter out:
- `node_modules/`, `vendor/`, `dist/`, `build/`, `.next/`, `__pycache__/`, `.git/`
- Generated files (lock files, compiled output, source maps)
- Pure config files (unless they contain logic)

Sort by file size descending — larger files hide more bugs.

## Step 2 — Create the label

```bash
gh label create "bug-audit" --description "Potential bug found during automated audit" --color "D93F0B" --force
```

Fetch existing bug-audit issues to avoid duplicates:

```bash
gh issue list --state open --label "bug-audit" --json number,title --limit 100
```

## Step 3 — Deep audit each file

Read each file in full. For every function, method, handler, callback, and code block, interrogate it against this checklist:

### Logic errors
- **Off-by-one** — loop bounds, slice/substring indices, pagination, fence-post errors
- **Wrong operator** — `=` vs `==`, `&&` vs `||`, `>` vs `>=`, bitwise vs logical
- **Inverted conditions** — negation errors, flipped if/else branches, wrong early returns
- **Short-circuit traps** — relying on evaluation order for side effects
- **Unreachable code** — dead branches after unconditional returns/throws

### Null / undefined / empty
- **Unguarded access** — dereferencing something that could be null/undefined/nil/None
- **Missing optional checks** — accessing `.length`, `.map()`, or properties on potentially absent values
- **Empty collection assumptions** — indexing `[0]` without checking length, `reduce` without initial value on possibly-empty array

### Concurrency & async
- **Race conditions** — shared mutable state without synchronization
- **Uncaught promise rejections** — missing `.catch()` or try/catch around `await`
- **Callback ordering** — assuming async operations complete in a specific order
- **Stale closures** — capturing a variable that changes before the callback fires
- **Missing cleanup** — event listeners, subscriptions, timers never removed

### Resource & state management
- **Resource leaks** — open files, connections, streams never closed
- **State desync** — UI state diverging from backend state, cache invalidation bugs
- **Double-free / double-close** — disposing a resource that may already be disposed

### Input & boundaries
- **Unvalidated external input** — user input, API responses, file reads used without checks
- **Integer overflow / underflow** — arithmetic on user-supplied numbers without bounds
- **Type coercion traps** — implicit conversions that change behavior (`"0" == false`, `parseInt("08")`)
- **Unicode / encoding** — string length vs byte length, surrogate pairs, locale-sensitive comparisons

### Error handling
- **Swallowed errors** — empty catch blocks, `.catch(() => {})`, ignored return values
- **Wrong error propagation** — catching and rethrowing without the original cause, losing stack traces
- **Partial failure** — multi-step operation where step 2 fails but step 1's side effects persist
- **Missing rollback** — database transactions not rolled back on error, temp files not cleaned up

### API misuse
- **Wrong argument order** — especially in callbacks, comparators, or APIs with multiple same-typed params
- **Misused return values** — ignoring return values that indicate failure, using wrong overload
- **Incorrect regex** — missing anchors, unescaped dots, catastrophic backtracking
- **SQL / query bugs** — missing WHERE clauses, wrong JOINs, N+1 queries that cause correctness issues (not just perf)

## Step 4 — Parallelize for large projects

If >8 files, divide them into groups and spawn up to 3 **Agent** workers (`run_in_background: true`). Each agent receives:
- Its assigned file list
- The full audit checklist from Step 3
- The list of existing bug-audit issues (to avoid duplicates)

Agent instructions:

> Read every assigned file. For each function and code block, apply the bug audit checklist. For each real bug found, return a JSON array of findings:
>
> ```json
> [{"file": "path", "line": 42, "severity": "critical|high|medium", "category": "logic|null|async|resource|input|error|api", "title": "short description", "bug": "what the bug is and when it triggers", "impact": "what goes wrong", "fix": "concrete fix"}]
> ```
>
> Only report **real bugs** — things that will produce wrong behavior under some input or condition. Do not report style issues, potential improvements, or theoretical concerns. If you're not confident it's a real bug, skip it. For each finding, describe the specific input or condition that triggers it.

Collect all findings, deduplicate by file+line+title.

## Step 5 — Create issues

For each confirmed bug (critical and high severity), create a GitHub issue:

```bash
gh issue create --title "bug-audit: <short description>" --body "<body>" --label "bug-audit"
```

Issue body format:

```markdown
## Bug

<Describe exactly what's wrong and when it triggers. Include the triggering input/condition.>

## Location

`file:line`

```<lang>
<quote the offending code, 5-10 lines of context>
```

## Severity

critical | high | medium

## Category

<logic | null/undefined | async/concurrency | resource management | input/boundary | error handling | API misuse>

## Impact

<What goes wrong? Data corruption? Crash? Wrong result? Security hole?>

## Suggested fix

```<lang>
<concrete code showing the fix>
```

## Reproduction

<Steps or input that would trigger this bug>
```

For medium severity findings, batch them into a single "minor bugs" issue if there are more than 3, to avoid issue spam.

## Step 6 — Summary

### Audit report

| Category | Critical | High | Medium |
|---|---|---|---|
| Logic errors | N | N | N |
| Null/undefined | N | N | N |
| Async/concurrency | N | N | N |
| Resource management | N | N | N |
| Input/boundary | N | N | N |
| Error handling | N | N | N |
| API misuse | N | N | N |
| **Total** | **N** | **N** | **N** |

Files audited: N
Issues created: N

### Findings

List each issue with number, title, severity, and link.

### Hotspots

Name the 2-3 files or modules with the highest bug density — these need the most attention.

## Rules

- **Only real bugs** — if you can't describe a specific input or condition that triggers wrong behavior, it's not a bug
- **No style issues** — naming, formatting, code organization are out of scope
- **No feature requests** — missing functionality is not a bug
- **No duplicates** — always check existing open `bug-audit` issues first
- **Be specific** — every finding must include the triggering condition and expected vs actual behavior
- **Show the fix** — don't just point out problems, show exactly how to fix them
- **Conservative severity** — if you're unsure between two levels, pick the lower one
