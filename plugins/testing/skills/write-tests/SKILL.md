---
name: write-tests
description: Generate comprehensive tests for a file or function, following the project's existing test patterns and framework
user-invocable: true
argument-hint: <file-or-function>
allowed-tools: Read, Grep, Glob, Bash(npm test *), Bash(npx *), Bash(pytest *), Bash(go test *), Bash(cargo test *)
---

Write tests for the specified code.

## Steps

1. Read the target at `$ARGUMENTS`.
2. Detect the test framework and patterns used in this project:
   - Look for existing test files near the target
   - Check config files (jest.config, pytest.ini, Cargo.toml, etc.)
   - Match the naming convention (`.test.ts`, `_test.go`, `test_*.py`, etc.)
3. Analyze the code to identify:
   - Public API surface to test
   - Input types and valid/invalid ranges
   - Edge cases and boundary conditions
   - Error paths and exception handling
   - Side effects that need mocking
4. Write tests covering:

### Happy path
- Normal inputs produce expected outputs
- All branches are exercised

### Edge cases
- Empty inputs, null/undefined, zero values
- Boundary values (max int, empty string, single element)
- Unicode, special characters where relevant

### Error cases
- Invalid inputs produce proper errors
- External failures are handled gracefully

### Integration points
- Interactions with dependencies behave correctly
- Side effects occur as expected

5. Run the tests to verify they pass.

## Rules
- Follow the project's existing test style exactly
- Use descriptive test names that explain the scenario
- One assertion per concept (not necessarily per test)
- Prefer real objects over mocks; mock only external boundaries
- Don't test private implementation details
- Tests should be deterministic — no random data, no timing dependencies
