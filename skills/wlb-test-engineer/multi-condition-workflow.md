# Workflow C: Multi-Condition Analysis

**Use when**: requirement has **multiple conditions** where each condition involves
a different input field, and those fields may be a mix of numeric (BVA) and
non-numeric (EP) data.

Example requirement:
> "Customers aged 18-65 with Gold or Silver membership can apply for a personal loan
> between 10,000 and 1,000,000 baht."

This has 3 conditions:
- age (18-65) → numeric → BVA
- membership (Gold, Silver) → category → EP
- loan amount (10,000-1,000,000) → numeric → BVA

Work through each step in order. **Only pause and wait for user input when a step
is marked with ⏸.** For all other steps, continue automatically.

## Step 1: Break requirement into conditions ⏸ Wait for user input

Analyze the requirement and decompose it into **individual conditions**.
Each condition is one input field with its own rule.

Present the breakdown as a table and **ask the user to confirm** the conditions
are correct and complete:

```markdown
I found the following conditions in this requirement:

| #  | Condition          | Field      | Type        | Rule                     | Technique |
|----|--------------------|------------|-------------|--------------------------|-----------|
| C1 | Age restriction    | age        | integer     | Must be 18-65            | BVA       |
| C2 | Membership type    | membership | category    | Gold or Silver only      | EP        |
| C3 | Loan amount        | amount     | decimal     | 10,000 - 1,000,000 baht  | BVA       |

Is this correct? Are there any conditions I missed?
```

The **Technique** column is auto-assigned:
- Numeric fields with ranges → **BVA**
- Non-numeric fields with categories/groups → **EP**

If a field is ambiguous, ask the user which technique to apply.

## Step 2: Analyze each condition with its technique

For each condition, run the appropriate technique's analysis steps
(from `bva-workflow.md` or `ep-workflow.md`). Present all analyses together,
grouped by condition.

**For BVA conditions** — run Steps 1-4 from `bva-workflow.md`:
- Clarify "one unit" ⏸ (ask with numbered options)
- Find boundary values
- Generate boundary diagram
- List test possibilities with expected outputs

**For EP conditions** — run Steps 1-4 from `ep-workflow.md`:
- Identify equivalence partitions ⏸ (ask user to confirm)
- Generate partition set diagram
- Select representative values with expected outputs

Present each condition's analysis under its own heading:

```markdown
### C1: Age restriction (BVA)

What does "one unit" mean for the **age** field?

  1 → 1 year (system checks whole years only)
  2 → 1 day (system checks exact birth date)
  3 → Enter manually: ___
```

After user confirms units for all BVA conditions, present all analyses:

```markdown
### C1: Age restriction (BVA)

| Field | Unit | Below Min | Min (on) | Above Min | Below Max | Max (on) | Above Max |
|-------|------|-----------|----------|-----------|-----------|----------|-----------|
| age   | year | 17        | 18       | 19        | 64        | 65       | 66        |

  [✗ Invalid]  [✓ Valid]  [✓ Valid]           [✓ Valid]  [✓ Valid]  [✗ Invalid]
       ↑            ↑          ↑                   ↑          ↑          ↑
       |            |          |                   |          |          |
  ◄─── 17 ──────── 18 ─────── 19 ─── ··· ──────── 64 ─────── 65 ─────── 66 ───►
     Below Min   Min (on)  Above Min           Below Max  Max (on)  Above Max

... (boundary diagram)


### C2: Membership type (EP)

... (partition analysis, set diagram)


### C3: Loan amount (BVA)

... (boundary analysis, diagram)
```

After all conditions are analyzed, create a **Test Value Summary** that collects
every test value from BVA and EP into one reference table. This is the pool of
values that will be used in the combined test scenario table.

```markdown
#### Test Value Summary

| Condition  | Technique | Position / Partition   | Value       | Valid/Invalid |
|------------|-----------|------------------------|-------------|---------------|
| C1: age    | BVA       | Below Min              | 17          | Invalid       |
| C1: age    | BVA       | Min (on)               | 18          | Valid         |
| C1: age    | BVA       | Above Min              | 19          | Valid         |
| C1: age    | BVA       | Below Max              | 64          | Valid         |
| C1: age    | BVA       | Max (on)               | 65          | Valid         |
| C1: age    | BVA       | Above Max              | 66          | Invalid       |
| C2: member | EP        | Premium membership     | Gold        | Valid         |
| C2: member | EP        | Premium membership     | Silver      | Valid         |
| C2: member | EP        | Basic membership       | Bronze      | Invalid       |
| C2: member | EP        | Unknown membership     | (blank)     | Invalid       |
| C3: amount | BVA       | Below Min              | 9,900       | Invalid       |
| C3: amount | BVA       | Min (on)               | 10,000      | Valid         |
| C3: amount | BVA       | Above Min              | 10,100      | Valid         |
| C3: amount | BVA       | Below Max              | 999,900     | Valid         |
| C3: amount | BVA       | Max (on)               | 1,000,000   | Valid         |
| C3: amount | BVA       | Above Max              | 1,000,100   | Invalid       |
```

This summary is used in Step 3 to build combined test scenarios and in Step 5
to generate the final test case tables with **all** technique-derived values
(not just simplified valid/invalid).

## Step 3: Choose combination strategy ⏸ Wait for user input

After analyzing each condition individually, you need to **combine** the conditions
into test scenarios. There are two strategies — ask the user to choose:

```
How should we combine the conditions into test scenarios?

  1 → Decision Table (all combinations)
      Tests every combination of valid/invalid across all conditions (2^N rules).
      More thorough, but produces more test cases.
      Best when: conditions are independent, system validates all at once.
      Example: 3 conditions → 8 rules → ~12 test cases

  2 → Sequential Condition (fail-fast)
      Checks conditions in a specific order. If one fails, the system stops
      and does NOT check the remaining conditions.
      Fewer test cases, more realistic for systems with step-by-step validation.
      Best when: system validates in order (e.g., check age first, then membership, then amount).
      Example: 3 conditions → ~8 test cases

Please reply with 1 or 2:
```

Then follow the chosen strategy below.

---

### Strategy 1: Decision Table

#### 3a: Identify condition states

For each condition, simplify to **Valid (V)** and **Invalid (I)** states:

```markdown
| Condition  | Valid (V)                          | Invalid (I)                        |
|------------|------------------------------------|------------------------------------|
| C1: age    | 18-65 (e.g., 18, 30, 65)          | <18 or >65 (e.g., 17, 66)         |
| C2: member | Gold, Silver                      | Bronze, blank, unknown             |
| C3: amount | 10,000-1,000,000 (e.g., 500,000)  | <10,000 or >1,000,000 (e.g., 9,900)|
```

#### 3b: Generate Decision Table

List all combinations. With N conditions, there are 2^N combinations.
For each combination, determine the **expected outcome** based on the requirement.

```markdown
#### Decision Table

| Rule | C1: age | C2: membership | C3: amount | Expected Outcome                    |
|------|---------|----------------|------------|-------------------------------------|
| R1   | V       | V              | V          | ✓ Loan application accepted         |
| R2   | V       | V              | I          | ✗ Reject: amount out of range       |
| R3   | V       | I              | V          | ✗ Reject: membership not eligible   |
| R4   | V       | I              | I          | ✗ Reject: membership not eligible   |
| R5   | I       | V              | V          | ✗ Reject: age out of range          |
| R6   | I       | V              | I          | ✗ Reject: age out of range          |
| R7   | I       | I              | V          | ✗ Reject: age out of range          |
| R8   | I       | I              | I          | ✗ Reject: age out of range          |
```

#### 3c: Expand Decision Table with test data

Replace V/I with actual test values from Step 2. For BVA conditions, use
the boundary values. For EP conditions, use representative values.

Each rule may generate **multiple test cases** because each invalid condition
can fail in different ways (e.g., below min vs above max).

```markdown
#### Expanded Decision Table

| Rule | Scenario                              | age | membership | amount    | Expected Outcome                           |
|------|---------------------------------------|-----|------------|-----------|---------------------------------------------|
| R1a  | All valid (min boundaries)            | 18  | Gold       | 10,000    | ✓ Loan application accepted                |
| R1b  | All valid (max boundaries)            | 65  | Silver     | 1,000,000 | ✓ Loan application accepted                |
| R1c  | All valid (mid values)                | 30  | Gold       | 500,000   | ✓ Loan application accepted                |
| R2a  | Valid age+member, amount below min    | 30  | Gold       | 9,900     | ✗ Reject: "Amount below minimum 10,000"    |
| R2b  | Valid age+member, amount above max    | 30  | Gold       | 1,000,100 | ✗ Reject: "Amount exceeds maximum 1,000,000"|
| R3a  | Valid age+amount, invalid membership  | 30  | Bronze     | 500,000   | ✗ Reject: "Membership not eligible"        |
| R5a  | Invalid age (below), valid rest       | 17  | Gold       | 500,000   | ✗ Reject: "Age must be between 18 and 65"  |
| R5b  | Invalid age (above), valid rest       | 66  | Gold       | 500,000   | ✗ Reject: "Age must be between 18 and 65"  |
| R8a  | All invalid                           | 17  | Bronze     | 9,900     | ✗ Reject: "Age must be between 18 and 65"  |
```

Focus on:
- **R1** (all valid): test with min boundaries, max boundaries, and typical values
- **Rules with one invalid condition**: test each way the condition can fail
- **Rules with multiple invalid conditions**: one representative test case each

---

### Strategy 2: Sequential Condition (fail-fast)

#### 3a: Choose condition sequence ⏸ Wait for user input

The system checks conditions **in order**. If one fails, it stops immediately
and does not check the rest. Ask the user which sequence makes sense:

```
In what order does the system check these conditions?

  1 → age → membership → amount (check personal info first, then financial)
  2 → membership → age → amount (check eligibility first)
  3 → amount → age → membership (check financial first)
  4 → Other order: ___

Please reply with the number:
```

#### 3b: Draw condition sequence flowchart

After the user confirms the sequence, draw a **flowchart** showing the
validation flow. This makes it clear that each condition is a gate —
if it fails, the process stops.

Example with sequence: age → membership → amount

```
        Sequential Condition Flowchart

                    ┌─────────┐
                    │  START  │
                    └────┬────┘
                         │
                         ▼
                ┌─────────────────┐
                │  C1: Check age  │
                │   (18-65?)      │
                └────────┬────────┘
                         │
                    ┌────┴────┐
                    │         │
               ✗ Invalid   ✓ Valid
                    │         │
                    ▼         ▼
            ┌──────────┐  ┌─────────────────────┐
            │  REJECT  │  │  C2: Check membership│
            │  age     │  │   (Gold/Silver?)      │
            │  error   │  └──────────┬────────────┘
            └──────────┘             │
                                ┌────┴────┐
                                │         │
                           ✗ Invalid   ✓ Valid
                                │         │
                                ▼         ▼
                        ┌──────────┐  ┌──────────────────┐
                        │  REJECT  │  │  C3: Check amount │
                        │  member  │  │   (10K-1M?)       │
                        │  error   │  └────────┬─────────┘
                        └──────────┘           │
                                          ┌────┴────┐
                                          │         │
                                     ✗ Invalid   ✓ Valid
                                          │         │
                                          ▼         ▼
                                  ┌──────────┐  ┌──────────┐
                                  │  REJECT  │  │  ACCEPT  │
                                  │  amount  │  │  Loan    │
                                  │  error   │  │  approved│
                                  └──────────┘  └──────────┘
```

#### 3c: Generate sequential test scenarios

With sequential checking, test scenarios are structured differently.
Each condition only needs to be tested when **all previous conditions passed**.

```markdown
#### Sequential Test Scenarios

| # | Scenario                        | C1: age | C2: membership | C3: amount | Reached | Expected Outcome                          |
|---|---------------------------------|---------|----------------|------------|---------|-------------------------------------------|
| 1 | C1 fails (below min)            | 17 ✗   | — (not checked)| — (not checked)| C1   | ✗ Reject: "Age must be between 18 and 65"|
| 2 | C1 fails (above max)            | 66 ✗   | — (not checked)| — (not checked)| C1   | ✗ Reject: "Age must be between 18 and 65"|
| 3 | C1 pass, C2 fails (invalid)     | 30 ✓   | Bronze ✗       | — (not checked)| C2   | ✗ Reject: "Membership not eligible"      |
| 4 | C1 pass, C2 fails (blank)       | 30 ✓   | (blank) ✗      | — (not checked)| C2   | ✗ Reject: "Membership required"          |
| 5 | C1+C2 pass, C3 fails (below)    | 30 ✓   | Gold ✓         | 9,900 ✗        | C3   | ✗ Reject: "Amount below minimum 10,000"  |
| 6 | C1+C2 pass, C3 fails (above)    | 30 ✓   | Gold ✓         | 1,000,100 ✗    | C3   | ✗ Reject: "Amount exceeds 1,000,000"     |
| 7 | All pass (min boundaries)       | 18 ✓   | Gold ✓         | 10,000 ✓       | END  | ✓ Loan application accepted              |
| 8 | All pass (max boundaries)       | 65 ✓   | Silver ✓       | 1,000,000 ✓    | END  | ✓ Loan application accepted              |
```

Key differences from Decision Table:
- **"— (not checked)"** means the system never evaluates this condition because
  a previous condition already failed
- **"Reached" column** shows how far the validation got
- **No combined invalid rows** — you never test C2+C3 invalid together because
  if C1 passes and C2 fails, C3 is never reached
- **Fewer test cases** — only N+1 paths instead of 2^N combinations

## Step 4: Collect real-world representative data ⏸ Wait for user input (REQUIRED)

Same rules as BVA/EP workflows. Ask the user for real-world values for **each condition**.

Present one table per condition:

```markdown
Please provide real-world values for each condition:

**C1: Age**
| Position   | Boundary Value | Real-world Value | Source / Reason |
|------------|---------------|------------------|-----------------|
| Below Min  | 17            | ❓ Please provide | ❓               |
| Valid      | 30            | ❓ Please provide | ❓               |
| Above Max  | 66            | ❓ Please provide | ❓               |

**C2: Membership**
| Partition          | Representative Value | Real-world Value | Source / Reason |
|--------------------|---------------------|------------------|-----------------|
| Premium membership | Gold                | ❓ Please provide | ❓               |
| Invalid membership | Bronze              | ❓ Please provide | ❓               |

**C3: Loan amount**
| Position   | Boundary Value | Real-world Value | Source / Reason |
|------------|---------------|------------------|-----------------|
| Below Min  | 9,900         | ❓ Please provide | ❓               |
| Valid      | 500,000       | ❓ Please provide | ❓               |
| Above Max  | 1,000,100     | ❓ Please provide | ❓               |
```

Same enforcement rules: push back if user tries to skip, mark `⚠️ TBD` if they
still choose to skip.

## Step 5: Generate final test case tables

Produce **two separate tables** based on the strategy chosen in Step 3.

The unit test table must use **every test value from the Test Value Summary** in
Step 2 — all BVA boundary points (6 per numeric field) and all EP representative
values (1 per partition). Each test case varies **one condition** using its
technique-derived values while keeping other conditions at a typical valid value.

Include a **Technique** column showing which technique (BVA/EP) and position
(e.g., Below Min, Min, Above Min) produced each test value.

---

### If Strategy 1 (Decision Table) was chosen:

#### Table 1: Unit Test Cases (Decision Table Technique)

Uses **all** boundary/partition values from the Test Value Summary.

Label: **"Unit Test Cases (Decision Table: BVA + EP)"**

```markdown
#### Unit Test Cases (Decision Table: BVA + EP)

| TC-ID | Rule | Technique     | Test Case Name                  | Test Description                                                              | Input | Input      | Input     | Output | Output                                   | Type    |
|       |      |               |                                 |                                                                               | age   | membership | amount    | Result | Message                                  |         |
|-------|------|---------------|---------------------------------|-------------------------------------------------------------------------------|-------|------------|-----------|--------|------------------------------------------|---------|
| UT-01 | R5   | C1:BVA BelowMin| Age below minimum (17)         | Apply age=17, Gold, 500,000 — age invalid, reject                             | 17    | Gold       | 500,000   | Reject | "Age must be between 18 and 65"          | Invalid |
| UT-02 | R1   | C1:BVA Min    | Age at minimum (18)             | Apply age=18, Gold, 500,000 — age at boundary, accept                         | 18    | Gold       | 500,000   | Accept | Loan application accepted                | Valid   |
| UT-03 | R1   | C1:BVA AboveMin| Age above minimum (19)         | Apply age=19, Gold, 500,000 — age inside range, accept                        | 19    | Gold       | 500,000   | Accept | Loan application accepted                | Valid   |
| UT-04 | R1   | C1:BVA BelowMax| Age below maximum (64)         | Apply age=64, Gold, 500,000 — age inside range, accept                        | 64    | Gold       | 500,000   | Accept | Loan application accepted                | Valid   |
| UT-05 | R1   | C1:BVA Max    | Age at maximum (65)             | Apply age=65, Gold, 500,000 — age at boundary, accept                         | 65    | Gold       | 500,000   | Accept | Loan application accepted                | Valid   |
| UT-06 | R5   | C1:BVA AboveMax| Age above maximum (66)         | Apply age=66, Gold, 500,000 — age invalid, reject                             | 66    | Gold       | 500,000   | Reject | "Age must be between 18 and 65"          | Invalid |
| UT-07 | R1   | C2:EP Premium | Membership Gold                 | Apply age=30, Gold, 500,000 — valid membership, accept                        | 30    | Gold       | 500,000   | Accept | Loan application accepted                | Valid   |
| UT-08 | R1   | C2:EP Premium | Membership Silver               | Apply age=30, Silver, 500,000 — valid membership, accept                      | 30    | Silver     | 500,000   | Accept | Loan application accepted                | Valid   |
| UT-09 | R3   | C2:EP Basic   | Membership Bronze               | Apply age=30, Bronze, 500,000 — invalid membership, reject                    | 30    | Bronze     | 500,000   | Reject | "Membership not eligible for loan"       | Invalid |
| UT-10 | R3   | C2:EP Unknown | Membership blank                | Apply age=30, blank, 500,000 — missing membership, reject                     | 30    | (blank)    | 500,000   | Reject | "Membership required"                    | Invalid |
| UT-11 | R2   | C3:BVA BelowMin| Amount below minimum (9,900)   | Apply age=30, Gold, 9,900 — amount invalid, reject                            | 30    | Gold       | 9,900     | Reject | "Amount below minimum 10,000"            | Invalid |
| UT-12 | R1   | C3:BVA Min    | Amount at minimum (10,000)      | Apply age=30, Gold, 10,000 — amount at boundary, accept                       | 30    | Gold       | 10,000    | Accept | Loan application accepted                | Valid   |
| UT-13 | R1   | C3:BVA AboveMin| Amount above minimum (10,100)  | Apply age=30, Gold, 10,100 — amount inside range, accept                      | 30    | Gold       | 10,100    | Accept | Loan application accepted                | Valid   |
| UT-14 | R1   | C3:BVA BelowMax| Amount below maximum (999,900) | Apply age=30, Gold, 999,900 — amount inside range, accept                     | 30    | Gold       | 999,900   | Accept | Loan application accepted                | Valid   |
| UT-15 | R1   | C3:BVA Max    | Amount at maximum (1,000,000)   | Apply age=30, Gold, 1,000,000 — amount at boundary, accept                    | 30    | Gold       | 1,000,000 | Accept | Loan application accepted                | Valid   |
| UT-16 | R2   | C3:BVA AboveMax| Amount above maximum (1,000,100)| Apply age=30, Gold, 1,000,100 — amount invalid, reject                       | 30    | Gold       | 1,000,100 | Reject | "Amount exceeds maximum 1,000,000"       | Invalid |
```

Key rules:
- **Every** BVA boundary value (6 per numeric field) and **every** EP partition
  representative appears as a test case
- **Technique** column shows exactly which technique and position produced the value
  (e.g., `C1:BVA Min`, `C2:EP Premium`)
- **Rule** column traces back to the Decision Table
- When testing one condition, other conditions use a **typical valid value**
  (e.g., age=30, membership=Gold, amount=500,000)

---

### If Strategy 2 (Sequential Condition) was chosen:

#### Table 1: Unit Test Cases (Sequential Condition Technique)

Uses **all** boundary/partition values. Each test stops at the first failing condition.
The **Reached** column shows how far the validation got. Use **—** for conditions
that were never checked.

Label: **"Unit Test Cases (Sequential Condition: BVA + EP)"**

```markdown
#### Unit Test Cases (Sequential Condition: BVA + EP)

| TC-ID | Reached | Technique      | Test Case Name                  | Test Description                                                                  | Input | Input      | Input         | Output | Output                                   | Type    |
|       |         |                |                                 |                                                                                   | age   | membership | amount        | Result | Message                                  |         |
|-------|---------|----------------|---------------------------------|-----------------------------------------------------------------------------------|-------|------------|---------------|--------|------------------------------------------|---------|
| UT-01 | C1      | C1:BVA BelowMin| Age below minimum (17)          | Apply age=17 — rejects at age, does not check further                             | 17    | —          | —             | Reject | "Age must be between 18 and 65"          | Invalid |
| UT-02 | C1      | C1:BVA AboveMax| Age above maximum (66)          | Apply age=66 — rejects at age, does not check further                             | 66    | —          | —             | Reject | "Age must be between 18 and 65"          | Invalid |
| UT-03 | C2      | C2:EP Basic    | Age pass, membership Bronze     | Apply age=30, Bronze — rejects at membership, does not check amount               | 30    | Bronze     | —             | Reject | "Membership not eligible for loan"       | Invalid |
| UT-04 | C2      | C2:EP Unknown  | Age pass, membership blank      | Apply age=30, blank — rejects at membership, does not check amount                | 30    | (blank)    | —             | Reject | "Membership required"                    | Invalid |
| UT-05 | C3      | C3:BVA BelowMin| All pass → amount below min     | Apply age=30, Gold, 9,900 — rejects at amount                                    | 30    | Gold       | 9,900         | Reject | "Amount below minimum 10,000"            | Invalid |
| UT-06 | C3      | C3:BVA AboveMax| All pass → amount above max     | Apply age=30, Gold, 1,000,100 — rejects at amount                                | 30    | Gold       | 1,000,100     | Reject | "Amount exceeds maximum 1,000,000"       | Invalid |
| UT-07 | END     | C1:BVA Min     | All pass (min age boundary)     | Apply age=18, Gold, 10,000 — all pass, accepted                                  | 18    | Gold       | 10,000        | Accept | Loan application accepted                | Valid   |
| UT-08 | END     | C1:BVA Max     | All pass (max age boundary)     | Apply age=65, Silver, 1,000,000 — all pass, accepted                              | 65    | Silver     | 1,000,000     | Accept | Loan application accepted                | Valid   |
| UT-09 | END     | C1:BVA AboveMin| All pass (above min age)        | Apply age=19, Gold, 500,000 — all pass, accepted                                 | 19    | Gold       | 500,000       | Accept | Loan application accepted                | Valid   |
| UT-10 | END     | C2:EP Premium  | All pass (Silver membership)    | Apply age=30, Silver, 500,000 — all pass, accepted                                | 30    | Silver     | 500,000       | Accept | Loan application accepted                | Valid   |
| UT-11 | END     | C3:BVA Min     | All pass (min amount boundary)  | Apply age=30, Gold, 10,000 — all pass, accepted                                  | 30    | Gold       | 10,000        | Accept | Loan application accepted                | Valid   |
| UT-12 | END     | C3:BVA Max     | All pass (max amount boundary)  | Apply age=30, Gold, 1,000,000 — all pass, accepted                                | 30    | Gold       | 1,000,000     | Accept | Loan application accepted                | Valid   |
```

Key rules for Sequential:
- **Invalid values** — only test the **most critical** invalid values per condition
  (Below Min and Above Max for BVA, one per invalid partition for EP).
  No need to test all 6 BVA points as invalid since the system stops at the
  first failure anyway.
- **Valid values** — test ALL valid BVA boundary points (Min, Above Min, Below Max,
  Max) and ALL valid EP partitions in the "all pass" rows. These verify the condition
  passes correctly and the system proceeds to the next check.
- **—** means the system never checked this field

---

### Table 2: Acceptance Test Cases (Business Scenarios) — same for both strategies

Uses real-world data from Step 4. Tests realistic end-to-end scenarios.

Label: **"Acceptance Test Cases (Business Scenarios)"**

```markdown
#### Acceptance Test Cases (Business Scenarios)

| TC-ID | Test Case Name                  | Test Description                                                                        | Input | Input      | Input     | Output | Output                                   | Type    |
|       |                                 |                                                                                         | age   | membership | amount    | Result | Message                                  |         |
|-------|--------|------------------------|-----------------------------------------------------------------------------------------|-------|------------|-----------|--------|------------------------------------------|---------|
| AT-01 | Young professional loan         | A 28-year-old Gold member applies for 200,000 — system should accept                    | 28    | Gold       | 200,000   | Accept | Loan application accepted                | Valid   |
| AT-02 | Senior member max loan          | A 60-year-old Silver member applies for 800,000 — system should accept                  | 60    | Silver     | 800,000   | Accept | Loan application accepted                | Valid   |
| AT-03 | Teenager attempts to apply      | A 16-year-old tries to apply (common rejection) — system should reject                  | 16    | Gold       | 200,000   | Reject | "Age must be between 18 and 65"          | Invalid |
| AT-04 | Basic member tries to apply     | A 35-year-old Bronze member applies — system should reject membership                   | 35    | Bronze     | 200,000   | Reject | "Membership not eligible for loan"       | Invalid |
| AT-05 | Amount too large                | A 40-year-old Gold member applies for 2,000,000 — system should reject amount           | 40    | Gold       | 2,000,000 | Reject | "Amount exceeds maximum 1,000,000"       | Invalid |
```

Number as AT-01, AT-02, etc.

Same `⚠️ TBD` rules apply when real-world data was not provided.
