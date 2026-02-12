---
description: Create or update a design document (design.md) from the spec
# argument-hint: [optional: specific area or concern to focus on]
---

Use the `spex-design` skill.

Read the project's AGENTS.md to understand the spec-as-source workflow and precedence hierarchy.

Read spec.md first — the spec is the source of truth. If there is no spec, stop and tell the user to write one first (using /spec).

If a design.md already exists, read it. You are updating, not replacing, unless the user says otherwise.

If the user provided arguments:

$ARGUMENTS

Focus the design work on that area. Otherwise, design the full spec.

The design must include: entity model, component breakdown traced to scenarios, technology choices with rationale, interface definitions detailed enough to write tests against, a build sequence where every step is buildable and testable, and a section for decisions and open questions.

If the spec is ambiguous or incomplete, note it as an open question rather than guessing silently.

Save the result to design.md (or the appropriate design file if the project uses multiple).
