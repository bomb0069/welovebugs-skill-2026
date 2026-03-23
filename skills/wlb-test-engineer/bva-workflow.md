# Workflow A: Boundary Value Analysis (BVA)

**Use when**: requirement has **numeric input fields** with defined ranges (age, price,
quantity, weight, etc.)

Work through each step in order. **Only pause and wait for user input when a step
is marked with ⏸.** For all other steps, continue automatically.

## Step 1: Understand the requirement and identify numeric inputs

Analyze the requirement and extract:

1. **What the feature does** — one-sentence summary
2. **Numeric input fields** — list every field that accepts a numeric value
3. **Rules for each numeric field** — valid range, min, max, special thresholds

Present this as a table:

```markdown
| Field    | Type    | Min | Max | Special rules              |
|----------|---------|-----|-----|----------------------------|
| age      | integer | 18  | 65  | Under 18 rejected          |
| quantity | integer | 1   | 100 | Over 100 requires approval |
```

If the requirement is missing range information, ask the user before proceeding.

## Step 2: Find boundary values ⏸ Wait for user input

For each numeric field, identify the boundary points. Use the standard BVA approach:

- **On the boundary** — the exact min and max values
- **Just below the boundary** — one unit below min, one unit below max
- **Just above the boundary** — one unit above min, one unit above max

**Before calculating boundaries, clarify what "one unit" means for each field.**
The meaning of ±1 depends on how the system actually handles the data. Ask the user
to confirm the smallest meaningful increment for each numeric field.

**Present numbered options for the user to choose from.** Generate the options
dynamically based on the field type. Always include a manual option as the last choice.

For each numeric field, present a question like this:

```
What does "one unit" mean for the **age** field?

  1 → 1 year (system checks whole years only)
  2 → 1 day (system checks exact birth date)
  3 → Enter manually: ___

Please reply with the number (e.g., "1"):
```

Another example for a price field:

```
What does "one unit" mean for the **price** field?

  1 → 1 (whole numbers only)
  2 → 0.1 (1 decimal place)
  3 → 0.01 (2 decimal places, e.g., currency)
  4 → Enter manually: ___

Please reply with the number (e.g., "3"):
```

Another example for a weight field:

```
What does "one unit" mean for the **weight** field?

  1 → 1 kg (whole numbers only)
  2 → 0.1 kg (1 decimal place)
  3 → 0.01 kg (2 decimal places)
  4 → 0.001 kg (3 decimal places)
  5 → Enter manually: ___

Please reply with the number (e.g., "2"):
```

When there are multiple numeric fields, present all questions together so the user
can answer them in one reply (e.g., "age=1, price=3").

**Do not assume. Always ask the user to confirm the unit before generating boundary values.**

Once the unit is confirmed, present the boundaries:

```markdown
| Field | Unit | Below Min | Min (on) | Above Min | Below Max | Max (on) | Above Max |
|-------|------|-----------|----------|-----------|-----------|----------|-----------|
| age   | year | 17        | 18       | 19        | 64        | 65       | 66        |
| price | 0.01 | 0.99      | 1.00     | 1.01      | 99.99     | 100.00   | 100.01    |
```

Then, for **each numeric field**, generate a visual boundary diagram showing
the full range from min to max in a single connected graph. This helps the user
see all boundary points at a glance.

When the field has **both min and max** boundaries, draw one diagram with all 6 points:

```
        Requirement: age must be between 18 and 65

  [✗ Invalid]  [✓ Valid]  [✓ Valid]           [✓ Valid]  [✓ Valid]  [✗ Invalid]
       ↑            ↑          ↑                   ↑          ↑          ↑
       |            |          |                   |          |          |
  ◄─── 17 ──────── 18 ─────── 19 ─── ··· ──────── 64 ─────── 65 ─────── 66 ───►
     Below Min   Min (on)  Above Min           Below Max  Max (on)  Above Max
```

When the field has **only one boundary** (min-only or max-only), draw a single-side diagram:

```
        Requirement: deposit max 30,000.00 per person (unit = 100)

                      [✓ Valid]       [✓ Valid]      [✗ Invalid]
                           ↑               ↑              ↑
                           |               |              |
                ◄───── 29,900.00 ──── 30,000.00 ──── 30,100.00 ───►
                        Below Max        Max (on)      Above Max
```

Generate one diagram per numeric field. Use the same language as the requirement
(e.g., if the requirement is in Thai, use Thai labels like
✓ ได้ / ✗ ไม่ได้ instead of ✓ Valid / ✗ Invalid).

## Step 3: Select test possibilities

Combine boundary values with their expected classification:

- **Valid** — value falls inside the accepted range
- **Invalid** — value falls outside the accepted range

Present each possibility:

```markdown
| # | Field | Value | Position    | Valid/Invalid |
|---|-------|-------|-------------|---------------|
| 1 | age   | 17    | Below Min   | Invalid       |
| 2 | age   | 18    | Min (on)    | Valid         |
| 3 | age   | 19    | Above Min   | Valid         |
| 4 | age   | 64    | Below Max   | Valid         |
| 5 | age   | 65    | Max (on)    | Valid         |
| 6 | age   | 66    | Above Max   | Invalid       |
```

## Step 4: Generate expected outputs

For each possibility, determine the expected system behavior based on the
requirement's business rules. Be specific — don't just say "success" or "error",
describe what the system should do.

```markdown
| # | Field | Value | Valid/Invalid | Expected Output                          |
|---|-------|-------|---------------|------------------------------------------|
| 1 | age   | 17    | Invalid       | Reject with message "Age must be 18-65"  |
| 2 | age   | 18    | Valid         | Accept, proceed to next step              |
| 3 | age   | 19    | Valid         | Accept, proceed to next step              |
| 4 | age   | 64    | Valid         | Accept, proceed to next step              |
| 5 | age   | 65    | Valid         | Accept, proceed to next step              |
| 6 | age   | 66    | Invalid       | Reject with message "Age must be 18-65"  |
```

## Step 5: Collect real-world representative data ⏸ Wait for user input (REQUIRED)

Boundary values from Step 2-4 are ideal for **unit testing** (developer tests the
exact boundary logic in code). But real users don't typically use boundary values
in their daily work. For **acceptance testing** (end-to-end business process), we
need realistic data that represents how customers actually behave.

**This step is critical. Do NOT skip it. Do NOT proceed to Step 6 until the user
provides real-world data or explicitly chooses to skip.**

For each boundary position, ask the user: **"What value would a real customer
typically use in this situation?"**

Present the request as a table the user must fill in:

```markdown
Please provide real-world values for each position:

| Position   | Boundary Value | Real-world Value | Source / Reason |
|------------|---------------|------------------|-----------------|
| Below Min  | 17            | ❓ Please provide | ❓               |
| Min (on)   | 18            | ❓ Please provide | ❓               |
| Above Min  | 19            | ❓ Please provide | ❓               |
| Below Max  | 64            | ❓ Please provide | ❓               |
| Max (on)   | 65            | ❓ Please provide | ❓               |
| Above Max  | 66            | ❓ Please provide | ❓               |
```

Help the user think about it with guiding questions:
- "What is the most common value your customers use? (from statistics, reports, or experience)"
- "What value would a typical customer enter in this field?"
- "When customers exceed the limit, what value do they usually try?"
- "Do you have any analytics, logs, or business reports showing common values?"

**If the user says they don't know or wants to skip**, ask one more time:
> "Real-world data is important because acceptance tests should reflect how
> customers actually use the system. Can you check with your team, product owner,
> or look at any usage reports?"

**If the user still chooses to skip**, accept it but mark the missing data clearly.
The acceptance test table in Step 6 will show a placeholder for every value the user
did not provide:

| Position   | Boundary Value | Real-world Value                          | Source / Reason                |
|------------|---------------|-------------------------------------------|--------------------------------|
| Below Max  | 29,900.00     | ⚠️ TBD — awaiting real-world data from team | No data provided yet           |
| Max (on)   | 30,000.00     | 30,000.00                                 | Same as boundary               |
| Above Max  | 30,100.00     | ⚠️ TBD — awaiting real-world data from team | No data provided yet           |

Example when user provides data — Requirement: deposit max 30,000.00 per person:

| Position   | Boundary Value | Real-world Value | Source / Reason                          |
|------------|---------------|------------------|------------------------------------------|
| Below Max  | 29,900.00     | 500.00           | Most common deposit from customer stats  |
| Max (on)   | 30,000.00     | 30,000.00        | Customers who deposit the full max       |
| Above Max  | 30,100.00     | 50,000.00        | Attempted large deposit, common rejection|

## Step 6: Generate final test case tables

Produce **two separate tables** using the template from `test-case-template.md`.

Both tables use **two-level column headers**:
- **Level 1** (top): repeats `Input` across input columns and `Output` across output columns
- **Level 2**: individual field names under each group

Determine input and output field names dynamically from the requirement.

---

### Table 1: Technique-based test cases (for Unit Testing)

These use the **exact boundary values** from BVA analysis. Purpose: guide developers
to write unit tests that verify the boundary logic in their code.

Label this table clearly: **"Unit Test Cases (BVA Technique)"**

```markdown
#### Unit Test Cases (BVA Technique)

| TC-ID | Test Case Name            | Test Description                                                          | Input  | Input            | Output | Output                                 | Type    |
|       |                           |                                                                           | name   | deposit          | Result | Message                                |         |
|-------|---------------------------|---------------------------------------------------------------------------|--------|------------------|--------|-----------------------------------------|---------|
| UT-01 | Deposit at max boundary   | Deposit 30,000.00 and system should accept the transaction                | John   | 30,000.00        | Accept | Transaction completed                  | Valid   |
| UT-02 | Deposit below max         | Deposit 29,900.00 and system should accept the transaction                | John   | 29,900.00        | Accept | Transaction completed                  | Valid   |
| UT-03 | Deposit above max         | Deposit 30,100.00 and system should reject with over-limit error          | John   | 30,100.00        | Reject | "Deposit cannot exceed 30,000.00"      | Invalid |
```

Number these as UT-01, UT-02, etc.

---

### Table 2: Business-based test cases (for Acceptance Testing)

These use **real-world representative data** from Step 5. Purpose: validate the
end-to-end business process with data that reflects actual customer behavior.

Label this table clearly: **"Acceptance Test Cases (Business Scenarios)"**

**When the user provided real-world data:**

```markdown
#### Acceptance Test Cases (Business Scenarios)

| TC-ID | Test Case Name            | Test Description                                                          | Input      | Input            | Output | Output                                 | Type    |
|       |                           |                                                                           | name       | deposit          | Result | Message                                |         |
|-------|---------------------------|---------------------------------------------------------------------------|------------|------------------|--------|-----------------------------------------|---------|
| AT-01 | Typical customer deposit  | Customer deposits 500.00 (common amount) and system should accept         | Sarah Lee  | 500.00           | Accept | Transaction completed, receipt shown   | Valid   |
| AT-02 | Full max deposit          | Customer deposits 30,000.00 (full limit) and system should accept         | Tom Park   | 30,000.00        | Accept | Transaction completed, receipt shown   | Valid   |
| AT-03 | Over-limit deposit        | Customer deposits 50,000.00 (common large attempt) and system should reject| Amy Chen  | 50,000.00        | Reject | "Deposit cannot exceed 30,000.00"      | Invalid |
```

**When the user skipped providing real-world data**, use `⚠️ TBD` as the value
and add a note row. This makes it visible that the test case is incomplete:

```markdown
#### Acceptance Test Cases (Business Scenarios)

> ⚠️ **Some test data is pending.** Values marked "TBD" need real-world data
> from the team before these test cases can be executed.

| TC-ID | Test Case Name            | Test Description                                                                      | Input      | Input                              | Output | Output                                 | Type    |
|       |                           |                                                                                       | name       | deposit                            | Result | Message                                |         |
|-------|---------------------------|---------------------------------------------------------------------------------------|------------|------------------------------------|--------|-----------------------------------------|---------|
| AT-01 | Typical customer deposit  | Customer deposits ⚠️ TBD and system should accept                                    | Sarah Lee  | ⚠️ TBD (awaiting real-world data)  | Accept | Transaction completed, receipt shown   | Valid   |
| AT-02 | Full max deposit          | Customer deposits 30,000.00 (full limit) and system should accept                     | Tom Park   | 30,000.00                          | Accept | Transaction completed, receipt shown   | Valid   |
| AT-03 | Over-limit deposit        | Customer deposits ⚠️ TBD and system should reject with over-limit error               | Amy Chen   | ⚠️ TBD (awaiting real-world data)  | Reject | "Deposit cannot exceed 30,000.00"      | Invalid |
```

Number these as AT-01, AT-02, etc.
