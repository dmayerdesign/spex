---
description: "Create or update a design document (design.md) from the spec"
tools:
    - vscode
    - execute
    - read
    - agent
    - edit
    - search
    - web
    - todo
---

Use the spex-design skill in [SKILL.md](../skills/spex-design/SKILL.md).

Read [AGENTS.md](../../AGENTS.md) to understand the spec-as-source workflow and precedence hierarchy.

Read spec.md first — the spec is the source of truth. If there is no spec, stop and tell the user to write one first (using /spec).

If a design.md already exists, read it. You are updating, not replacing, unless the user says otherwise.

The user may specify a particular area or concern to focus on. Otherwise, design the full spec.

The design must include: entity model, component breakdown traced to scenarios, technology choices with rationale, interface definitions detailed enough to write tests against, a build sequence where every step is buildable and testable, and a section for decisions and open questions.

If the spec is ambiguous or incomplete, note it as an open question rather than guessing silently.

Save the result to design.md (or the appropriate design file if the project uses multiple).
