---
name: refactor
description: Identify refactoring opportunities in a file or module and apply them safely while preserving behavior
user-invocable: true
argument-hint: <file-or-directory>
allowed-tools: Read, Grep, Glob, Edit, Bash(git diff *)
---

Analyze code and perform safe refactoring.

## Steps

1. Read the target at `$ARGUMENTS`.
2. Identify refactoring opportunities:

### Code smells to look for
- **Long functions** (>30 lines) — extract helpers
- **Deep nesting** (>3 levels) — early returns, guard clauses
- **Duplication** — extract shared logic (only when 3+ occurrences)
- **God objects** — classes/modules with too many responsibilities
- **Feature envy** — code that uses another module's data more than its own
- **Primitive obsession** — raw strings/numbers where types would help
- **Dead code** — unused functions, unreachable branches

3. For each opportunity, assess:
   - **Risk**: How likely is this refactoring to break something?
   - **Impact**: How much does this improve readability/maintainability?
   - **Effort**: How many files need to change?

4. Present findings sorted by impact/risk ratio (high impact + low risk first).

5. For each refactoring, show:
   - Current code
   - Proposed code
   - Why this is better

6. Ask the user which refactorings to apply.
7. Apply approved changes and verify the diff looks correct.

## Rules
- NEVER change behavior — refactoring is structure-only
- Verify all callers are updated when changing signatures
- If tests exist, suggest running them after changes
- Prefer small, incremental changes over big-bang rewrites
- When in doubt, leave it alone
