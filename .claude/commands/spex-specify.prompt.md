---
description: Write or update a behavioral spec (spec.md) for a feature
# argument-hint: [feature description or area to specify]
---

Use the `spex-specify` skill.

Read the project's AGENTS.md to understand the spec-as-source workflow and precedence hierarchy.

If a spec.md already exists, read it first. You are updating, not starting from scratch unless the file is empty or missing.

If the user provided arguments:

$ARGUMENTS

Use that as the starting point — ask clarifying questions if the intent is unclear, but don't stall. Get scenarios written.

If no arguments were provided, ask the user what feature or behavior they want to specify.

Write scenarios in given/when/then form. Happy paths first, then error paths. Use concrete examples, not abstractions. Keep the vocabulary consistent — the names you choose here will flow through design, tests, and code.

Save the result to spec.md (or the appropriate spec file if the project uses multiple).
