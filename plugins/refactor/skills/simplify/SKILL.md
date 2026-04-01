---
name: simplify
description: Simplify overly complex code by reducing nesting, removing redundancy, and improving clarity without changing behavior
user-invocable: true
argument-hint: <file>
allowed-tools: Read, Grep, Glob, Edit
---

Simplify the specified code for maximum clarity.

## Steps

1. Read the file at `$ARGUMENTS`.
2. Identify complexity hotspots:

### Simplification patterns

**Control flow:**
- Convert nested if/else to early returns
- Replace complex conditionals with named booleans or helper functions
- Simplify switch/match with lookup tables where appropriate
- Remove unnecessary else after return/throw/continue

**Data flow:**
- Inline variables used only once (unless the name adds clarity)
- Replace imperative loops with declarative alternatives (map/filter/reduce) when clearer
- Simplify boolean expressions (`if (x === true)` → `if (x)`)

**Structure:**
- Flatten unnecessary wrapper functions
- Remove redundant type assertions or casts
- Collapse single-use abstractions back into their call site
- Remove defensive code that can never trigger

3. For each simplification:
   - Show before and after
   - Confirm it preserves behavior
   - Explain why simpler is better here

4. Apply changes and show the final diff.

## Rules
- Simpler means fewer concepts to hold in your head, not fewer characters
- Don't sacrifice clarity for cleverness
- Preserve all behavior — this is NOT a refactoring that changes structure
- If a piece of "complex" code is complex because the problem is complex, leave it and add a comment instead
