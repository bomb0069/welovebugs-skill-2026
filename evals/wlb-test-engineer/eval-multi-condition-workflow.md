# Eval: Multi-Condition Workflow (Workflow C)

Tests for Multi-Condition Analysis — 2+ conditions with Decision Table or Sequential strategies.

---

### EVAL-MC01: Two conditions — Decision Table strategy

**Requirement:**
> Discount eligibility: customer must be a Gold or Platinum member and purchase amount
> must be at least 1,000 THB. Gold members get 10% discount, Platinum gets 15%.
> Non-qualifying members or purchases under 1,000 THB get no discount.

**User Responses:**
1. Workflow selection → "C"
2. Condition confirmation → confirm 2 conditions
3. Strategy selection → "Decision Table"
4. Real-world data (per condition):
   - Membership: "Most customers are Gold. We have about 20% Platinum."
   - Purchase amount: "Average purchase is 2,500 THB. Small purchases around 300 THB happen often."

**Evaluation Criteria:**

Step 1 — Conditions:
- [ ] Identifies C1: membership type (EP) and C2: purchase amount (BVA)
- [ ] Shows condition breakdown table with technique assignment
- [ ] Pauses (⏸) for user confirmation

Step 2 — Per-condition analysis:
- [ ] **C1 (EP)**: identifies partitions — Valid: {Gold, Platinum}, Invalid: {Silver, Basic, etc.}
- [ ] **C1 Unit Tests**: uses IDs C1-UT01, C1-UT02, ... (NOT UT-01)
- [ ] **C2 (BVA)**: identifies boundary at 1,000 THB, generates boundary points
- [ ] **C2 Unit Tests**: uses IDs C2-UT01, C2-UT02, ...
- [ ] Produces Test Value Summary table collecting all values from both conditions

Step 3 — Decision Table:
- [ ] Step 3a: identifies condition states (V/I for each condition)
- [ ] Step 3b: generates Decision Table with 2^2 = 4 rules (R1–R4)
- [ ] Table shows: R1 (V,V), R2 (V,I), R3 (I,V), R4 (I,I)
- [ ] Each rule has expected outcome
- [ ] Step 3c: expands with actual test values from per-condition analysis

Step 4 — Real-world data:
- [ ] Pauses (⏸) to collect real-world data
- [ ] Asks separately for each condition

Step 5 — Final merged tables:
- [ ] **Test Scenarios** with IDs TS-01, TS-02, ... (NOT UT-01 or C1-UT01)
- [ ] **Acceptance Test Scenarios** with IDs ATS-01, ATS-02, ...
- [ ] Both tables include ALL condition columns (membership + amount)
- [ ] Both tables include **Technique** column (e.g., "C1:EP Gold", "C2:BVA Min")
- [ ] Both tables include **Rule** column (R1, R2, R3, R4)
- [ ] TS table uses technique values; ATS table uses real-world values
- [ ] Each scenario varies ONE condition while others use typical valid value

---

### EVAL-MC02: Three conditions — Decision Table strategy

**Requirement:**
> Insurance eligibility: applicant must be 25–60 years old, annual income at least
> 500,000 THB, and health status must be "Healthy" or "Minor Condition". Applicants
> with "Serious Condition" are rejected regardless of other criteria.

**User Responses:**
1. Workflow selection → "C"
2. Condition confirmation → confirm 3 conditions
3. Strategy selection → "Decision Table"
4. Real-world data → provide for each condition

**Evaluation Criteria:**

Step 1 — Conditions:
- [ ] C1: age (numeric → BVA), range 25–60
- [ ] C2: income (numeric → BVA), min 500,000
- [ ] C3: health status (non-numeric → EP), {Healthy, Minor Condition} vs {Serious Condition}

Step 3 — Decision Table:
- [ ] Generates 2^3 = 8 rules (R1–R8)
- [ ] All 8 combinations of V/I across 3 conditions are present
- [ ] Expected outcomes correct: only R1 (all valid) results in approval

Step 5 — Final tables:
- [ ] Technique column shows which condition is being varied (e.g., "C1:BVA Below Min")
- [ ] All three condition input columns present in every row

---

### EVAL-MC03: Two conditions — Sequential strategy

**Requirement:**
> Withdrawal rules: customer must have an active account status, and withdrawal
> amount must be between 100 and 50,000 THB per transaction.

**User Responses:**
1. Workflow selection → "C"
2. Condition confirmation → confirm 2 conditions
3. Strategy selection → "Sequential"
4. Condition order → "1. Account status, 2. Amount" (check status first)
5. Real-world data → provide for each condition

**Evaluation Criteria:**

Step 3a — Sequence order:
- [ ] Asks user to specify which condition is checked first
- [ ] Pauses (⏸) for confirmation

Step 3b — Sequential flowchart:
- [ ] Draws flowchart showing gates: first check account status, then check amount
- [ ] Invalid at gate 1 → stops (amount not checked)
- [ ] Valid at gate 1 → proceeds to gate 2

Step 3c — Sequential test scenarios:
- [ ] Scenario for invalid account status: amount column shows "—" (not checked)
- [ ] Scenario for valid account + invalid amount: shows actual invalid amount
- [ ] Scenario for valid account + valid amount: shows both valid values

Step 5 — Final tables:
- [ ] Include **Reached** column (instead of Rule column)
- [ ] Reached shows which gate the test reached (e.g., "Gate 1: Fail", "Gate 2: Pass")
- [ ] Invalid-at-gate-1 rows use "—" for unchecked condition columns

---

### EVAL-MC04: Conditions with all-numeric inputs

**Requirement:**
> Parcel shipping rate: weight must be 0.1–30 kg, and longest dimension must be
> 10–150 cm. Parcels outside either range are rejected.

**User Responses:**
1. Workflow selection → "C"
2. Condition confirmation → confirm 2 conditions
3. Strategy selection → "Decision Table"
4. Real-world data → "Typical parcels: 2 kg / 30 cm, 5 kg / 60 cm. Oversized: 35 kg or 200 cm."

**Evaluation Criteria:**
- [ ] Both conditions classified as BVA (both numeric with ranges)
- [ ] C1 (weight): boundary points at 0.09, 0.1, 0.11, 29.99, 30.0, 30.01
- [ ] C2 (dimension): boundary points at 9, 10, 11, 149, 150, 151
- [ ] Per-condition unit tests use C1-UTxx and C2-UTxx naming
- [ ] Decision table has 4 rules for 2 conditions
- [ ] Merged TS table varies one condition at a time

---

### EVAL-MC05: User changes strategy mid-workflow

**Requirement:**
> Free trial eligibility: user must be a new customer (no previous account) and
> must select either Basic or Standard plan (Premium is excluded from trial).

**User Responses:**
1. Workflow selection → "C"
2. Condition confirmation → confirm
3. Strategy selection → "Sequential"
4. (After seeing flowchart) → user requests to switch to "Decision Table" instead

**Evaluation Criteria:**
- [ ] Skill accommodates strategy change without restarting per-condition analysis
- [ ] Regenerates from Step 3 with Decision Table approach
- [ ] Per-condition unit tests (C1-UTxx, C2-UTxx) remain unchanged
- [ ] Final tables use Rule column (not Reached column)
