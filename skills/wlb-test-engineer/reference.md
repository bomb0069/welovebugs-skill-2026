# Boundary Value Analysis Reference

This file provides detailed guidance on the Boundary Value Analysis (BVA) technique.
Load this reference when the user needs deeper explanation or when dealing with
complex numeric boundaries.

## What is Boundary Value Analysis?

BVA is a test design technique that focuses on values at the edges of input ranges.
Defects are more likely to occur at boundaries than in the middle of a range.

## How to identify boundaries

Given a valid range **[min, max]** for a numeric field:

| Position    | Value   | Classification |
|-------------|---------|----------------|
| Below Min   | min - 1 | Invalid        |
| Min (on)    | min     | Valid          |
| Above Min   | min + 1 | Valid          |
| Below Max   | max - 1 | Valid          |
| Max (on)    | max     | Valid          |
| Above Max   | max + 1 | Invalid        |

## Handling different numeric types

### Integers
Use ±1 as the step. Example: range 1-100 → test 0, 1, 2, 99, 100, 101.

### Decimals / Currency
Use the smallest meaningful unit. Example: price 0.01-999.99 → test 0.00, 0.01, 0.02,
999.98, 999.99, 1000.00.

### Percentages
Typically 0-100. Test: -1, 0, 1, 99, 100, 101. Watch for whether 0 and 100 are
inclusive or exclusive.

## Multiple numeric fields

When a requirement has multiple numeric inputs:

1. **Test each field independently first** — vary one field at a time while keeping
   others at a typical valid value.
2. **Note field interactions** — if the requirement states rules that combine fields
   (e.g., "total = price × quantity, must not exceed 10,000"), also test boundary
   combinations.

## Common boundary pitfalls

- **Off-by-one errors** — the most common defect BVA catches
- **Inclusive vs exclusive ranges** — "greater than 0" vs "greater than or equal to 0"
- **Zero as a boundary** — often a special case (division, empty quantity)
- **Negative numbers** — check if negative values are possible and how they're handled
- **Maximum system limits** — integer overflow, max database column size
