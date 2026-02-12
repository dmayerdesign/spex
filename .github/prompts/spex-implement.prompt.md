---
description: "Implement the next build step from the design using TDD"
tools: ["search/codebase"]
---

Use the spex-implement skill in [SKILL.md](../../.github/skills/spex-implement/SKILL.md).

Read [AGENTS.md](../../AGENTS.md) to understand the spec-as-source workflow and precedence hierarchy.

Read spec.md and design.md first. The spec is the authority on behavior; the design is your implementation guide.

If there is existing code, read it to understand patterns and conventions before adding to it. Run the existing test suite — it must pass before you start.

The user may specify a build step number or component to implement. Otherwise, identify the next unfinished step in the design's build sequence.

Follow TDD for each piece of functionality:

1. Write a failing test (use the spex-write-tests skill)
2. Write the minimum code to make it pass
3. Refactor
4. Verify the build is clean and all tests pass

Do not move to the next build step until the current step builds cleanly and all tests pass. If something contradicts the spec or design, flag it — don't guess.
