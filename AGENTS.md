# Instructions for agents

**Your main priority is to follow the "spec as source" workflow described below.**

---

# The `spex` methodology: markdown as source

## Development workflow

You are working with a developer to build a production-ready application.

### The developer's main responsibilities

- Specify high-level requirements
- Approve or veto high-level architecture decisions
- Regularly test, verify, and report bugs manually

### The agent's responsibilities: everything else

- Help to flesh out accurate high-level specs when asked
- Generate top-quality, idiomatic production code and tests
- Ensure all documentation and code is generated with **ease of understanding and readability** in mind
    - The developer needs to have an easy time understanding and updating implementation details

### Use the `spex-iterate` skill for most tasks

This ensures that we follow a consistent test-driven workflow, documenting our changes correctly as we go.

### Guiding principals

- **Less is more**
    - You must aim to be as economical as possible with the volume of documentation and code
    - As a general rule, avoid stating the obvious; focus mostly on surprises and deviations from standard assumptions
    - Only be redundant when doing so addresses a real ambiguity
- **Always be considering the perspective of the application's end users**
    - Only behavior that impacts the app's end-user experience need to be specified and tested
- **Every test must tie directly back to a given/when/then scenario in the spec**
- **By default, approach the application as a "plugin" architecture**
    - The **application logic** is a state machine with side effects, and is the target of most unit tests
    - The application logic defines abstract interfaces that the plugins implement
    - Anything considered an "implementation detail" (relative to the application logic) is handled by these plugins

### Project organization

Our source of truth in any given folder is the `spec/` folder.
The `spec/` folder for a "to-do" app might look like this:

```
spec/
├—— features/
│   ├── basic-todos/
│   │   ├—— changes/
│   │   │   └—— 2026-02-21-fix-styling-on-completed-todos/
│   │   │       ├—— bug-report.md
│   │   │       ├—— plan.md
│   │   │       ├—— tasks.md
│   │   │       └—— summary.md
│   │   ├—— spec.md
│   │   ├—— logic.md
│   │   └—— details.md
│   └── photo-uploads/
│       ├—— spec.md
│       ├—— logic.md
│       └—— details.md
├—— changes/
│       2026-02-21-add-photo-uploads-feature/
│       ├—— spec.md
│       ├—— plan.md
│       ├—— tasks.md
│       └—— summary.md
├—— resources/
│   └—— brand-assets/
│       └—— app-logo.png
├—— spec.md
├—— logic.md
└—— details.md
```

#### `features/` are nested specs

- Each feature has the same folder structure as this top-level `spec/` folder, minus `features/` (no deep nesting allowed here)
    - changes/
    - spec.md
    - logic.md
    - details.md

#### `changes/` contain every change made to the code

- Create a change whenever you are asked to iterate on a new deliverable chunk of work
- Before creating a new change, always check whether it would be more appropriate to extend an existing change
- If a change belongs to a specific feature, it should live inside that feature; if it reaches across multiple features, it can live in the top-level `changes/` folder

#### `resources/` are there to help

- Here you might find brand assets, reference implementations, or any non-source-code files to help in your implementation.

### File-level division of responsibility

All files must be considered on a continuum from _fully human-owned_ on one end to _fully agent-owned_ on the other.

#### 1. `spec.md` files and `bug-report.md` are owned by the human

- As an agent, you must never edit a file named `spec.md` or `bug-report.md` except when explicitly asked to by a human
- A spec is a collection of scenarios (in "given/when/then" form) grouped by user workflow
- Every scenario describes a **state transition** and its **side effects**

#### 2. `logic.md` files are mostly owned by the human

- As an agent, you must collaborate with the human developer to make sure this file describes every **state transition** and **side effect** possible in the application
- DO NOT include code snippets; DO include pseudocode when appropriate

#### 3. `details.md` files are mostly owned by you, the agent

- As an agent, you must document any noteworthy implementation details, in collaboration with the human developer
- Avoid over-specifying
    - Use ranges and qualitative descriptions when possible
    - Assume that well-known standards/conventions/idioms are implicitly understood and followed
- DO NOT include extensive code snippets; DO include pseudocode when appropriate

#### 4. All other documentation and code artifacts are owned by you, the agent

- As an agent you have discretion to generate any files you need to efficiently and effectively implement the spec
- Some options:
    - **summary.md** -- every change should have a summary that includes any architectural or design decisions that were made during the course of implementation
    - **plan.md** -- lay out a plan for how to attack a change
    - **tasks.md** -- track your progress on a change

---

## Don't get stuck or caught in a loop

**Warning signs of being stuck:**

- Repeatedly using phrases like "Wait, I should..." or "Actually..." or "Let me reconsider..."
- Planning the same tool calls multiple times without executing
- Repeating the same text over and over
- Overthinking the order of operations for independent tasks
- Spending more time planning than executing

**If it feels like you are stuck:** stop, reassess, and ask the developer for guidance if necessary.
