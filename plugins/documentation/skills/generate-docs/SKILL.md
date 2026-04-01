---
name: generate-docs
description: Generate or update documentation for a module, API, or component by reading the source code and producing clear, accurate docs
user-invocable: true
argument-hint: <file-or-directory>
allowed-tools: Read, Grep, Glob
---

Generate documentation for the specified code.

## Steps

1. Read the target at `$ARGUMENTS`. If it's a directory, identify the main entry points and public API surface.
2. For each public function, class, type, or endpoint:
   - Determine its purpose from the implementation
   - Identify parameters, return types, and side effects
   - Find usage examples by grepping for call sites in the codebase
3. Detect the existing documentation format in the project (JSDoc, docstrings, rustdoc, godoc, etc.) and match it.

## Output format

Produce documentation in the style that matches the project:

**For API endpoints:**
```
### METHOD /path
Description
Parameters: ...
Response: ...
Example: ...
```

**For functions/classes:**
Match the language's standard doc comment format with:
- Brief description
- Parameter descriptions
- Return value
- Example usage (drawn from real call sites when possible)
- Thrown exceptions / error cases

**For modules/packages:**
- Overview of purpose and responsibilities
- Public API surface
- Usage patterns
- Dependencies and relationships

## Rules
- Document what ISN'T obvious from the signature — the why, not the what
- Include real examples from the codebase over contrived ones
- Flag any undocumented public APIs you find
- Don't generate docs for private/internal code unless asked
