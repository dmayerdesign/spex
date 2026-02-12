---
name: spex-specify
description: "Specify the software's behavior by listing scenarios written in given/when/then form, focused as much as possible on what an end user experiences. Use this skill when the user wants to define what a feature, system, or product should do before any code is written. Triggers include: requests to 'write a spec', 'define requirements', 'describe the behavior', 'list scenarios', 'write acceptance criteria', 'what should it do', or any task where the goal is to capture intended behavior rather than implement it. Also use when the user has a vague idea and needs help turning it into concrete, testable scenarios. Do NOT use for implementation, design, architecture, or test-writing — those are separate skills."
---

# Writing a Spec

## Overview

The spec is the highest-precedence artifact in a spex project. The authority hierarchy is: **spec.md → design.md → tests → implementation**. Everything downstream — the design, the tests, the code — exists to serve what the spec describes. If any of them conflict with the spec, the spec wins.

A spec captures _what the software does_ from the perspective of someone using it. It is not a technical document. It doesn't name databases, frameworks, or components. It describes behaviors: what happens when a user does something, what they see, what changes, what goes wrong. The format is scenarios in **given/when/then** form.

## Before You Start

1. **Understand the user's intent.** What problem is the software solving? Who is using it? If the developer has a rough idea but not a clear picture, help them sharpen it before writing scenarios. Ask questions.
2. **Identify the actors.** Who interacts with the system? Often it's just "the user," but there may be distinct roles (admin, guest, API consumer). Name them using the developer's language.
3. **Identify the core nouns.** What are the things in the system? Carts, orders, documents, tasks, accounts. These become the vocabulary that flows through the entire project — design, tests, and code will all use these exact words.
4. **Don't think about implementation.** The spec is deliberately technology-free. No databases, no endpoints, no frameworks. If you catch yourself writing "the API returns" or "the database stores," back up and describe what the user experiences instead.

## Scenario Format

Every behavior is captured as a scenario in given/when/then form:

```
Scenario: Adding an item to an empty cart
  Given the user has an empty cart
  When they add a product to the cart
  Then the cart contains one item
  And the cart total reflects the product's price
```

**Given** establishes the starting state. **When** describes the action. **Then** describes the observable outcome. Each part should describe something a user could see or verify — not an internal system state.

Guidelines for good scenarios:

- **One behavior per scenario.** If a scenario has five "Then" clauses covering unrelated outcomes, it's probably multiple scenarios.
- **Name scenarios descriptively.** The name should tell you what's being tested without reading the body. "Adding an item to an empty cart" is good. "Cart test 1" is not.
- **Use concrete examples, not abstractions.** "Given the discount code is expired" is better than "given the discount code is in an invalid state." Specific cases are easier to verify and harder to misinterpret.
- **Cover the sad paths.** For every happy-path scenario, ask: what could go wrong? What if the input is invalid? What if the thing doesn't exist? What if they don't have permission? Error scenarios are requirements too — they'll get their own tests and their own code paths.
- **Stay at the user's level.** Describe what the user experiences, not what the system does internally. "Then the user sees an error message" rather than "then the system logs an exception and returns a 422."

## Workflow

### Step 1: Start with the Happy Path

Write scenarios for the core thing the software does when everything goes right. This is usually obvious from the developer's description. If it's a todo app, the happy path is creating, completing, and listing tasks. If it's a checkout flow, it's adding items, applying discounts, and placing an order.

Get the happy path scenarios written and reviewed before branching into edge cases. They establish the vocabulary and the shape of the system.

### Step 2: Walk the Error Paths

For each happy-path scenario, ask what can go wrong:

- Invalid input (missing fields, wrong types, out-of-range values)
- Missing or nonexistent things (item not found, user doesn't exist)
- State conflicts (already completed, expired, duplicate)
- Permission failures (not logged in, wrong role)
- Boundary conditions (empty list, maximum exceeded, zero quantity)

Each error condition that produces a distinct user-visible outcome gets its own scenario. Don't group them — "various validation errors" is not a scenario.

### Step 3: Find the Implicit Scenarios

Some behaviors aren't obvious from the developer's initial description but emerge when you think carefully:

- **What happens on revisit?** If a user leaves and comes back, what do they see? Is state preserved?
- **What about sequences?** Does the order of actions matter? Can you do step 3 before step 2?
- **What about multiples?** What if there are zero items? One? A hundred?
- **What about concurrency?** Can two users affect the same thing? Does it matter?

Don't invent requirements — but do ask the developer about gaps. If you spot an ambiguity, write it as an open question rather than silently resolving it.

### Step 4: Organize by Feature Area

Group scenarios under headings that match the natural divisions of the system. For a shopping app, that might be: Cart, Checkout, Discounts, Order History. These groupings often preview the component structure the design will use, but don't force architectural thinking — group by user-facing feature, not by technical boundary.

Within each group, put happy-path scenarios first, then error scenarios. This makes the spec easy to scan.

### Step 5: Review for Completeness and Consistency

Before finalizing, check:

- **Does every noun appear in at least one scenario?** If the developer mentioned "notifications" but no scenario exercises them, either add scenarios or confirm they're out of scope.
- **Are the scenarios consistent?** If one scenario says the cart empties after checkout and another implies items persist, there's a contradiction. Resolve it.
- **Is the vocabulary stable?** Don't call it "cart" in one scenario and "basket" in another. Pick one name and use it everywhere. This vocabulary will propagate through the entire project.
- **Are the scenarios testable?** Every "Then" clause should be something an automated test could verify. "Then the user has a good experience" is not testable. "Then the dashboard shows the three most recent orders" is.

## Principles

### The Spec Owns the Vocabulary

The names you use in the spec — for actors, entities, actions, and states — become the shared language of the project. The design (via `spex-design`) will carry them into component names. Tests (via `spex-write-tests`) will use them in test names and assertions. Code (via `spex-implement`) will use them in classes, functions, and variables. Choose clear, specific names and use them consistently.

### Scenarios Are Requirements

Every scenario is a promise about how the software behaves. The design must account for it. The tests must verify it. The code must implement it. Don't write scenarios casually — if it's in the spec, it's required.

Conversely, if a behavior isn't in the spec, it's not required. The design shouldn't add features the spec doesn't describe, and the implementation shouldn't either. If new behavior is needed, it starts here, as a new scenario.

### Stay Technology-Free

The spec should be implementable in any language, framework, or architecture. Don't mention REST, SQL, React, or Redis. Describe the behavior, not the mechanism. The one exception is when the technology _is_ the requirement — "the CLI accepts a --verbose flag" is a spec-level concern because it's part of the user's experience.

### Prefer Precision Over Brevity

A scenario that's too specific is easy to generalize later. A scenario that's too vague will be interpreted differently by every reader. When in doubt, be concrete. "Given the user has three items in their cart totaling $45.00" is better than "given the user has items in their cart."

## Updating an Existing Spec

When requirements change, update the spec first — before touching the design, tests, or code. Add, modify, or remove scenarios as needed. Then flag the changes so downstream artifacts can be updated: the design (using `spex-design`), the tests (using `spex-write-tests`), and the implementation (using `spex-implement`).

Don't leave stale scenarios in the spec. A scenario that no longer reflects intended behavior is worse than a missing scenario — it actively misleads.

## Output

A `spec.md` containing: a brief description of what the software does and who it's for, followed by scenarios grouped by feature area, in given/when/then form. Happy paths first, then error paths. Concrete, testable, technology-free, and using a consistent vocabulary throughout.
