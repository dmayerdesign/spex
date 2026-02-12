---
description: "Write or update tests that verify spec scenarios"
tools:
    - vscode
    - execute
    - read
    - agent
    - edit
    - search
    - todo
---

Use the spex-write-tests skill in [SKILL.md](../skills/spex-write-tests/SKILL.md).

Read [AGENTS.md](../../AGENTS.md) to understand the spec-as-source workflow and precedence hierarchy.

Read spec.md and design.md first. Tests trace to scenarios in the spec and test through interfaces defined in the design.

If there are existing tests, read them to understand the conventions, framework, and structure before adding new ones.

The user may specify a particular scenario, component, or build step to test. Otherwise, identify untested scenarios and fill the gaps.

Name each test after the scenario it verifies. Test behavior through public interfaces, not implementation details. One behavior per test. Happy paths first, then error paths.

Run the tests after writing them. In a TDD context, new tests should fail (red) — that's expected. Confirm the test suite itself executes without infrastructure errors.
