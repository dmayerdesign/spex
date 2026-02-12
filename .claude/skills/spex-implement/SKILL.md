---
name: spex-implement
description: "Implement a new or existing design. Use this skill when a design document exists (or when the user provides enough context to imply one) and the user wants working code. Triggers include: 'build this', 'implement the design', 'write the code', 'start coding', 'create the project', or any request to produce working software from a plan. Also use when the user wants to implement a specific step from a build sequence, add a feature described in a design, or refactor code to match an updated design. Do NOT use for writing specs (use spex-specify), creating designs (use spex-design), or writing tests (use spex-write-tests)."
---

# Implementing a Design

## Overview

Implementation is the lowest-precedence artifact in a spex project. The authority hierarchy is: **spec.md → design.md → tests → implementation**. Your job is to produce code that satisfies the tests, follows the design, and faithfully implements the behaviors described in the spec. When any of these conflict, defer upward: tests override implementation details, design overrides tests if there's a mismatch, and the spec rules all.

Implementation follows a **test-driven development (TDD)** discipline: for each piece of functionality, use the `spex-write-tests` skill to write a failing test first, then write the minimum code to make it pass, then clean up. The build must stay green throughout.

## Pre-Implementation Checklist

Before writing any code, verify you have what you need:

1. **Read the spec** (`spec.md` or equivalent). You need to understand the behaviors, not just the technical plan. The spec is the highest authority — if you find yourself uncertain about a requirement during implementation, go back to the spec.
2. **Read the design** (`design.md` or equivalent). Understand the component breakdown, data model, and build sequence. The design is your implementation guide, but if it contradicts the spec, the spec wins. Flag the conflict to the developer.
3. **Identify which step you're implementing.** Don't try to build everything at once. Follow the build sequence from the design. If the user asks for "the whole thing," build it step by step, keeping the build green at each step before moving on.
4. **Check for existing code.** If there's already a codebase, understand its structure, conventions, and patterns before adding to it. Match what's there — don't introduce a new style.
5. **Confirm the test infrastructure.** Before writing any production code, make sure the project can run tests. If there's no test runner configured, set one up as your very first action.

## Implementation Principles

### Test-Driven Development

Every piece of functionality follows the red-green-refactor cycle:

1. **Red:** Use the `spex-write-tests` skill to write a test for the next behavior you're about to implement. Run it. It should fail. If it passes, either the behavior already exists or the test is wrong.
2. **Green:** Write the minimum production code to make the test pass. Don't gold-plate — just make it pass.
3. **Refactor:** Clean up the code you just wrote (and the test if needed). Rename, extract, simplify. Run the tests again to confirm everything still passes.

The granularity of each cycle depends on the complexity. For simple functions or data models, one test per behavior is fine. For more complex flows, break them into smaller steps — test the core logic first, then the wiring.

Don't write a batch of tests all at once and then implement everything. The power of TDD is the tight feedback loop: one test, one behavior, verify, repeat.

**Don't get stuck in TDD loops.** If a test is hard to write, it may mean the design needs a different decomposition — flag it to the developer rather than churning. If you find yourself repeatedly rewriting the same test or going back and forth between test and implementation without converging, stop and reassess. Ask the developer for guidance if needed.

### Build Early, Build Often

The project must compile/build/parse cleanly at all times. Treat a broken build as a stop-the-world event.

**When to verify the build:**

- After setting up or modifying project structure
- After adding or changing any dependency
- After completing each red-green-refactor cycle
- After wiring components together
- Before moving to the next step in the build sequence

**What "verify the build" means depends on the stack:**

- Compiled languages: the project compiles without errors or warnings
- Interpreted languages: all files parse correctly, imports resolve, and the test suite runs (even if some tests are still pending)
- Frontend projects: the dev server starts, or the build command succeeds

If the build breaks, fix it immediately before doing anything else. Do not push forward hoping to "fix it later." A broken build means you've lost your safety net.

### Trace Everything to Scenarios

Every file, function, and module you create should be justifiable by pointing to a scenario in the spec that requires it. If you're writing code that no scenario motivates, stop and ask whether it's actually needed.

When writing code, use comments to link back to scenarios where the connection isn't obvious:

```python
# Scenario: "Applying an expired discount code"
# — expired codes return an error, they don't silently fail
if discount.expires_at < now:
    raise DiscountExpiredError(code=discount.code)
```

You don't need to annotate every line — just the points where a non-obvious design choice was made because of a specific scenario.

### Follow the Design's Vocabulary

The spec establishes the vocabulary. The design carries it into component and entity names. The code must use the same words. If the spec says "cart," the design says "CartService," and the code should have a `CartService` (or `cart_service`) — not `BasketManager` or `ShoppingBag`.

Naming consistency across spec → design → tests → code is one of the highest-leverage things you can do for long-term maintainability. This vocabulary is established at the spec level and flows downward — implementation doesn't get to rename things.

### Build in the Designed Order

The design's build sequence exists for a reason — each step builds on the last and produces something runnable. Follow it. Resist the urge to "just quickly add" something from a later step because it seems easy. Out-of-order building leads to untestable intermediate states and hidden dependencies.

If you're implementing a single step:

1. Build only what that step calls for
2. Verify the build is clean and all tests pass
3. Make sure it doesn't break anything from previous steps

### Keep Functions and Modules Small

A function should do one thing. A module should own one responsibility (matching the component breakdown in the design). If a function is getting long, it's probably doing too much — split it.

Heuristics:

- If a function has more than 3 levels of indentation, extract the inner logic
- If a module has more than ~300 lines, it probably owns too many responsibilities
- If you need a comment to explain _what_ a block does (not _why_), it should be its own function with a descriptive name

### Handle Errors Where Scenarios Demand Them

The spec's error scenarios tell you exactly where error handling matters. Implement those error paths thoroughly. For everything else, keep error handling simple and consistent — don't build elaborate error recovery for situations no scenario describes.

When an error scenario exists in the spec, the error path is a first-class feature, not an afterthought. Give it the same care as the happy path — and test it with the same rigor. Error scenarios should have their own tests written via `spex-write-tests`.

### Don't Build What's Not in the Design

If the design doesn't mention caching, don't add caching. If the design doesn't mention an admin panel, don't scaffold one. If you think something is missing from the design, flag it to the developer rather than silently adding it. The right response to a gap is to update the design (using `spex-design`), not to improvise during implementation. Remember: implementation is the lowest-precedence artifact. It doesn't get to introduce new requirements.

## Workflow

### Step 1: Set Up the Project and Test Infrastructure

If starting fresh, create the project structure that matches the design's component breakdown. Use the technology choices from the design. Set up the basics:

- Project scaffold (package.json, requirements.txt, go.mod, etc.)
- Directory structure reflecting the components
- Configuration for the chosen stack
- **Test runner and test framework** — this is not optional, it's first-priority
- A way to run the project locally

**Verify the build.** The empty project should build cleanly and the test runner should execute successfully (with zero tests).

If extending an existing project, skip the scaffold but confirm the test suite runs green before making any changes.

### Step 2: Build the Data Model (Test-First)

Start with the data model from the design. For each entity:

1. **Write tests first** using `spex-write-tests`: validation rules, required fields, state transitions, default values — whatever the spec's scenarios imply about the shape of the data.
2. **Implement** the schemas, migrations, type definitions, or whatever your stack uses.
3. **Verify the build and run all tests.** Everything should be green.

Validate the model against the scenarios: can you represent all the states described in the spec? If the spec mentions "an abandoned cart," make sure the cart model has a status that supports that — and a test that exercises it.

### Step 3: Build Components per the Sequence (Test-First)

Follow the build sequence step by step. For each step, apply TDD at the component level:

1. **Write tests for the core logic first** using `spex-write-tests` — services, business rules, the scenarios this step claims to satisfy. Run them; they should fail (red).
2. **Implement the core logic** to make the tests pass (green).
3. **Refactor** if needed. Run tests again.
4. **Verify the build is clean.**
5. **Write tests for the interfaces** (API routes, UI components, CLI commands) if applicable, then implement them.
6. **Wire components together** and run all tests — both the new ones and all previous ones.
7. **Verify the full build is clean** before moving to the next step.

Within each component, implement the happy path first, then the error cases. Each gets its own test-first cycle.

**Checkpoint rule:** Never move to the next step in the build sequence unless the build is clean and all tests pass. If something is broken, fix it now.

### Step 4: Verify Each Step

After completing each step in the build sequence, do a broader verification beyond just the unit tests:

- Run the full test suite (not just the tests you just wrote)
- Run the application and manually test the flow if practical
- Use a REPL to exercise the core logic if applicable

Don't wait until the end to verify. Each step should leave the project in a fully working, fully tested state.

### Step 5: Wire It End-to-End

Once all steps are built, verify the full flow works end-to-end. Walk through the spec's happy-path scenarios from start to finish. This is where integration issues surface — mismatched interfaces, missing data transformations, incorrect assumptions about ordering.

Write integration or end-to-end tests using `spex-write-tests` for the critical paths through the system. These are the tests that verify the scenarios the spec describes actually work when all the pieces are connected.

**Final build verification:** the build is clean, all tests (unit and integration) pass, and the application runs.

## Resolving Conflicts

During implementation you may discover that the design doesn't quite match the spec, or that a test encodes a behavior the design doesn't mention, or that existing code does something differently from what the design calls for.

Apply the precedence hierarchy: **spec → design → tests → implementation.**

- If the **design contradicts the spec**: follow the spec. Flag the discrepancy to the developer so the design can be updated (using `spex-design`).
- If a **test contradicts the design**: the design generally wins. Flag it so the test can be corrected (using `spex-write-tests`). But if the test is actually encoding a spec requirement the design missed, the spec wins — update the design.
- If **existing code contradicts the design**: follow the design. The existing code needs to change, not the design.
- If you're **unsure which is right**: stop and ask the developer. Don't guess and don't churn. This is a case where asking for guidance is the right move.

## Code Quality Standards

### Readability Over Cleverness

Write code that reads like a description of what it does. Avoid:

- Unnecessary abstractions that obscure the flow
- Clever one-liners that require mental unpacking
- Premature optimization that makes the logic harder to follow

Prefer:

- Descriptive names (even if they're long)
- Straightforward control flow
- Comments that explain _why_, not _what_

### Consistent Style

Match the style of the existing codebase. If there is no existing codebase, pick a conventional style for the language and stick to it. Consistency matters more than any particular style choice.

### Minimal Dependencies

Add external dependencies only when they provide significant value. For each dependency, ask:

- Does this save substantial implementation effort?
- Is it well-maintained and widely used?
- Does the design or a scenario require capabilities this provides?

If you can implement something in 20-50 lines of straightforward code, prefer that over adding a dependency.

**After adding any dependency, verify the build.**

### Configuration Over Hardcoding

Values that the design identifies as configurable (or that are obviously environment-specific) should be configurable from the start: port numbers, database URLs, API keys, feature flags, timeouts. Use environment variables or a config file.

Values that are genuinely constant and are motivated by the spec (like "delivery windows are 2-hour blocks") can be constants in code, ideally named and documented.

## Working with Existing Code

When implementing into an existing codebase:

1. **Read before writing.** Understand the patterns, naming conventions, and architectural decisions already in place.
2. **Run the existing test suite** before making any changes. It should pass. If it doesn't, flag this to the developer — don't start implementing on top of a broken test suite.
3. **Match existing patterns.** If the codebase uses repository pattern for data access, use it too. If it uses functional style, follow suit.
4. **Don't refactor while implementing.** If you notice something that should be refactored, note it but keep it separate from the feature implementation. Mixing refactoring with new features makes both harder to verify.
5. **Respect existing boundaries.** If the codebase has a clear separation between modules, don't reach across it. If the design suggests a boundary change, discuss it explicitly.

## Output

Implementation produces working, tested code files organized in the project structure. After implementing, make sure:

- The project builds without errors
- All tests pass (both new and pre-existing)
- The implemented scenarios work when manually verified
- Any setup steps are documented (README, comments, or inline notes)
- Files are saved to the appropriate project directory

If the implementation spans multiple files, present them in dependency order (foundations first, then the things that use them) so the structure is easy to follow.
