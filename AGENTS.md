# Instructions for agents

## This is a spec-as-source project

- Human developers, with the help of agents as desired, create one or more `spec.md` files that comprehensively describe the behavior of the software, focused on how end users experience its behavior.
- Human developers and agents collaborate on developing `design.md` files whose purpose is to provide step-by-step instructions that any developer or agent can follow to successfully implement the software in adherence with the spec.
- Writing the application code that implements these designs is the primary job of agents.

## List of source files for a feature

Features consist of the following files, listed here in order of their precedence.

**Whenever there is doubt about a requirement or spec**, remember this hierarchy, from highest to lowest authority:

### 1. spec.md

The spec rules all, and is the source of truth for the design and tests.
Use the `spex-specify` skill when editing spec.md files.

### 2. design.md

Design docs are the source of truth for implementation guidance and certain details.
Use the `spex-design` skill when editing design.md files.

### 3. Tests

Use the `spex-write-tests` skill when adding or updating test coverage.

### 4. Reference implementations (optional)

These exist to guide the design, but can be overridden by tests, design, or spec.

### 5. The implementation (source code)

Use the `spex-implement` skill to write production code.

Refer to the implementation itself so that patterns and idioms are consistent across the project, but always defer to tests, design, and spec.

---

## Don't get stuck or caught in a loop

**Warning signs of being stuck:**

- Repeatedly using phrases like "Wait, I should..." or "Actually..." or "Let me reconsider..."
- Planning the same tool calls multiple times without executing
- Repeating the same text over and over
- Overthinking the order of operations for independent tasks
- Spending more time planning than executing

**If it feels like you are stuck:** stop, reassess, and ask the developer for guidance if necessary.
