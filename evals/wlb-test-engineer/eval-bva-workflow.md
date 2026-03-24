# Eval: BVA Workflow (Workflow A)

Tests for Boundary Value Analysis workflow — single numeric input with range.

---

### EVAL-BVA01: Basic age range with integer boundaries

**Requirement:**
> Users must be between 18 and 65 years old to create an account.

**User Responses:**
1. Workflow selection → "A"
2. One unit confirmation → "1" (1 year)
3. Real-world data → "Typical users are 25, 35, and 50. Underage attempts happen from 15-year-olds. Some elderly users are 70."

**Evaluation Criteria:**

Step 1 — Extract:
- [ ] Identifies field: age
- [ ] Identifies range: 18–65
- [ ] Identifies type: integer

Step 2 — Boundary values:
- [ ] Asks user to confirm "one unit" before generating boundaries
- [ ] Offers numbered options (e.g., 1→ 1 year, 2→ 1 month, etc.)
- [ ] After confirmation, generates 6 boundary points: 17, 18, 19, 64, 65, 66

Step 3 — Boundary diagram:
- [ ] Produces ASCII visual boundary diagram
- [ ] Diagram shows valid region (18–65) and invalid regions
- [ ] All 6 boundary points are plotted on the diagram
- [ ] Boundary labels match the numeric values

Step 4 — Test possibilities:
- [ ] Lists all boundary value test cases with expected classification (Valid/Invalid)
- [ ] 17 → Invalid, 18 → Valid, 19 → Valid, 64 → Valid, 65 → Valid, 66 → Invalid

Step 5 — Expected outputs:
- [ ] Each test case has a specific behavior description (not just "pass/fail")
- [ ] Valid cases describe success behavior (e.g., "account created successfully")
- [ ] Invalid cases describe rejection behavior with error message

Step 6 — Real-world data collection:
- [ ] Pauses (⏸) to ask for real-world data
- [ ] Asks guiding questions about typical user ages

Step 7 — Final tables:
- [ ] **Unit Test Cases table** with IDs UT-01, UT-02, ... using boundary values
- [ ] **Acceptance Test Cases table** with IDs AT-01, AT-02, ... using user-provided real-world data (25, 35, 50, 15, 70)
- [ ] Both tables use two-level column headers (Level 1: Input/Output, Level 2: field names)
- [ ] Each row has: TC-ID, Test Case Name, Test Description, input columns, output columns, Type
- [ ] Type column shows Valid or Invalid correctly
- [ ] Test Descriptions are human-readable sentences

---

### EVAL-BVA02: Decimal range with currency

**Requirement:**
> The minimum deposit amount is 100.00 THB and the maximum is 50,000.00 THB.

**User Responses:**
1. Workflow selection → "A"
2. One unit confirmation → "1" (0.01 THB)
3. Real-world data → "Most customers deposit 500, 1000, or 5000 THB. Some try small amounts like 50 THB."

**Evaluation Criteria:**

Step 2 — Boundary values:
- [ ] Recognizes currency precision (2 decimal places)
- [ ] Offers 0.01 as a unit option
- [ ] Generates boundary points: 99.99, 100.00, 100.01, 49999.99, 50000.00, 50000.01

Step 3 — Boundary diagram:
- [ ] Diagram correctly shows the decimal boundary points
- [ ] Valid region is 100.00–50,000.00

Step 7 — Final tables:
- [ ] UT table uses exact boundary values (99.99, 100.00, etc.)
- [ ] AT table uses real-world values (500, 1000, 5000, 50)
- [ ] Output columns describe specific system behaviors for deposits

---

### EVAL-BVA03: Single-sided boundary (minimum only)

**Requirement:**
> Order quantity must be at least 1 item. There is no maximum limit.

**User Responses:**
1. Workflow selection → "A"
2. One unit confirmation → "1" (1 item)
3. Real-world data → "Customers usually order 2-5 items. Bulk orders can be 100+."

**Evaluation Criteria:**
- [ ] Identifies only a lower boundary (min = 1)
- [ ] Generates boundary points for lower bound: 0, 1, 2
- [ ] Does NOT fabricate an upper boundary
- [ ] May ask if there is a practical/system maximum, but does not assume one
- [ ] Boundary diagram shows open-ended valid region on the upper side

---

### EVAL-BVA04: Skip real-world data (pushback behavior)

**Requirement:**
> Passengers aged 2–11 qualify for child fare.

**User Responses:**
1. Workflow selection → "A"
2. One unit confirmation → "1" (1 year)
3. Real-world data → "skip"
4. Second pushback → "skip" (insist)

**Evaluation Criteria:**
- [ ] First skip attempt: skill pushes back and asks again for real-world data
- [ ] Second skip attempt: skill accepts but marks AT table cells with ⚠️ TBD
- [ ] AT table preamble includes a note about missing real-world data
- [ ] UT table is still complete with boundary values (1, 2, 3, 10, 11, 12)
