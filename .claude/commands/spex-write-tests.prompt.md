---
description: Write or update tests that verify spec scenarios
# argument-hint: [optional: specific scenario, component, or build step to test]
---

Use the `spex-write-tests` skill.

Read the project's AGENTS.md to understand the spec-as-source workflow and precedence hierarchy.

Read spec.md and design.md first. Tests trace to scenarios in the spec and test through interfaces defined in the design.

If there are existing tests, read them to understand the conventions, framework, and structure before adding new ones.

If the user provided arguments:

$ARGUMENTS

Focus on writing tests for that scenario, component, or build step. Otherwise, identify untested scenarios and fill the gaps.

Name each test after the scenario it verifies. Test behavior through public interfaces, not implementation details. One behavior per test. Happy paths first, then error paths.

Run the tests after writing them. In a TDD context, new tests should fail (red) — that's expected. Confirm the test suite itself executes without infrastructure errors.
