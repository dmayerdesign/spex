---
name: spex-implement
description: "Implement a new or existing design. Use this skill when a design document exists (or when the user provides enough context to imply one) and the user wants working code. Triggers include: 'build this', 'implement the design', 'write the code', 'start coding', 'create the project', or any request to produce working software from a plan. Also use when the user wants to implement a specific step from a build sequence, add a feature described in a design, or refactor code to match an updated design. Do NOT use for writing specs (use spex-specify), creating designs (use spex-design), or writing tests (use spex-write-tests)."
---

# Implementing a Design

## Overview

Implementation turns a design document into working code. The design tells you what components to build, what data model to use, and in what order to build them. Your job is to translate that into code that is correct, readable, and traceable back to the scenarios in the spec.

## Pre-Implementation Checklist

Before writing any code, verify you have what you need:

1. **Read the spec** (`spec.md` or equivalent). You need to understand the behaviors, not just the technical plan.
2. **Read the design** (`design.md` or equivalent). Understand the component breakdown, data model, and build sequence.
3. **Identify which step you're implementing.** Don't try to build everything at once. Follow the build sequence from the design. If the user asks for "the whole thing," build it step by step, committing each step to a working state before moving on.
4. **Check for existing code.** If there's already a codebase, understand its structure, conventions, and patterns before adding to it. Match what's there — don't introduce a new style.

## Implementation Principles

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

The spec establishes the vocabulary. The design carries it into component and entity names. The code should use the same words. If the spec says "cart," the design says "CartService," and the code should have a `CartService` (or `cart_service`) — not `BasketManager` or `ShoppingBag`.

Naming consistency across spec → design → code → tests is one of the highest-leverage things you can do for long-term maintainability.

### Build in the Designed Order

The design's build sequence exists for a reason — each step builds on the last and produces something runnable. Follow it. Resist the urge to "just quickly add" something from a later step because it seems easy. Out-of-order building leads to untestable intermediate states and hidden dependencies.

If you're implementing a single step:
1. Build only what that step calls for
2. Verify it works (manually or with a quick smoke test)
3. Make sure it doesn't break anything from previous steps

### Keep Functions and Modules Small

A function should do one thing. A module should own one responsibility (matching the component breakdown in the design). If a function is getting long, it's probably doing too much — split it.

Heuristics:
- If a function has more than 3 levels of indentation, extract the inner logic
- If a module has more than ~300 lines, it probably owns too many responsibilities
- If you need a comment to explain *what* a block does (not *why*), it should be its own function with a descriptive name

### Handle Errors Where Scenarios Demand Them

The spec's error scenarios tell you exactly where error handling matters. Implement those error paths thoroughly. For everything else, keep error handling simple and consistent — don't build elaborate error recovery for situations no scenario describes.

When an error scenario exists in the spec, the error path is a first-class feature, not an afterthought. Give it the same care as the happy path.

### Don't Build What's Not in the Design

If the design doesn't mention caching, don't add caching. If the design doesn't mention an admin panel, don't scaffold one. If you think something is missing from the design, flag it to the user rather than silently adding it. The right response to a gap is to update the design, not to improvise during implementation.

## Workflow

### Step 1: Set Up the Project

If starting fresh, create the project structure that matches the design's component breakdown. Use the technology choices from the design. Set up the basics:
- Project scaffold (package.json, requirements.txt, go.mod, etc.)
- Directory structure reflecting the components
- Configuration for the chosen stack
- A way to run the project locally

If extending an existing project, skip this step and work within the existing structure.

### Step 2: Build the Data Model

Start with the data model from the design. Create the schemas, migrations, type definitions, or whatever your stack uses to define entities. This is the foundation everything else builds on.

Validate the model against the scenarios: can you represent all the states described in the spec? If the spec mentions "an abandoned cart," make sure the cart model has a status that supports that.

### Step 3: Build Components per the Sequence

Follow the build sequence step by step. For each step:

1. **Build the core logic first** (services, business rules)
2. **Then the interfaces** (API routes, UI components, CLI commands)
3. **Then wire them together**

Within each component, implement the happy path first, then the error cases. This matches how the spec is usually organized and gives you a working flow to verify before handling edge cases.

### Step 4: Verify Each Step

After completing each step in the build sequence, verify that the scenarios it claims to satisfy actually work. This can be:
- Running the application and manually testing the flow
- Writing a quick smoke test
- Using a REPL to exercise the core logic

Don't wait until the end to verify. Each step should leave the project in a working state.

### Step 5: Wire It End-to-End

Once all steps are built, verify the full flow works end-to-end. Walk through the spec's happy-path scenarios from start to finish. This is where integration issues surface — mismatched interfaces, missing data transformations, incorrect assumptions about ordering.

## Code Quality Standards

### Readability Over Cleverness

Write code that reads like a description of what it does. Avoid:
- Unnecessary abstractions that obscure the flow
- Clever one-liners that require mental unpacking
- Premature optimization that makes the logic harder to follow

Prefer:
- Descriptive names (even if they're long)
- Straightforward control flow
- Comments that explain *why*, not *what*

### Consistent Style

Match the style of the existing codebase. If there is no existing codebase, pick a conventional style for the language and stick to it. Consistency matters more than any particular style choice.

### Minimal Dependencies

Add external dependencies only when they provide significant value. For each dependency, ask:
- Does this save substantial implementation effort?
- Is it well-maintained and widely used?
- Does the design or a scenario require capabilities this provides?

If you can implement something in 20-50 lines of straightforward code, prefer that over adding a dependency.

### Configuration Over Hardcoding

Values that the design identifies as configurable (or that are obviously environment-specific) should be configurable from the start: port numbers, database URLs, API keys, feature flags, timeouts. Use environment variables or a config file.

Values that are genuinely constant and are motivated by the spec (like "delivery windows are 2-hour blocks") can be constants in code, ideally named and documented.

## Working with Existing Code

When implementing into an existing codebase:

1. **Read before writing.** Understand the patterns, naming conventions, and architectural decisions already in place.
2. **Match existing patterns.** If the codebase uses repository pattern for data access, use it too. If it uses functional style, follow suit.
3. **Don't refactor while implementing.** If you notice something that should be refactored, note it but keep it separate from the feature implementation. Mixing refactoring with new features makes both harder to verify.
4. **Respect existing boundaries.** If the codebase has a clear separation between modules, don't reach across it. If the design suggests a boundary change, discuss it explicitly.

## Output

Implementation produces working code files organized in the project structure. After implementing, make sure:
- The project runs without errors
- The implemented scenarios work when manually verified
- Any setup steps are documented (README, comments, or inline notes)
- Files are saved to the appropriate project directory

If the implementation spans multiple files, present them in dependency order (foundations first, then the things that use them) so the structure is easy to follow.
