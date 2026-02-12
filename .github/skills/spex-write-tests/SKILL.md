---
name: spex-write-tests
description: "Add concise but thorough test coverage where needed, ensuring each test links back to one or more scenarios in the spec. Use this skill when the user wants to add, improve, or review tests for code that has a behavioral spec. Triggers include: 'write tests', 'add test coverage', 'test this', 'are there enough tests', 'cover these scenarios', or any request to verify that implemented code satisfies the spec's scenarios. Also use when the user asks to check which scenarios are untested or wants to understand the relationship between tests and spec. Do NOT use for writing specs (use spex-specify), creating designs (use spex-design), or implementing features (use spex-implement)."
---

# Writing Tests That Trace to Scenarios

## Overview

Tests verify that the implementation satisfies the spec. Every test should trace back to one or more scenarios in the spec, and every scenario in the spec should be covered by at least one test. This bidirectional traceability is the core discipline — it ensures tests are meaningful (they verify real behavior) and complete (no specified behavior goes unverified).

## What Makes Good Tests

Good tests:
- **Map clearly to scenarios.** You can look at any test and point to the scenario it verifies. You can look at any scenario and find the tests that cover it.
- **Test behavior, not implementation.** They verify *what* happens, not *how* it happens internally. They don't break when you refactor.
- **Are independent.** Each test sets up its own state, runs, and cleans up. No test depends on another test running first.
- **Are readable.** Someone unfamiliar with the code can read a test and understand what behavior it's verifying.
- **Fail for the right reason.** When a test fails, the failure message tells you which behavior broke, not just which assertion didn't hold.

Bad tests:
- **Test implementation details.** Verifying that a specific private method was called, or that an internal data structure has a specific shape.
- **Don't link to scenarios.** "test_function_works" tells you nothing about what behavior is covered.
- **Are fragile.** Break when you rename a variable, reorder fields, or refactor without changing behavior.
- **Are redundant.** Multiple tests verifying the same scenario in the same way.
- **Are incomplete.** Cover only the happy path when the spec has error scenarios too.

## Test Structure

### Naming Convention

Test names encode the scenario they verify. Use this pattern:

```
test_<scenario_name_in_snake_case>
```

The name should match or closely paraphrase the scenario name from the spec. If a scenario is "Applying an expired discount code," the test is:

```python
def test_applying_an_expired_discount_code():
    ...
```

```javascript
test('applying an expired discount code', () => {
    ...
});
```

This naming convention is the primary traceability mechanism. Someone reading the test file should be able to mentally map each test back to the spec without any additional documentation.

### Test Body: Arrange / Act / Assert

Each test follows the same three-phase structure that mirrors given/when/then:

```
Arrange  →  set up the preconditions           (maps to Given)
Act      →  perform the action                  (maps to When)
Assert   →  verify the outcome                  (maps to Then)
```

Separate these phases visually with blank lines. This makes the mapping to the scenario explicit:

```python
def test_applying_a_percentage_discount_code():
    # Given the cart contains items totaling $10.50
    cart = create_cart_with_items([
        ("Sourdough Bread", 6.00),
        ("Oat Milk", 4.50),
    ])

    # When the customer applies discount code "SAVE20"
    result = cart.apply_discount("SAVE20")

    # Then the cart total shows $8.40 and the discount line is correct
    assert result.total == 8.40
    assert result.discount_line == "SAVE20: -$2.10"
```

The `# Given / # When / # Then` comments are optional but helpful, especially when the mapping isn't immediately obvious.

### Scenario Coverage Tag

At the top of each test file (or test class), include a coverage summary that maps scenarios to tests. This serves as a quick-reference index:

```python
"""
Scenario Coverage:
- "Adding an in-stock item to the cart"       → test_adding_an_in_stock_item
- "Adding an out-of-stock item to the cart"   → test_adding_an_out_of_stock_item
- "Removing an item from the cart"            → test_removing_an_item
- "Applying a percentage discount code"       → test_applying_a_percentage_discount_code
- "Applying an expired discount code"         → test_applying_an_expired_discount_code
"""
```

This header makes it trivially easy to audit coverage: if a scenario from the spec isn't listed, it's not tested.

## Workflow

### Step 1: Read the Spec and Map Scenarios

Before writing any test, read the full spec and list every scenario. Then check which scenarios already have tests (if any code/tests exist). The gap between "scenarios in the spec" and "scenarios with tests" is your work list.

Produce a coverage checklist (mentally or in writing):

```
[x] Adding an in-stock item           — test exists in test_cart.py
[ ] Adding an out-of-stock item        — NO TEST
[x] Removing an item                   — test exists in test_cart.py
[ ] Applying a percentage discount     — NO TEST
[ ] Applying an expired discount       — NO TEST
```

### Step 2: Read the Implementation

Understand how the code is structured before writing tests. You need to know:
- What functions/methods to call in your tests
- What setup is required (database, fixtures, mocks)
- What the return types and error types look like
- What testing patterns the codebase already uses (if any tests exist)

If tests already exist, match their style and conventions. If not, choose a testing framework appropriate for the language and stack, and establish simple conventions.

### Step 3: Write Tests for Uncovered Scenarios

For each uncovered scenario, write one test (sometimes more if a scenario has multiple important variants). Follow the naming convention and arrange/act/assert structure.

**Start with happy-path scenarios**, then error cases, then edge cases. This matches the natural priority: core behavior first, failure modes second, boundary conditions third.

**One scenario = one test (usually).** If a scenario has multiple "And then" clauses, it's fine for one test to verify all of them. Don't split a single scenario into multiple tests — that fragments the traceability.

**Occasionally one scenario needs multiple tests** if the given/when/then describes a category of behavior with meaningfully different inputs. For example, if the spec says "applying a discount code" and the design reveals both percentage and fixed-amount discounts, you might write:

```python
def test_applying_a_percentage_discount_code():
    ...

def test_applying_a_fixed_amount_discount_code():
    ...
```

Both trace to the same scenario (or closely related scenarios) and that's fine. The key is that the mapping remains clear.

### Step 4: Write Helpers, Not Frameworks

If multiple tests need similar setup, extract helper functions. Keep them simple and specific:

```python
def create_cart_with_items(items):
    """Helper: creates a cart and adds items. Each item is (name, price)."""
    cart = Cart.create(customer_id="test-customer")
    for name, price in items:
        cart.add_item(product_name=name, unit_price=price, quantity=1)
    return cart
```

**Don't build a test framework.** Helpers should be plain functions that set up state. Avoid:
- Abstract base test classes with complex inheritance
- Custom assertion libraries
- "Smart" fixtures that auto-discover and auto-configure
- Data-driven test generators (unless you have dozens of trivially similar cases)

Keep the test code boring and explicit. Test code is documentation — clarity trumps DRYness.

### Step 5: Verify Tests Pass and Fail Correctly

After writing tests:

1. **Run them and confirm they pass.** If a test fails, either the test is wrong or the implementation has a bug. Investigate and fix the right one.
2. **Intentionally break the implementation and confirm the relevant test fails.** This validates that the test is actually testing what you think it is. A test that passes regardless of what the code does is worse than no test — it provides false confidence.
3. **Check the failure message.** When a test fails, does the message clearly indicate which behavior broke? If the message is cryptic, improve the assertion or add a message:

```python
assert result.total == 8.40, (
    f"Expected discounted total of 8.40, got {result.total}. "
    f"Discount may not have been applied correctly."
)
```

### Step 6: Update the Coverage Header

After writing all tests, update the scenario coverage summary at the top of the test file. Confirm every scenario from the spec is listed with its corresponding test.

## Test Levels

### Unit Tests (Most Tests Live Here)

Test individual functions and methods in isolation. Mock or stub external dependencies (databases, APIs, file systems) so tests are fast and focused.

Unit tests map most naturally to scenarios because they verify specific behaviors with specific inputs and outputs — exactly what scenarios describe.

### Integration Tests (Use Sparingly, Targeted)

Test that components work together correctly. Use these for:
- Scenarios that span multiple components (like the checkout flow)
- Data model scenarios (can you actually save and retrieve entities?)
- API contract scenarios (does the endpoint accept and return what the spec implies?)

Integration tests are slower and more complex. Write them for cross-component scenarios, but don't duplicate what unit tests already cover.

### End-to-End Tests (Rare, High-Value Only)

Test the full stack from UI to database. Reserve these for the most critical happy-path scenarios — the flows that absolutely must work. If the spec has a core workflow (browse → cart → checkout → confirmation), one or two E2E tests covering that flow are valuable. Don't E2E-test every scenario.

## What Not to Test

- **Implementation details.** Don't test that a private method is called, or that an internal cache has a specific structure. Test the observable behavior.
- **Third-party libraries.** Don't test that your database driver can insert a row. Test that *your code* correctly uses the driver to satisfy a scenario.
- **Scenarios that don't exist in the spec.** If you find yourself writing a test for behavior that isn't in any scenario, either the spec has a gap (flag it) or you're testing something that doesn't matter (skip it).

## Output

Tests are saved as test files in the appropriate location for the project (e.g., `tests/`, `__tests__/`, `*_test.go`, etc.). After writing tests:

- All tests pass when run
- The scenario coverage header is present and complete
- Test names clearly correspond to scenario names
- The test file is readable as a standalone document of the system's expected behavior
