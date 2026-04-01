---
name: explain
description: Explain how a piece of code works — trace the execution flow, clarify complex logic, and map dependencies
user-invocable: true
argument-hint: <file-or-symbol>
allowed-tools: Read, Grep, Glob
---

Explain the specified code to help the user understand it.

## Steps

1. Read the target at `$ARGUMENTS`. This could be:
   - A file path — explain the module
   - A `file:line` — explain the specific function/block
   - A symbol name — find and explain it
2. Trace the execution flow:
   - What calls this code?
   - What does this code call?
   - What are the inputs and outputs?
3. Identify the non-obvious parts:
   - Why was it done this way?
   - What edge cases does it handle?
   - What are the implicit assumptions?

## Output structure

**TL;DR** — One sentence summary of what this code does.

**How it works** — Walk through the logic step by step. Use numbered steps for sequential flows, bullet points for branching logic.

**Key details:**
- Dependencies and their roles
- Side effects (I/O, state mutations, events)
- Error handling strategy
- Performance characteristics

**Gotchas** — Non-obvious behaviors, implicit assumptions, or common mistakes when modifying this code.

## Rules
- Adjust depth to complexity — don't over-explain simple code
- Use the project's terminology, not generic CS terms
- Reference specific line numbers
- If the code is confusing, say so — don't pretend bad code is clear
