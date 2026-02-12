---
name: spex-specify
description: "Specify the software's behavior by listing scenarios written in given/when/then form, focused as much as possible on what an end user experiences. Use this skill when the user wants to define what a feature, system, or product should do before any code is written. Triggers include: requests to 'write a spec', 'define requirements', 'describe the behavior', 'list scenarios', 'write acceptance criteria', 'what should it do', or any task where the goal is to capture intended behavior rather than implement it. Also use when the user has a vague idea and needs help turning it into concrete, testable scenarios. Do NOT use for implementation, design, architecture, or test-writing — those are separate skills."
---

# Specifying Software Behavior

## Overview

A spec is a list of scenarios that describe what the software does from the perspective of someone using it. Each scenario follows the **given/when/then** structure. The spec is the single source of truth for what the software should do — design, implementation, and tests all trace back to it.

## What Makes a Good Spec

A spec is good when:
- A non-technical stakeholder can read it and confirm "yes, that's what I want"
- A developer can read it and know exactly what to build without guessing
- A tester can read it and derive test cases directly from the scenarios
- It says **what** happens, not **how** it's built

A spec is bad when:
- It describes internal mechanisms, database schemas, or API shapes
- Scenarios are so vague they could mean multiple things
- It mixes behavior with implementation ("the React component renders a div...")
- It's a wall of prose instead of discrete, verifiable scenarios

## Spec Structure

Every spec has three parts:

### 1. Preamble

A short block (3-8 sentences) that establishes context. It answers:
- What is this thing? (one sentence)
- Who uses it and why? (one or two sentences)
- What are the key constraints or boundaries? (if any)

Keep it brief. The scenarios do the heavy lifting.

```markdown
## Preamble

FreshCart is a grocery delivery app. Customers browse a catalog, add items
to a cart, and check out for scheduled delivery. Orders can only be placed
within delivery zones, and delivery windows are 2-hour blocks from 8am-8pm.
```

### 2. Scenarios

The heart of the spec. Each scenario is a single, concrete example of behavior. Use this format:

```markdown
### Scenario: <short descriptive name>

**Given** <a starting state or precondition>
**When** <the user does something or an event occurs>
**Then** <the observable outcome>
```

Rules for scenarios:

- **One behavior per scenario.** If you're writing "and then also..." you need a second scenario.
- **Use concrete examples, not abstract rules.** Write `Given the cart contains "Organic Bananas" at $3.49` not `Given the cart contains an item with a price`. Concrete examples are easier to verify and harder to misinterpret.
- **Stay at the user's level.** The user sees a confirmation message, not a 201 HTTP response. The user's balance updates, not a database row.
- **Given = state, When = action, Then = outcome.** Don't sneak actions into Given or state into Then.
- **Name scenarios descriptively.** `Adding an out-of-stock item` beats `Scenario 7`. The name should tell you what's interesting about this scenario without reading the body.

Multiple Given/When/Then clauses are fine when they genuinely belong to one behavior:

```markdown
### Scenario: Applying a percentage discount code

**Given** the cart contains "Sourdough Bread" at $6.00
**And given** the cart contains "Oat Milk" at $4.50
**When** the customer applies discount code "SAVE20"
**Then** the cart total shows $8.40
**And then** the discount line reads "SAVE20: -$2.10"
```

### 3. Open Questions

Capture anything unresolved. This section prevents the spec from silently papering over ambiguity. Every question should be specific enough to have a concrete answer.

```markdown
## Open Questions

- What happens if a delivery window fills up while the customer is checking out?
- Should expired discount codes show an error message, or silently do nothing?
- Is there a maximum cart size?
```

## Workflow

### Step 1: Understand the Intent

Before writing anything, make sure you understand what the user wants to specify. Ask clarifying questions if needed, but don't over-interview — it's often better to draft scenarios and let the user react to them than to ask 15 questions upfront.

Key things to clarify early:
- Who are the users? (There may be more than one type.)
- What's the core action or workflow?
- Are there any hard constraints? (regulatory, performance, platform)

### Step 2: Identify Scenario Groups

Organize scenarios into logical groups. Common groupings:
- By workflow stage (browsing → cart → checkout → delivery)
- By user role (customer, admin, driver)
- By feature area (search, payments, notifications)

Within each group, cover:
1. **The happy path** — the normal, expected flow
2. **Edge cases** — boundaries, limits, empty states
3. **Error cases** — what goes wrong and what the user sees
4. **Permissions/access** — who can and can't do what

A useful heuristic: for every happy-path scenario, ask "what if it fails?" and "what if the input is weird?" Those usually produce 1-3 more scenarios.

### Step 3: Write Scenarios

Write all scenarios using the format above. Aim for:
- **Concrete over abstract.** Real example values, real names, real numbers.
- **Independent scenarios.** Each should stand on its own. Don't write "Given the outcome of Scenario 3..."
- **No implementation leakage.** If you catch yourself writing "the API returns" or "the database contains" or "the component renders", rewrite it from the user's perspective.

### Step 4: Review and Tighten

After drafting, review with these checks:
- **Duplicate check:** Do any two scenarios test the same thing with trivially different inputs? Merge or cut.
- **Gap check:** Walk through the workflow end-to-end. Are there decision points with no scenario?
- **Clarity check:** Could someone misinterpret any scenario in a way that leads to wrong behavior? If yes, make it more specific.
- **Scope check:** Has the spec silently expanded beyond what was asked for? Flag anything that feels like scope creep and ask the user.

### Step 5: Document Open Questions

Be honest about what's unresolved. It's far better to have 5 open questions in the spec than to have 5 silent assumptions baked into the scenarios. If you had to make an assumption to write a scenario, call it out.

## Principles

**Favor many small scenarios over few large ones.** A scenario with 8 "And then" clauses is really 4 scenarios pretending to be one. Split them.

**Scenarios are examples, not rules.** `Given the customer has ordered 3 times this month / When they place a 4th order / Then they receive a "Loyal Customer" badge` is better than a rule that says "customers who order 4+ times per month get a badge." The example is unambiguous; the rule invites questions (calendar month? rolling 30 days? what if they cancel one?).

**Don't spec what you don't need.** If the user asked you to spec the checkout flow, don't also spec the account registration flow "for completeness." Stay focused. You can always add more scenarios later.

**Name things from the user's vocabulary.** If the user says "cart", don't call it "basket" in the spec. If the user says "gig", don't call it "job." Matching vocabulary prevents a whole class of miscommunication.

**Specs evolve.** The first draft is never the last. Write it, get feedback, revise. The structure here is designed to make that easy — you can add, remove, or rewrite individual scenarios without disturbing the rest.

## Output Format

Produce the spec as a single Markdown document. Use this skeleton:

```markdown
# <Name of the Thing Being Specified>

## Preamble

<3-8 sentences of context>

## Scenarios

### <Group Name> (optional grouping header)

#### Scenario: <descriptive name>

**Given** ...
**When** ...
**Then** ...

#### Scenario: <descriptive name>

**Given** ...
**When** ...
**Then** ...

### <Next Group>

...

## Open Questions

- ...
- ...
```

Save the spec as `spec.md` in the project root (or wherever the user indicates). This file is the canonical artifact — design, implementation, and tests should all reference it.
