# Eval: State Transition Workflow (Workflow D)

Tests for State Transition Testing — requirements with states, statuses, and lifecycles.

---

### EVAL-ST01: Simple linear lifecycle

**Requirement:**
> Order lifecycle: New → Processing → Shipped → Delivered.
> Orders cannot skip states or go backwards.

**User Responses:**
1. Workflow selection → "D"
2. State confirmation → confirm 4 states
3. Transition confirmation → confirm transitions
4. Real-world data → "Orders are placed daily. Processing takes 1-2 days. Shipping takes 3-5 days."

**Evaluation Criteria:**

Step 1 — States:
- [ ] Identifies 4 states: New, Processing, Shipped, Delivered
- [ ] Marks New as Initial and Delivered as Final
- [ ] Shows state table with Initial?/Final? columns
- [ ] Pauses (⏸) for user confirmation

Step 2 — Transitions:
- [ ] Lists 3 transitions: New→Processing, Processing→Shipped, Shipped→Delivered
- [ ] Each transition has From, Event, To columns
- [ ] Pauses (⏸) for user validation

Step 3 — Diagrams:
- [ ] **State Transition Diagram**: box-and-arrow ASCII showing 4 states connected linearly
- [ ] **State Transition Matrix**: 4×4 grid with valid transitions marked and "—" for invalid cells
- [ ] Matrix clearly shows that backward/skip transitions are "—"

Step 4 — Test scenarios:
- [ ] **4a Valid Transitions**: 3 test cases (one per valid transition)
- [ ] **4b Invalid Transitions**: picks important blocked transitions (e.g., Shipped→New, Delivered→Processing)
- [ ] **4c Path Tests**: at least 1 happy path (New→Processing→Shipped→Delivered)
- [ ] Invalid transitions chosen based on realism (what a user might actually attempt)

Step 5 — Real-world data:
- [ ] Pauses (⏸) to collect real-world data
- [ ] Asks about who performs transitions, what triggers them

Step 6 — Final tables:
- [ ] **Unit Test Cases** with UT-01, UT-02, ... using Current State → Event → New State columns
- [ ] **Acceptance Test Cases** with AT-01, AT-02, ... using real-world context
- [ ] AT table includes actor information (who triggers the transition)
- [ ] Both valid and invalid transitions appear in the tables
- [ ] Type column: Valid for expected transitions, Invalid for blocked transitions

---

### EVAL-ST02: Lifecycle with branches and loops

**Requirement:**
> Leave request process:
> - Employee creates a request (Draft)
> - Employee submits → status becomes Pending
> - Manager reviews: can Approve or Reject
> - If Rejected, employee can Edit and resubmit (back to Pending)
> - If Approved, HR processes it → Completed
> - Employee can Cancel a Draft or Pending request at any time

**User Responses:**
1. Workflow selection → "D"
2. State confirmation → confirm states
3. Transition confirmation → confirm transitions
4. Real-world data → "Most requests are approved. About 10% rejected, usually resubmitted once. Cancellations are rare."

**Evaluation Criteria:**

Step 1 — States:
- [ ] Identifies: Draft, Pending, Approved, Rejected, Completed, Cancelled
- [ ] Marks Draft as Initial
- [ ] Marks Completed and Cancelled as Final states

Step 2 — Transitions:
- [ ] Draft → Pending (Submit)
- [ ] Pending → Approved (Approve)
- [ ] Pending → Rejected (Reject)
- [ ] Rejected → Pending (Edit & Resubmit) — loop detected
- [ ] Approved → Completed (Process)
- [ ] Draft → Cancelled (Cancel)
- [ ] Pending → Cancelled (Cancel)
- [ ] Does NOT add transitions not stated in requirement (e.g., Approved → Cancelled)

Step 3 — Diagrams:
- [ ] State diagram shows the loop (Rejected → Pending)
- [ ] State diagram shows branching (Pending → Approved / Rejected)
- [ ] Matrix shows Cancel is valid from Draft and Pending but not from other states

Step 4 — Test scenarios:
- [ ] Valid transitions: all 7 transitions listed above
- [ ] Invalid transitions: at least tests for Completed→Draft, Cancelled→Pending, Approved→Rejected
- [ ] Path tests: Happy path (Draft→Pending→Approved→Completed), Rejection+resubmit path, Cancel path

---

### EVAL-ST03: Minimal two-state toggle

**Requirement:**
> Feature toggle: a feature can be Enabled or Disabled. Admin can toggle between states.

**User Responses:**
1. Workflow selection → "D"
2. State confirmation → confirm 2 states
3. Transition confirmation → confirm
4. Real-world data → "Features are toggled a few times during rollout. Usually enabled permanently after testing."

**Evaluation Criteria:**
- [ ] Identifies 2 states: Enabled, Disabled
- [ ] Neither marked as Final (both can transition to each other)
- [ ] 2 valid transitions: Enabled→Disabled, Disabled→Enabled
- [ ] Matrix is 2×2 with both cells showing valid transitions
- [ ] No invalid transitions possible (every combination is valid)
- [ ] Path test shows toggle sequence (e.g., Disabled→Enabled→Disabled→Enabled)

---

### EVAL-ST04: States with guard conditions

**Requirement:**
> Payment status: Unpaid → Paid (when full amount received), Unpaid → Partially Paid
> (when partial amount received), Partially Paid → Paid (when remaining amount received),
> Paid → Refunded (when refund requested within 30 days).
> Refunded is final.

**User Responses:**
1. Workflow selection → "D"
2. State confirmation → confirm
3. Transition confirmation → confirm (may ask about guard conditions)
4. Real-world data → "Most payments are full upfront. Partial payments happen for large orders. Refunds are about 5%."

**Evaluation Criteria:**
- [ ] Identifies 4 states: Unpaid, Partially Paid, Paid, Refunded
- [ ] Captures guard conditions in transitions (e.g., "full amount", "partial amount", "within 30 days")
- [ ] Transition events include the conditions that trigger them
- [ ] Invalid transition tests include: Refunded→Paid, Partially Paid→Refunded (no direct refund)
- [ ] Does NOT assume Paid→Unpaid is valid (not stated in requirement)
