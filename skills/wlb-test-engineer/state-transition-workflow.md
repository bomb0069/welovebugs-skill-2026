# Workflow D: State Transition Testing

**Use when**: requirement describes **statuses, stages, or lifecycle** of something —
orders, documents, tickets, accounts, payments, approvals, etc. The key signal is
that the requirement talks about something changing from one state to another.

Example requirements:
> "An order starts as Draft, then moves to Submitted. After review it can be
> Approved or Rejected. Rejected orders go back to Draft. Approved orders move
> to Shipped, then Delivered."

> "A leave request can be Pending, Approved, Rejected, or Cancelled. Only Pending
> requests can be approved or rejected. Approved requests can be cancelled."

Work through each step in order. **Only pause and wait for user input when a step
is marked with ⏸.** For all other steps, continue automatically.

## Step 1: Identify all states ⏸ Wait for user input

Analyze the requirement and extract every **state** (status, stage) that the
entity can be in.

Present the states as a table and **ask the user to confirm**:

```markdown
I found the following states for **Order**:

| # | State     | Description                        | Initial? | Final? |
|---|-----------|------------------------------------|----------|--------|
| 1 | Draft     | Order is being created             | ✓        |        |
| 2 | Submitted | Order sent for review              |          |        |
| 3 | Approved  | Order passed review                |          |        |
| 4 | Rejected  | Order failed review                |          |        |
| 5 | Shipped   | Order dispatched to customer       |          |        |
| 6 | Delivered | Order received by customer         |          | ✓      |

Is this correct? Are there any states I missed?
```

Mark which states are:
- **Initial** — the starting state (where the entity begins)
- **Final** — end states (where the entity stops changing)

## Step 2: Identify all transitions

List every **valid transition** — a change from one state to another, including
what **event/action** triggers it.

```markdown
| # | From State | Event / Action     | To State  |
|---|------------|--------------------|-----------|
| 1 | Draft      | Submit order       | Submitted |
| 2 | Submitted  | Approve            | Approved  |
| 3 | Submitted  | Reject             | Rejected  |
| 4 | Rejected   | Revise and resubmit| Draft     |
| 5 | Approved   | Ship order         | Shipped   |
| 6 | Shipped    | Confirm delivery   | Delivered |
```

## Step 3: Draw state transition diagram

Draw a **visual state transition diagram** showing all states and transitions.
Use arrows with event labels.

```
        State Transition Diagram: Order

        ┌─────────┐   Submit    ┌───────────┐   Approve   ┌──────────┐
        │         │────────────►│           │────────────►│          │
        │  Draft  │             │ Submitted │             │ Approved │
        │         │◄────────────│           │             │          │
        └─────────┘   Revise    └───────────┘             └──────────┘
           [START]                    │                         │
                                     │ Reject                  │ Ship
                                     ▼                         ▼
                                ┌───────────┐            ┌──────────┐
                                │           │            │          │
                                │ Rejected  │            │ Shipped  │
                                │           │            │          │
                                └───────────┘            └──────────┘
                                     │                         │
                                     │ Revise                  │ Confirm
                                     │ (goes to Draft)         │ delivery
                                     ▼                         ▼
                                  ┌─────┐                ┌───────────┐
                                  │Draft│                │ Delivered │
                                  └─────┘                │   [END]   │
                                                         └───────────┘
```

For more complex diagrams, also present as a **state transition matrix**
(states × events) so nothing is missed:

```markdown
#### State Transition Matrix

|            | Submit    | Approve   | Reject    | Revise    | Ship      | Confirm   |
|------------|-----------|-----------|-----------|-----------|-----------|-----------|
| Draft      | Submitted | —         | —         | —         | —         | —         |
| Submitted  | —         | Approved  | Rejected  | —         | —         | —         |
| Approved   | —         | —         | —         | —         | Shipped   | —         |
| Rejected   | —         | —         | —         | Draft     | —         | —         |
| Shipped    | —         | —         | —         | —         | —         | Delivered |
| Delivered  | —         | —         | —         | —         | —         | —         |
```

Cells with **—** are **invalid transitions** (should not be allowed by the system).

## Step 4: Identify test scenarios

From the diagram and matrix, derive two types of test scenarios:

### 4a: Valid transition tests

Test every valid transition (each arrow in the diagram):

```markdown
#### Valid Transitions

| # | From State | Event            | To State  | Expected Behavior                                |
|---|------------|------------------|-----------|--------------------------------------------------|
| 1 | Draft      | Submit order     | Submitted | Status changes to Submitted, reviewer notified   |
| 2 | Submitted  | Approve          | Approved  | Status changes to Approved, warehouse notified   |
| 3 | Submitted  | Reject           | Rejected  | Status changes to Rejected, reason required       |
| 4 | Rejected   | Revise & resubmit| Draft    | Status changes to Draft, editable again           |
| 5 | Approved   | Ship order       | Shipped   | Status changes to Shipped, tracking number added  |
| 6 | Shipped    | Confirm delivery | Delivered | Status changes to Delivered, marked as complete   |
```

### 4b: Invalid transition tests

Test transitions that should **NOT** be allowed. Pick the most important
invalid transitions from the matrix (cells with —):

```markdown
#### Invalid Transitions (should be blocked)

| # | From State | Attempted Event  | Expected Behavior                                      |
|---|------------|------------------|--------------------------------------------------------|
| 1 | Draft      | Approve          | System should block: "Cannot approve a draft order"    |
| 2 | Draft      | Ship             | System should block: "Cannot ship a draft order"       |
| 3 | Submitted  | Ship             | System should block: "Order must be approved first"    |
| 4 | Approved   | Reject           | System should block: "Cannot reject an approved order" |
| 5 | Shipped    | Approve          | System should block: "Order already shipped"           |
| 6 | Delivered  | Submit           | System should block: "Order already completed"         |
| 7 | Delivered  | Ship             | System should block: "Order already delivered"         |
```

**How to select which invalid transitions to test:**
- Don't test every cell with — (too many). Focus on:
  - Transitions that users might **realistically attempt** (e.g., trying to ship before approval)
  - Transitions that would cause **data integrity issues** if allowed
  - Transitions from **final states** (nothing should change after completion)

### 4c: Path tests (optional but recommended)

Test complete **paths** through the state diagram — sequences of transitions
from initial to final state:

```markdown
#### Path Tests

| # | Path Name        | Sequence                                            |
|---|------------------|-----------------------------------------------------|
| 1 | Happy path       | Draft → Submitted → Approved → Shipped → Delivered  |
| 2 | Rejection path   | Draft → Submitted → Rejected → Draft                |
| 3 | Resubmit path    | Draft → Submitted → Rejected → Draft → Submitted → Approved → Shipped → Delivered |
```

## Step 5: Collect real-world representative data ⏸ Wait for user input (REQUIRED)

The transitions from Step 4 define **what to test**. For acceptance tests,
we need realistic data about **how real users trigger these transitions**.

Ask the user:

```markdown
Please provide real-world context for each transition:

| Transition           | What triggers it in real life?   | Who does it?  | Common scenario        |
|----------------------|---------------------------------|---------------|------------------------|
| Draft → Submitted    | ❓ Please provide                | ❓             | ❓                      |
| Submitted → Approved | ❓ Please provide                | ❓             | ❓                      |
| Submitted → Rejected | ❓ Please provide                | ❓             | ❓                      |
| Rejected → Draft     | ❓ Please provide                | ❓             | ❓                      |
| Approved → Shipped   | ❓ Please provide                | ❓             | ❓                      |
| Shipped → Delivered  | ❓ Please provide                | ❓             | ❓                      |
```

Guiding questions:
- "Who performs this action in your system? (e.g., customer, manager, warehouse staff)"
- "What data does the user enter when triggering this transition?"
- "What is a common real-world scenario for this transition?"

Same enforcement rules: push back if user tries to skip, mark `⚠️ TBD` if they
still choose to skip.

## Step 6: Generate final test case tables

Produce **two separate tables**.

---

### Table 1: Unit Test Cases (State Transition Technique)

Tests every valid transition and key invalid transitions with technical data.

Label: **"Unit Test Cases (State Transition)"**

The table format for state transition uses these columns:
- Input: **Current State**, **Event/Action**, and any additional input fields
- Output: **New State**, **Result**, **Message**

```markdown
#### Unit Test Cases (State Transition)

| TC-ID | Test Case Name                  | Test Description                                                                | Input  | Input          | Output    | Output | Output                                    | Type    |
|       |                                 |                                                                                 | State  | Event          | New State | Result | Message                                   |         |
|-------|-------|-------------------------------|---------------------------------------------------------------------------------|--------|----------------|-----------|--------|--------------------------------------------|---------|
| UT-01 | Valid: Draft → Submitted        | Submit a Draft order and system should change status to Submitted               | Draft     | Submit      | Submitted | Accept | Status changed, reviewer notified         | Valid   |
| UT-02 | Valid: Submitted → Approved     | Approve a Submitted order and system should change status to Approved           | Submitted | Approve     | Approved  | Accept | Status changed, warehouse notified        | Valid   |
| UT-03 | Valid: Submitted → Rejected     | Reject a Submitted order and system should change status to Rejected            | Submitted | Reject      | Rejected  | Accept | Status changed, reason recorded           | Valid   |
| UT-04 | Valid: Rejected → Draft         | Revise a Rejected order and system should change status back to Draft           | Rejected  | Revise      | Draft     | Accept | Status changed, editable again            | Valid   |
| UT-05 | Valid: Approved → Shipped       | Ship an Approved order and system should change status to Shipped               | Approved  | Ship        | Shipped   | Accept | Status changed, tracking added            | Valid   |
| UT-06 | Valid: Shipped → Delivered      | Confirm delivery of Shipped order and system should change status to Delivered  | Shipped   | Confirm     | Delivered | Accept | Status changed, order complete            | Valid   |
| UT-07 | Invalid: Draft → Approve        | Try to approve a Draft order and system should block the action                 | Draft     | Approve     | Draft     | Reject | "Cannot approve a draft order"            | Invalid |
| UT-08 | Invalid: Submitted → Ship       | Try to ship a Submitted order and system should block the action                | Submitted | Ship        | Submitted | Reject | "Order must be approved first"            | Invalid |
| UT-09 | Invalid: Delivered → Submit      | Try to submit a Delivered order and system should block the action              | Delivered | Submit      | Delivered | Reject | "Order already completed"                 | Invalid |
```

Number as UT-01, UT-02, etc.

---

### Table 2: Acceptance Test Cases (Business Scenarios)

Tests realistic end-to-end **paths** and common invalid attempts using real-world data.

Label: **"Acceptance Test Cases (Business Scenarios)"**

```markdown
#### Acceptance Test Cases (Business Scenarios)

| TC-ID | Test Case Name                     | Test Description                                                                                          | Input     | Input          | Input             | Output    | Output | Output                                 | Type    |
|       |                                    |                                                                                                           | State     | Event          | Actor             | New State | Result | Message                                |         |
|-------|--------|----------------------------|-----------------------------------------------------------------------------------------------------------|-----------|----------------|-------------------|-----------|--------|-----------------------------------------|---------|
| AT-01 | Happy path: order to delivery      | Sales creates order, manager approves, warehouse ships, customer confirms — full lifecycle                | Draft     | Full path      | Sales→Mgr→WH→Cust | Delivered | Accept | Order completed successfully           | Valid   |
| AT-02 | Rejection and resubmit             | Sales creates order, manager rejects due to wrong price, sales revises and resubmits                     | Draft     | Submit→Reject→Revise→Submit | Sales→Mgr→Sales | Submitted | Accept | Order resubmitted for review  | Valid   |
| AT-03 | Warehouse tries to ship before approval | Warehouse staff tries to ship an order that hasn't been approved yet                                 | Submitted | Ship           | Warehouse         | Submitted | Reject | "Order must be approved first"         | Invalid |
| AT-04 | Customer tries to reopen completed order | Customer contacts support wanting to change a delivered order                                        | Delivered | Revise         | Customer          | Delivered | Reject | "Order already completed"              | Invalid |
```

Number as AT-01, AT-02, etc.

Same `⚠️ TBD` rules apply when real-world data was not provided.
