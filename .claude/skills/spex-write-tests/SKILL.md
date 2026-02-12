---
name: spex-write-tests
description: "Add concise but thorough test coverage where needed, ensuring each test links back to one or more scenarios in the spec. Use this skill when the user wants to add, improve, or review tests for code that has a behavioral spec. Triggers include: 'write tests', 'add test coverage', 'test this', 'are there enough tests', 'cover these scenarios', or any request to verify that implemented code satisfies the spec's scenarios. Also use when the user asks to check which scenarios are untested or wants to understand the relationship between tests and spec. Do NOT use for writing specs (use spex-specify), creating designs (use spex-design), or implementing features (use spex-implement)."
---

# Writing Tests

## Overview

Tests sit at the third level of the precedence hierarchy: **spec.md → design.md → tests → implementation**. Tests translate the spec's scenarios into executable verification. Each test should trace to one or more scenarios in the spec — if a test doesn't correspond to a scenario, question whether it's needed. If a scenario doesn't have a test, that's a gap.

In a spex project, tests are written _before_ the code they verify. The implementation skill (`spex-implement`) follows a TDD workflow: it calls this skill to write a failing test, then writes code to make it pass. Tests are the mechanism that connects the spec's behavioral promises to the actual code.

## Before You Start

1. **Read the spec.** Every scenario is a candidate for one or more tests. You need to understand the behaviors, not just the function signatures.
2. **Read the design.** The component breakdown and interface definitions tell you _where_ to test and _what the boundaries are_. The design's interfaces are your test surfaces.
3. **Check what's already tested.** If there's an existing test suite, understand its structure, conventions, and coverage before adding to it. Match the existing style.
4. **Confirm the test infrastructure.** The test runner should already be set up (this happens in Step 1 of `spex-implement`). If it's not, set it up before writing any tests.

## Principles

### Every Test Traces to a Scenario

Name tests after the scenario they verify. If the spec has "Applying an expired discount code," the test should be named something like `test_applying_expired_discount_code_returns_error`. A reader should be able to map any test back to the spec without guessing.

If you find yourself writing a test that doesn't correspond to any scenario — a test for an internal helper, a defensive check against an edge case the spec doesn't mention — pause. It might be legitimate (infrastructure glue needs testing too), but it should be the exception, not the pattern. Most tests should be traceable.

### Test Behavior, Not Implementation

Tests verify _what_ happens, not _how_ it happens internally. Test inputs and outputs through the component's public interface, as defined in the design. Don't reach into private state, mock internal collaborators unnecessarily, or assert on implementation details that could change without affecting behavior.

A good test breaks only when the behavior changes. A brittle test breaks when the code is refactored but the behavior is identical. Aim for the former.

### One Behavior Per Test

Each test should verify one scenario or one specific aspect of a scenario. If a test has multiple unrelated assertions, split it. When a test fails, the name and the failure should tell you exactly which behavior is broken.

Complex scenarios can span multiple tests — "Completing a checkout" might need separate tests for inventory deduction, order creation, and confirmation email. But each test should be independently meaningful.

### Happy Path First, Then Errors

Mirror the spec's structure: write tests for the happy-path scenarios first, then the error scenarios. This matches the order the implementer will work in during TDD — get the core behavior working, then handle the ways it can fail.

Error scenarios in the spec are first-class requirements. Each one gets its own test with the same care as the happy path. "Applying an expired discount code" is not a lesser test than "Applying a valid discount code."

### Keep Tests Fast and Independent

Tests should run quickly and in any order. No test should depend on another test's side effects. Each test sets up its own state (the "Given"), performs the action (the "When"), and checks the outcome (the "Then") — mirroring the scenario format from the spec.

If setup is complex, extract shared helpers or fixtures, but keep the test body readable. A reader should understand what a test does without chasing through layers of abstraction.

## Workflow

### Step 1: Map Scenarios to Tests

Read the spec and list every scenario. For each one, determine:

- **What component does this test exercise?** The design's component breakdown tells you this.
- **What's the test surface?** Use the interface defined in the design — public function, API endpoint, CLI command.
- **How many tests does this scenario need?** Usually one, but complex scenarios with multiple distinct outcomes may need several.

This mapping is the test plan. If you notice scenarios that the design's components can't exercise through their interfaces, flag it — the design may need adjustment (using `spex-design`).

### Step 2: Write Tests for the Current Build Step

During TDD, you're writing tests for the build step currently being implemented. Don't write tests for steps that haven't been built yet — the code they'd test doesn't exist, and the interfaces may shift.

For the current step, write tests in this order:

1. **Core logic / happy path** — the main behavior the step adds.
2. **Variations** — different inputs, boundary values, alternate flows described in the spec.
3. **Error cases** — the error scenarios from the spec that this step's components handle.

Run each test after writing it. It should fail (red). Hand off to `spex-implement` to make it pass.

### Step 3: Verify Coverage Against the Spec

After a build step is complete (tests passing, build green), check: are all the scenarios this step claims to satisfy actually tested? Walk through the spec's scenarios for this step and confirm each has a corresponding test.

If the full implementation is complete, do a final coverage check across the entire spec. Every scenario should have at least one test. Document any gaps — either write the missing tests or flag scenarios that turned out to be untestable and need spec clarification (using `spex-specify`).

## Resolving Conflicts

Apply the precedence hierarchy: **spec → design → tests → implementation.**

- **Test contradicts the spec:** fix the test. The spec is the authority on behavior.
- **Test contradicts the design's interface:** usually fix the test to match the design. But if the test is encoding a spec requirement the design missed, flag it so the design can be updated (using `spex-design`).
- **Test is hard to write against the current design:** this often means the component boundaries are wrong — the design needs adjustment, not a hacky test. Flag it rather than working around it.
- **Unsure what the expected behavior is:** go back to the spec. If the spec is ambiguous, ask the developer. Don't encode a guess as a test — a wrong test is worse than a missing one.

## Output

Test files organized to match the project's conventions, containing tests that are named after spec scenarios, test behavior through public interfaces, run fast and independently, and collectively cover every scenario in the spec. After writing tests, verify the test suite runs (even if new tests fail because the code isn't written yet — that's expected in TDD).
