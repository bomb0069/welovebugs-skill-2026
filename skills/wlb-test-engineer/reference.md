# Test Case Design Techniques Reference

This file provides additional detail on test design techniques. Load this reference
when the user asks for deeper coverage or when dealing with complex requirements.

## Equivalence Partitioning

Divide input data into groups (partitions) where the system is expected to behave
the same way. Test one representative value from each partition.

**Example**: Age field accepting 18-65
- Valid partition: 18-65 (test with 30)
- Invalid partition below: 0-17 (test with 10)
- Invalid partition above: 66+ (test with 70)

## Boundary Value Analysis

Test at the exact edges of equivalence partitions. Defects cluster at boundaries.

**Example**: Age field accepting 18-65
- Test: 17, 18, 19, 64, 65, 66

## Decision Table Testing

Use when a requirement has multiple conditions that combine to produce different outcomes.

| Condition A | Condition B | Expected Outcome |
|-------------|-------------|------------------|
| True        | True        | Action 1         |
| True        | False       | Action 2         |
| False       | True        | Action 3         |
| False       | False       | Action 4         |

## State Transition Testing

Use when the system has distinct states and defined transitions between them.

1. Identify all states
2. Identify valid transitions
3. Test each valid transition
4. Test invalid transitions (attempting actions not allowed in a given state)

**Example**: Order status
```
Draft -> Submitted -> Approved -> Fulfilled -> Closed
                   -> Rejected -> Draft (revision)
```

Invalid: Draft -> Fulfilled (skipping approval)

## Pairwise / Combinatorial Testing

When there are many input parameters, testing all combinations is infeasible.
Pairwise testing covers every pair of parameter values at least once, reducing
the number of test cases while catching most interaction defects.

Use this when:
- There are 3+ independent input fields
- Full combinatorial coverage would produce hundreds of cases
- The user needs a practical subset

## User Story Test Derivation

Given a user story in the format:
> As a [role], I want [goal], so that [benefit]

Derive tests by asking:
1. What does the happy path look like for this goal?
2. What happens when the goal cannot be achieved? (error cases)
3. What are the limits of the goal? (boundary cases)
4. Are there different variations of the role? (access control)
5. What other features does this interact with? (integration)
