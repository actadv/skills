---
name: test-coverage
description: Analyze test coverage gaps for a file or module and suggest specific tests to improve coverage
user-invocable: true
argument-hint: [file-or-directory]
allowed-tools: Read, Grep, Glob
---

Analyze test coverage and identify gaps.

## Steps

1. If `$ARGUMENTS` is provided, focus on that path. Otherwise, analyze the full `src/` or `lib/` directory.
2. For each source file, find its corresponding test file using common conventions:
   - `src/foo.ts` → `src/foo.test.ts`, `test/foo.test.ts`, `__tests__/foo.test.ts`
   - `pkg/foo.go` → `pkg/foo_test.go`
   - `app/foo.py` → `tests/test_foo.py`, `app/test_foo.py`
3. For files WITH tests, analyze coverage:
   - Read the source and identify all public functions/methods
   - Read the tests and identify what's covered
   - Flag untested functions, uncovered branches, missing edge cases
4. For files WITHOUT tests, flag them entirely.

## Output format

### Coverage Summary
| File | Functions | Tested | Coverage | Priority |
|------|-----------|--------|----------|----------|
| ... | ... | ... | ...% | high/med/low |

### Gaps by priority

**High priority** (public API, error-prone, recently changed):
- `file:function` — reason this matters

**Medium priority** (internal logic with branching):
- `file:function` — reason

**Low priority** (simple/stable code):
- `file:function` — reason

### Suggested tests
For each high-priority gap, provide a concrete test case outline:
```
describe('function', () => {
  it('should handle X', () => { ... })
})
```

## Rules
- Prioritize by risk: public API > complex logic > simple getters
- Recent changes (check git log) deserve higher priority
- Don't suggest testing trivial code (type definitions, re-exports)
