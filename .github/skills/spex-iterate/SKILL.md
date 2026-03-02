---
name: spex-iterate
description: "Advance the project one production-ready step closer to faithfully implementing the spec."
---

# The `spex` methodology: markdown as source

**Your main priority is to follow the "spec as source" workflow described below.**

## Development workflow: overview

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

## Development workflow: high-level process

Follow a standard test-driven development (TDD) workflow.
Iterate on the application until:

- It fully implements the **spec**
- It obeys all **logic** and **details** specified by the developer
- There are 0 compiler errors and the entire project builds
- All tests pass

### How to approach the TDD workflow

- Tests translate the spec's scenarios into executable verification
- Each test must trace to one or more scenarios in the spec
    - If a test doesn't correspond to a scenario, question whether it's needed
    - If a scenario doesn't have a test, that's a gap
- Every piece of functionality follows the red-green-refactor cycle:
    - **1) Red:** Write a test for the next behavior you're about to implement. Run it. It should fail. If it passes, either the behavior already exists or the test is wrong.
        - When tests are "red", the build must still pass; test failures must be behavioral rather than static. This matters mainly because build errors tend to slow down development, and it's important we maintain high velocity in our workflow.
    - **2) Green:** Write the minimum production code to make the test pass.
    - **3) Refactor:** Clean up the code you just wrote (and the test if needed). Rename, extract, simplify. Run the tests again to confirm everything still passes.
- The power of TDD is the tight feedback loop: one test, one behavior, verify, repeat
- **Don't get stuck in repetitive loops.** If a test is hard to write, it may mean the design needs a different decomposition — flag it to the developer rather than churning. If you find yourself repeatedly rewriting the same test or going back and forth between test and implementation without converging, stop and reassess. Ask the developer for guidance if needed.

### Reminders for how to be maximally helpful during implementation

- Take ownership of making sure the build is always passing
    - The project must compile/build/parse cleanly at all times. Treat a broken build as a stop-the-world event.
    - **When to verify the build:**
        - After setting up or modifying project structure
        - After adding or changing any dependency
        - After completing each red-green-refactor cycle
        - After wiring components together
        - Before moving to the next step in the build sequence
- Follow existing vocabulary and naming conventions
    - Naming consistency across all documentation and code is one of the highest-leverage things you can do for long-term maintainability
    - This vocabulary is established at the spec level and flows downward — implementation doesn't get to rename things
- Keep functions and modules small
- Don't refactor while implementing. If you notice something that should be refactored, note it but keep it separate from the feature implementation. Mixing refactoring with new features makes both harder to verify.
- Respect existing boundaries. If the codebase has a clear separation between modules, maintain that separation.
- Add logging liberally, especially around complex logic and state transitions, to pre-emptively aid debugging and future maintenance.

### Don't get stuck or caught in a loop

**Warning signs of being stuck:**

- Repeatedly using phrases like "Wait, I should..." or "Actually..." or "Let me reconsider..."
- Planning the same tool calls multiple times without executing
- Repeating the same text over and over
- Overthinking the order of operations for independent tasks
- Spending more time planning than executing

**If it feels like you are stuck:** stop, reassess, and ask the developer for guidance if necessary.

### Regression hardening checklist -- REQUIRED FOR ALL BUGS:

When a bug reappears after being reported fixed, add at least one test that would have failed before the fix and that validates the user-visible outcome directly (not just intermediate state).

- Prefer assertion targets that encode end-user correctness:
    - For map camera behavior, assert resolved coordinates/centers (or equivalent deterministic camera inputs), not only that a pin list is non-empty.
    - For carousel-driven map updates, assert both initial load and slide-change behavior.
- Add at least one “alternative payload shape” case when backend data shape variance is plausible (for example, bounds-only location vs explicit point location).
- Document the regression cause and mitigation in the change `summary.md`.

---

## Development workflow: step-by-step process

### Before beginning to iterate, make sure you fully understand what is required

1. **Identify the scope.** Don't try to build everything at once. If no plan exists, create one.
2. **Understand the spec.** If there is a spec.md, read it. If there isn't, ask the developer to write one first.
3. **Understand the logic.** If there is a logic.md, read it. If there isn't, ask the developer to write one first. This will detail the state transitions and side effects that the test needs to worry about.
4. **Understand the existing tests, if any.** If there's an existing test suite, understand its structure, conventions, and coverage before adding to it. Match the existing style and organization, unless it becomes inconvenient.
5. **Understand the existing implementation, if any.**

### The feature or fix being implemented must be represented as a change in **`changes/`**

A feature or fix might have been reported ad-hoc by the user in the current conversation, or it might be a step in a **`plan.md`** file.

1. Create a change in the **`changes/`** folder, or extend an existing change if that is more appropriate
    - If a change belongs to a specific feature, it should live inside that feature; if it reaches across multiple features, it can live in the top-level `changes/` folder
2. Develop a **`plan.md`** to outline how you will approach the work
3. Develop a **`tasks.md`** to break down the work into manageable pieces and track your progress
4. Implement the code by going task-by-task, following the TDD workflow outlined above
5. When finished, generate a **`summary.md`** that summarizes the change, including any architectural or design decisions that were made during the course of implementation
6. Check to make sure files were all generated as expected, and that the project builds without errors
