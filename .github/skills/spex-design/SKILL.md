---
name: spex-design
description: "Create a detailed plan for implementing the spec. Use this skill when a behavioral spec (scenarios in given/when/then form) already exists and the user wants to plan how to build it before writing code. Triggers include: 'design this', 'plan the implementation', 'how should we build this', 'create an architecture', 'what's the technical approach', or any request to bridge the gap between a spec and code. Also use when the user has a spec and asks about technology choices, component structure, data modeling, or sequencing of work. Do NOT use for writing the spec itself (use spex-specify), writing code (use spex-implement), or writing tests (use spex-write-tests)."
---

# Designing an Implementation Plan

## Overview

A design takes a behavioral spec (scenarios in given/when/then form) and produces a concrete plan for building it. The design answers "how will we build this?" while the spec answers "what should it do?" The design is the bridge between the two — detailed enough that implementation becomes a matter of execution rather than invention.

## What Makes a Good Design

A design is good when:
- Every scenario in the spec has a clear path to being satisfied
- A developer can pick up any section and start building without needing to make architectural decisions
- Technology choices are justified, not just stated
- The sequencing makes sense — earlier pieces don't depend on later ones
- It's honest about tradeoffs and what it's punting on

A design is bad when:
- It restates the spec in technical language without adding anything
- It makes technology choices without explaining why
- It describes a perfect end-state but gives no clue how to get there incrementally
- It's so abstract that two developers would build completely different things from it
- It ignores the constraints implied by the scenarios (performance, concurrency, etc.)

## Design Structure

### 1. Summary

2-4 sentences. What are we building, what's the core technical approach, and what's the most important architectural decision?

```markdown
## Summary

FreshCart is a web application with a React frontend and a Node.js API server
backed by PostgreSQL. The core architectural decision is to treat the cart as
a server-side resource (not purely client-side) so that pricing, inventory
checks, and discount logic are authoritative. Delivery window availability is
managed through a slot-reservation system with optimistic locking to handle
concurrent checkouts.
```

### 2. Scenario-to-Component Map

This is the most important section. For every scenario (or group of closely related scenarios) in the spec, name the components involved and sketch the interaction. This creates explicit traceability from behavior to implementation.

```markdown
## Scenario Map

### "Adding an in-stock item to the cart"
- **UI**: ProductCard emits `add-to-cart` event → CartSidebar updates count
- **API**: `POST /cart/items` — validates inventory, adds item, returns updated cart
- **Data**: `cart_items` table (cart_id, product_id, quantity, unit_price)
- **Key logic**: Inventory check is read-at-request-time, not reserved until checkout

### "Applying a percentage discount code"
- **UI**: CartSidebar discount input → calls API → displays discount line
- **API**: `POST /cart/discount` — validates code, calculates discount, returns updated totals
- **Data**: `discount_codes` table (code, type, value, expires_at, usage_limit)
- **Key logic**: Discount calculated server-side; cart total is never computed client-side
```

You don't need to list every field or every line of code. The goal is that someone reading the scenario can immediately see where in the codebase to look and what the data flow is.

### 3. Data Model

Define the core entities, their relationships, and the key fields. Focus on what's needed to satisfy the scenarios — don't model things that no scenario references.

```markdown
## Data Model

### Cart
- id, customer_id, status (active | checked_out | abandoned), created_at
- A customer has at most one active cart at a time

### CartItem
- cart_id, product_id, quantity, unit_price_at_addition
- unit_price is captured at addition time (price may change later)

### DiscountCode
- code (unique), type (percentage | fixed), value, expires_at, usage_limit, times_used
```

Guidelines:
- Name entities using the vocabulary from the spec, not generic database jargon
- Call out important constraints inline ("at most one active cart")
- Note any denormalization and why it exists ("price captured at addition time because...")
- If using a non-relational store, describe the document/key structure instead

### 4. Component Breakdown

List the major components/modules and what each one is responsible for. This isn't a class diagram — it's a responsibility map.

```markdown
## Components

### CartService
Owns all cart mutation logic: add item, remove item, apply discount, calculate totals.
All pricing logic lives here — the frontend never computes prices.

### InventoryService
Checks and reserves stock. Read-time checks for cart additions; hard reservation at checkout.
Exposes: `checkAvailability(productId, quantity)`, `reserve(items[])`, `release(reservationId)`

### DeliverySlotManager
Manages 2-hour delivery windows. Handles slot availability queries and optimistic-lock reservations.
A slot reservation expires after 10 minutes if checkout isn't completed.

### CheckoutOrchestrator
Coordinates the checkout flow: validate cart → reserve inventory → reserve delivery slot → process payment → confirm order.
If any step fails, it unwinds previous steps (saga pattern).
```

### 5. Technology Choices

State what you're using and why. Link choices back to scenarios or constraints when possible.

```markdown
## Technology Choices

| Choice | Rationale |
|--------|-----------|
| React + TypeScript | User asked for a web app; TypeScript catches type errors in cart/pricing logic |
| Node.js + Express | Matches frontend language; sufficient for expected throughput |
| PostgreSQL | Relational model fits cart/order/inventory relationships; row-level locking for slot reservations |
| Redis | Session storage + short-lived slot reservation locks |
```

Only include choices that matter. Don't list your linter or your test runner here unless the choice is non-obvious and worth justifying.

### 6. Sequencing

Break the build into ordered steps where each step produces something runnable or testable. This isn't a project plan with dates — it's a dependency-aware build order.

```markdown
## Build Sequence

### Step 1: Cart basics
Build: CartService, CartItem data model, `POST/DELETE /cart/items`, CartSidebar UI
Satisfies: "Adding an in-stock item", "Removing an item from cart", "Viewing cart contents"
Testable: Can add/remove items and see updated cart

### Step 2: Pricing and discounts
Build: Discount logic in CartService, DiscountCode model, discount API endpoint, discount UI
Satisfies: "Applying a percentage discount", "Applying an expired discount code"
Depends on: Step 1 (needs a cart with items)

### Step 3: Inventory checks
Build: InventoryService, inventory validation in add-to-cart flow
Satisfies: "Adding an out-of-stock item", "Item going out of stock while in cart"
Depends on: Step 1

### Step 4: Delivery scheduling
Build: DeliverySlotManager, slot selection UI, slot reservation
Satisfies: "Selecting a delivery window", "Delivery window filling during checkout"
Depends on: Step 1

### Step 5: Checkout
Build: CheckoutOrchestrator, payment integration, order confirmation
Satisfies: "Completing a checkout", "Payment failure during checkout"
Depends on: Steps 1-4
```

Guidelines:
- Each step should reference the scenarios it satisfies
- Each step should produce something that can be run or demonstrated
- Earlier steps should have fewer dependencies
- If two steps are independent, note that they can be done in parallel

### 7. Open Questions and Risks

Anything that came up during design that needs input, and any risks worth flagging.

```markdown
## Open Questions

- The spec mentions "delivery zones" but doesn't define how they're determined. Is this zip-code-based? Radius from warehouse? This affects DeliverySlotManager significantly.
- Discount stacking: the spec has scenarios for single discount codes but doesn't say whether multiple codes can be combined. Designed for single-code for now.

## Risks

- Optimistic locking on delivery slots could cause poor UX under high concurrency (many customers competing for the last slot in a window). May need a queue-based approach if this proves problematic.
- Capturing `unit_price_at_addition` means price changes require a decision: update existing carts or not? Spec doesn't cover this. Designed for "price at addition stands" for now.
```

## Workflow

### Step 1: Read the Spec

Read the entire spec before designing anything. Look for:
- **Implicit technical requirements**: scenarios about concurrent users imply locking or queuing. Scenarios about "real-time updates" imply websockets or polling. Scenarios about "within 2 seconds" imply performance constraints.
- **Data relationships**: what entities exist? What references what? What's the cardinality?
- **Workflow dependencies**: which scenarios describe steps in a sequence? This often maps to a service that orchestrates the sequence.
- **Error scenarios**: these often reveal the hardest design problems (rollback, partial failure, retry).

### Step 2: Draft the Scenario Map

Go through every scenario and sketch which components are involved. This is the most important step — it forces you to confront every behavior the system needs to support before committing to an architecture.

If you find scenarios that don't map cleanly to any component, that's a signal you're missing a component. If you find a component that's involved in everything, it's probably too big and should be split.

### Step 3: Define the Data Model

Derive the data model from the scenario map. Every entity in the model should be traceable to at least one scenario. If you're modeling something no scenario references, either the spec has a gap (flag it) or you're over-engineering (cut it).

### Step 4: Define Components and Interactions

Group related logic into components. Each component should have a clear responsibility that you can state in one sentence. If you can't, it's doing too much.

### Step 5: Choose Technologies

Make choices based on the constraints surfaced in earlier steps. Default to boring, well-understood technologies unless a scenario specifically demands something exotic.

### Step 6: Sequence the Build

Order the work so that each step builds on the previous one and produces something demonstrable. Front-load the core happy-path scenarios — get the basic flow working end-to-end before handling edge cases.

### Step 7: Flag Questions and Risks

Be explicit about what you're assuming, what you're punting on, and what might go wrong. This is not a sign of weakness in the design — it's a sign of honesty.

## Principles

**Every design decision should trace to a scenario.** If you can't point to a scenario that motivates a decision, you're either over-engineering or the spec is incomplete. Flag it either way.

**Design for the scenarios you have, not the ones you imagine.** It's tempting to add abstractions "in case we need to support X later." Resist. Design for the spec as written. If X becomes a real scenario, the spec will be updated and the design will evolve.

**Name things consistently with the spec.** If the spec says "cart", the design says "cart", the code says "cart". Don't introduce synonyms.

**Prefer simple over clever.** A straightforward design that's easy to understand and modify beats an elegant design that's hard to follow. The design will be read by future-you and by other developers. Optimize for their comprehension.

**Be specific about boundaries.** "The frontend never computes prices" is a useful design statement. "Pricing logic is centralized" is not — it's too vague to enforce.

## Output Format

Produce the design as a single Markdown document. Use this skeleton:

```markdown
# Design: <Name of the Thing>

## Summary

<2-4 sentences>

## Scenario Map

### "<Scenario name from spec>"
- **UI**: ...
- **API**: ...
- **Data**: ...
- **Key logic**: ...

<repeat for each scenario or scenario group>

## Data Model

### <Entity>
- <fields, constraints, relationships>

## Components

### <ComponentName>
<1-2 sentence responsibility statement>
<key interfaces or methods if helpful>

## Technology Choices

| Choice | Rationale |
|--------|-----------|
| ...    | ...       |

## Build Sequence

### Step 1: <name>
Build: ...
Satisfies: ...
Testable: ...

## Open Questions

- ...

## Risks

- ...
```

Save the design as `design.md` alongside the spec. The design should always reference the spec, and the spec should remain the authority on *what* the system does.
