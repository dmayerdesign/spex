---
name: spex-iterate
description: "Advance the project one production-ready step closer to faithfully implementing the spec."
---

# Test-driven implementation

Follow a standard test-driven development (TDD) workflow.
Iterate on the application until:

- It fully implements the **spec**
- It obeys all **logic** and **details** specified by the developer
- There are 0 compiler errors and the entire project builds
- All tests pass

## Before beginning to iterate, make sure you fully understand what is required

1. **Identify the scope.** Don't try to build everything at once. If no plan exists, create one.
2. **Understand the spec.** If there is a spec.md, read it. If there isn't, ask the developer to write one first.
3. **Understand the logic.** If there is a logic.md, read it. If there isn't, ask the developer to write one first. This will detail the state transitions and side effects that the test needs to worry about.
4. **Understand the existing tests, if any.** If there's an existing test suite, understand its structure, conventions, and coverage before adding to it. Match the existing style and organization, unless it becomes inconvenient.
5. **Understand the existing implementation, if any.**

## How to approach the TDD workflow

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

## Reminders for how to be maximally helpful during implementation

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

## Output

Implementation produces working, tested code files organized in the project structure. After implementing, make sure:

- The project builds without errors
- All tests pass (both new and pre-existing)
- The implemented scenarios work when manually verified
- Files are saved to the appropriate project directory
