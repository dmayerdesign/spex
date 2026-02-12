---
name: spex-design
description: "Create a detailed plan for implementing the spec. Use this skill when a behavioral spec (scenarios in given/when/then form) already exists and the user wants to plan how to build it before writing code. Triggers include: 'design this', 'plan the implementation', 'how should we build this', 'create an architecture', 'what's the technical approach', or any request to bridge the gap between a spec and code. Also use when the user has a spec and asks about technology choices, component structure, data modeling, or sequencing of work. Do NOT use for writing the spec itself (use spex-specify), writing code (use spex-implement), or writing tests (use spex-write-tests)."
---

# Designing from a Spec

## Overview

A design document bridges _what_ the software does (the spec) and _how_ it gets built (the implementation). It sits at the second level of the precedence hierarchy: **spec.md → design.md → tests → implementation**. The spec is the authority on behavior; the design is the authority on structure. If they conflict, the spec wins.

The design's primary audience is the agent or developer who will implement it using `spex-implement` with a TDD workflow. A good design makes implementation mechanical: follow the build sequence step by step, writing tests (via `spex-write-tests`) and code for each step, without needing to make architectural decisions on the fly.

## Before You Start

1. **Read every scenario in the spec.** Pay special attention to error scenarios and edge cases — they drive component boundaries more than happy paths do.
2. **Extract the nouns and verbs.** The spec's vocabulary becomes your entity and component names. This is not optional.
3. **Note technology constraints** the developer has specified. If none, you'll recommend and justify.
4. **Check for existing code.** Don't design a greenfield architecture for a brownfield project.

## Design Principles

### Use the Spec's Vocabulary

If the spec says "cart," your design has a Cart entity, a CartService, and a cart table. Don't rename things to fit a pattern. The vocabulary flows downward: spec → design → tests → code.

### Every Component Traces to Scenarios

If you can't name the scenario that requires a component, it probably shouldn't exist. If you find yourself designing something no scenario motivates — an admin panel, a caching layer — flag it to the developer rather than including it. The right response to a gap is to update the spec (using `spex-specify`), not to silently expand scope.

### Design for Testability

Implementation will be test-driven. Your design must make that practical:

- Components should have **clear interfaces** with well-defined inputs and outputs.
- **Business logic should be separable from infrastructure** — core rules in pure components, I/O at the edges.
- The **build sequence should produce testable increments** — each step adds functionality that can be verified before the next step begins.

If you can't describe in a sentence how a component would be tested, its boundaries are wrong.

### Decide What the Spec Doesn't

The spec is silent on persistence, transport, error propagation, concurrency, configuration, and external integrations. Make explicit decisions about each and write them down. Implicit decisions become inconsistent implementations.

### Keep It Buildable in Order

Each step in the build sequence must produce something that builds, can be tested, and works. If a step requires four other incomplete steps before anything is verifiable, the sequence is wrong.

### Avoid Over-Specification

Favor bullet points, flowcharts, or pseudocode over code snippets.
Favor qualitative descriptions over quantities, except when prescriptive precision is needed to avoid ambiguity.

## Workflow

### Step 1: Extract the Entity Model

From the spec's scenarios, identify entities, their attributes, state transitions, and relationships. Use the spec's exact terminology. Don't invent entities the spec doesn't imply — note infrastructure entities separately and justify them.

### Step 2: Define the Components

Group related behaviors into components. For each one: name it from the spec's vocabulary, state its responsibility in one sentence, define its interface (what it accepts, what it returns, what errors it raises), and list the scenarios it serves. Separate business logic from infrastructure.

### Step 3: Choose the Technology

Language, framework (if any), data store, test framework, and external dependencies — each with rationale. Prefer boring technology. Include the test framework as a first-class choice, not an afterthought. Justify each external dependency; fewer is better.

### Step 4: Design the Interfaces

Specify contracts in enough detail that the implementer doesn't guess: API endpoints with methods, paths, request/response shapes, and status codes; function signatures with parameters, return types, and errors; example payloads rather than abstract descriptions. These definitions are what `spex-write-tests` consumes directly — precision here makes tests easy to write.

### Step 5: Plan the Build Sequence

This is the most critical section. Each step must:

1. **Be testable on its own.** After completing it, the implementer writes and runs tests via `spex-write-tests`.
2. **Build on previous steps** with no forward references.
3. **Leave the project buildable.** The implementation skill enforces this — the design must make it achievable.
4. **Trace to scenarios.** By the final step, every scenario is accounted for.

For each step, specify: what to build, which scenarios it addresses, what to test before moving on, and what must be in place from prior steps.

### Step 6: Note Decisions and Open Questions

Document judgment calls with reasoning ("chose SQLite because the spec describes a single-user tool"), assumptions that need confirmation ("assumed empty-cart checkout returns an error — confirm with spec"), and deferred concerns ("spec doesn't mention auth; would affect API design if added later"). Open questions are invitations for the developer to update the spec (using `spex-specify`) or confirm the assumption.

## Resolving Conflicts

- **Spec seems incomplete or contradictory:** flag it. Suggest a resolution, but the spec should be updated (using `spex-specify`) rather than the design silently interpreting ambiguity.
- **Existing code fights the spec:** the spec wins. Include refactoring steps in the build sequence.
- **Technology constraint makes a scenario impractical:** flag the tradeoff with options and a recommendation.
- **Unsure about a decision:** write down the options, tradeoffs, and your recommendation. Let the developer decide. Don't get stuck seeking perfection — a documented decision beats an agonized-over one that blocks progress.

## Updating an Existing Design

When the spec changes or implementation reveals a needed adjustment, update the design surgically. Re-read the current spec, identify impacted components and build steps, make the change, and flag downstream impacts to `spex-implement` and `spex-write-tests`.

## Output

A `design.md` containing: entity model, component breakdown (with scenarios each serves), technology choices with rationale, interface definitions, build sequence, and decisions/open questions. Favor concreteness — example payloads over schema descriptions, specific names over placeholders, one clear recommendation over an unprioritized list of options.
