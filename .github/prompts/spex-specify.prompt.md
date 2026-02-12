---
description: "Write or update a behavioral spec (spec.md) for a feature"
tools: ["search/codebase"]
---

Use the spex-specify skill in [SKILL.md](../../.github/skills/spex-specify/SKILL.md).

Read [AGENTS.md](../../AGENTS.md) to understand the spec-as-source workflow and precedence hierarchy.

If a spec.md already exists, read it first. You are updating, not starting from scratch unless the file is empty or missing.

The user may provide a feature description or area to specify as additional input. If not, ask what feature or behavior they want to specify. Ask clarifying questions if the intent is unclear, but don't stall — get scenarios written.

Write scenarios in given/when/then form. Happy paths first, then error paths. Use concrete examples, not abstractions. Keep the vocabulary consistent — the names you choose here will flow through design, tests, and code.

Save the result to spec.md (or the appropriate spec file if the project uses multiple).
