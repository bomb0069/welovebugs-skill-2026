# Workflow B: Equivalence Partitioning (EP)

**Use when**: requirement has **non-numeric input fields** such as text, categories,
status, file types, user roles, dropdown selections, etc.

This workflow handles two complexity levels:
- **Simple EP** — single field with independent partitions (e.g., file type)
- **Combined EP** — multiple fields whose partitions combine to produce different
  outcomes (e.g., department × employee level = bonus rate)

Work through each step in order. **Only pause and wait for user input when a step
is marked with ⏸.** For all other steps, continue automatically.

## Step 1: Understand the requirement and identify input fields

Analyze the requirement and extract:

1. **What the feature does** — one-sentence summary
2. **Input fields** — list every field with its data type and accepted values
3. **Rules for each field** — what values are valid, invalid, and any special conditions

Present this as a table:

```markdown
| Field        | Type       | Accepted Values                        | Special rules                    |
|-------------|------------|----------------------------------------|----------------------------------|
| file_type   | string     | .pdf, .doc, .docx, .xlsx               | Other extensions rejected        |
| user_role   | enum       | admin, editor, viewer                  | Only admin can delete            |
| email       | string     | Valid email format                     | Must contain @ and domain        |
| country     | dropdown   | TH, US, JP, SG, ...                   | Determines tax calculation       |
```

If the requirement is unclear about what values are accepted, ask the user before proceeding.

## Step 2: Identify equivalence partitions ⏸ Wait for user input

For each input field, divide all possible values into **partitions** — groups where
the system is expected to behave the same way.

Each partition is either:
- **Valid partition** — values the system should accept
- **Invalid partition** — values the system should reject

**Ask the user to confirm the partitions are correct before proceeding.**

Present the partitions as a table:

```markdown
| Field      | Partition Name            | Example Values               | Valid/Invalid |
|-----------|---------------------------|------------------------------|---------------|
| file_type | Supported document        | .pdf, .doc, .docx            | Valid         |
| file_type | Supported spreadsheet     | .xlsx, .csv                  | Valid         |
| file_type | Unsupported image         | .jpg, .png, .gif             | Invalid       |
| file_type | Unsupported executable    | .exe, .bat, .sh              | Invalid       |
| file_type | Empty / no extension      | (blank), "file"              | Invalid       |
| user_role | Privileged role           | admin                        | Valid         |
| user_role | Standard role             | editor, viewer               | Valid         |
| user_role | Unknown role              | superuser, guest, (blank)    | Invalid       |
```

Then generate a **visual partition diagram** for each field showing the partitions
as sets/territories. Draw the valid partitions inside a box and invalid partitions
outside:

```
        Field: file_type

        ┌─────────────────────────────────────────────────────┐
        │                  ✓ Valid Partitions                  │
        │                                                     │
        │  ┌──────────────────┐  ┌──────────────────────┐     │
        │  │   Supported      │  │   Supported           │    │
        │  │   Document       │  │   Spreadsheet         │    │
        │  │                  │  │                        │    │
        │  │  .pdf .doc .docx │  │  .xlsx .csv            │    │
        │  └──────────────────┘  └──────────────────────┘     │
        │                                                     │
        └─────────────────────────────────────────────────────┘

        ┌─────────────────────────────────────────────────────┐
        │                  ✗ Invalid Partitions                │
        │                                                     │
        │  ┌──────────────┐ ┌──────────────┐ ┌─────────────┐  │
        │  │ Unsupported  │ │ Unsupported  │ │  Empty /     │  │
        │  │ Image        │ │ Executable   │ │  No ext      │  │
        │  │              │ │              │ │              │  │
        │  │ .jpg .png    │ │ .exe .bat    │ │ (blank)      │  │
        │  │ .gif         │ │ .sh          │ │ "file"       │  │
        │  └──────────────┘ └──────────────┘ └─────────────┘  │
        │                                                     │
        └─────────────────────────────────────────────────────┘
```

Another example with roles:

```
        Field: user_role

        ┌──────────────────────────────────────────────┐
        │               ✓ Valid Partitions              │
        │                                              │
        │  ┌─────────────────┐  ┌──────────────────┐   │
        │  │  Privileged     │  │  Standard        │   │
        │  │                 │  │                  │   │
        │  │  admin          │  │  editor, viewer  │   │
        │  └─────────────────┘  └──────────────────┘   │
        │                                              │
        └──────────────────────────────────────────────┘

        ┌──────────────────────────────────────────────┐
        │               ✗ Invalid Partitions            │
        │                                              │
        │  ┌───────────────────────────────────────┐   │
        │  │  Unknown role                         │   │
        │  │                                       │   │
        │  │  superuser, guest, (blank)            │   │
        │  └───────────────────────────────────────┘   │
        │                                              │
        └──────────────────────────────────────────────┘
```

Generate one diagram per field. Use the same language as the requirement.

### Step 2b: Check for combined partitions ⏸ Wait for user input

After identifying partitions for each field, check whether the fields are
**independent** or **combined**:

- **Independent** — each field's outcome doesn't depend on other fields
  (e.g., file type validation doesn't depend on user role)
  → Skip to Step 3 (simple EP)

- **Combined** — two or more fields combine to produce different outcomes
  (e.g., department + employee level = different bonus rates)
  → Continue with Step 2c (combined EP)

Ask the user:

```
Do these fields affect each other?

  1 → Independent — each field is validated separately
      (e.g., file type doesn't depend on user role)

  2 → Combined — fields combine to produce different outcomes
      (e.g., department + employee level = different bonus rate)

Please reply with 1 or 2:
```

If the user chooses **1 (Independent)**, skip to Step 3.

If the user chooses **2 (Combined)**, continue with Step 2c.

### Step 2c: Identify combined partition groups

When fields combine, their partitions form **combined groups** where the
outcome depends on the combination. Present the individual partitions first,
then show how they combine.

**Example requirement:**
> "Yearly bonus calculation: each department has a different base rate.
> Employee level adds a plus rate: Manager +1%, Leader +0.5%, Normal +0%."

Individual partitions:

```markdown
| Field      | Partition Name | Values                    | Rate   |
|------------|---------------|---------------------------|--------|
| department | Engineering   | Engineering               | 10%    |
| department | Sales         | Sales                     | 8%     |
| department | HR            | HR                        | 7%     |
| department | Unknown dept  | (blank), other            | Invalid|
| level      | Manager       | Manager                   | +1%    |
| level      | Leader        | Leader                    | +0.5%  |
| level      | Normal        | Staff, Junior, Senior     | +0%    |
| level      | Unknown level | (blank), other            | Invalid|
```

Then draw a **combined partition diagram** using an **overlapping sets** style.
This is like a set theory / Venn diagram where:
- The **outer rectangle** = Universal set (all employees, base rate)
- **Department partitions** = vertical area sets (each with its own rate)
- **Level partitions** = horizontal band sets that cross through departments
- **Intersections** where department and level overlap = combined partition with calculated rate
- **Special partitions** = sets that sit outside the main grid (e.g., Blacklist, Secretary)

```
  Combined Partitions: Department × Level → Bonus Rate

  ┌──────────────────────────────────────────────────────────────────────────────────┐
  │  Universal: All Employees — Base Rate 5%                                        │
  │                                                                                 │
  │        ┌─ Engineering ─┐    ┌─ Finance ────┐    ┌─ Tester ────┐                 │
  │        │      7%       │    │    6.5%      │    │    10%      │   ┌───────────┐  │
  │        │               │    │              │    │             │   │ Secretary │  │
  │        │               │    │              │    │             │   │    0%     │  │
  │  ┌─────┼───────────────┼────┼──────────────┼────┼─────────────┼─┐ └───────────┘  │
  │  │Mgr  │ Eng+Mgr      │    │ Fin+Mgr      │    │ Test+Mgr    │ │               │
  │  │ +1% │ 7+1 = 8%     │    │ 6.5+1 = 7.5% │    │ 10+1 = 11%  │ │               │
  │  └─────┼───────────────┼────┼──────────────┼────┼─────────────┼─┘               │
  │        │               │    │              │    │             │                  │
  │  ┌─────┼───────────────┼────┼──────────────┼────┼─────────────┼─┐               │
  │  │Lead │ Eng+Lead      │    │ Fin+Lead     │    │ Test+Lead   │ │               │
  │  │+0.5%│ 7+0.5 = 7.5% │    │6.5+0.5 = 7%  │    │10+0.5=10.5% │ │               │
  │  └─────┼───────────────┼────┼──────────────┼────┼─────────────┼─┘               │
  │        │               │    │              │    │             │   ┌───────────┐  │
  │        │ Eng+Normal    │    │ Fin+Normal   │    │ Test+Normal │   │ Blacklist │  │
  │        │ 7+0 = 7%     │    │ 6.5+0 = 6.5% │    │ 10+0 = 10%  │   │   -1%     │  │
  │        │               │    │              │    │             │   └───────────┘  │
  │        └───────────────┘    └──────────────┘    └─────────────┘                  │
  │                                                                                 │
  └──────────────────────────────────────────────────────────────────────────────────┘
```

**How to read the diagram:**
- **Outer box** = Universal set — every employee starts with the base rate
- **Vertical columns** = Department sets — each adds its department-specific rate
- **Horizontal bands** (Manager +1%, Leader +0.5%) = Level sets that cross through
  all departments — each level adds a plus rate on top of the department rate
- **Intersection cells** = where a department column meets a level band,
  showing the combined rate (e.g., Engineering ∩ Manager = 7% + 1% = 8%)
- **Special sets** outside the grid = partitions with unique rules
  (Secretary gets 0% regardless of level, Blacklist gets -1% penalty)

This diagram makes each **intersection** a distinct test partition. Every cell
where two sets overlap is one test case.

Generate one diagram per requirement. Use the same language as the requirement
(e.g., if the requirement is in Thai, use Thai labels).

### Step 2d: Build combined partition summary table

List every valid combination with its expected outcome. This becomes the
test case source:

```markdown
#### Combined Partition Summary

| # | Department  | Level   | Base Rate | Plus Rate | Total Bonus | Expected Output              |
|---|-------------|---------|-----------|-----------|-------------|-------------------------------|
| 1 | Engineering | Manager | 10%       | +1%       | 11%         | Bonus calculated at 11%       |
| 2 | Engineering | Leader  | 10%       | +0.5%     | 10.5%       | Bonus calculated at 10.5%     |
| 3 | Engineering | Normal  | 10%       | +0%       | 10%         | Bonus calculated at 10%       |
| 4 | Sales       | Manager | 8%        | +1%       | 9%          | Bonus calculated at 9%        |
| 5 | Sales       | Leader  | 8%        | +0.5%     | 8.5%        | Bonus calculated at 8.5%      |
| 6 | Sales       | Normal  | 8%        | +0%       | 8%          | Bonus calculated at 8%        |
| 7 | HR          | Manager | 7%        | +1%       | 8%          | Bonus calculated at 8%        |
| 8 | HR          | Leader  | 7%        | +0.5%     | 7.5%        | Bonus calculated at 7.5%      |
| 9 | HR          | Normal  | 7%        | +0%       | 7%          | Bonus calculated at 7%        |

**Invalid combinations (any field invalid → reject):**

| #  | Department | Level   | Expected Output                         |
|----|------------|---------|------------------------------------------|
| 10 | (blank)    | Manager | Reject: "Department is required"         |
| 11 | Other      | Normal  | Reject: "Unknown department"             |
| 12 | Engineering| (blank) | Reject: "Employee level is required"     |
| 13 | Engineering| Other   | Reject: "Unknown employee level"         |
| 14 | (blank)    | (blank) | Reject: "Department is required"         |
```

**Important**: Note that some combinations may produce the **same outcome**
(e.g., HR+Manager = 8% and Sales+Normal = 8%). These are still separate test
cases because they test different partition paths even though the result is the same.

After building this summary, continue to Step 3 using the combined partitions
as the test case source instead of individual field partitions.

## Step 3: Select representative values

### For simple EP (independent fields):

From each partition, select **one representative value** to use in test cases.
The idea: if the system handles one value from the partition correctly, it should
handle all values in that partition the same way.

```markdown
| # | Field     | Partition Name         | Representative Value | Valid/Invalid |
|---|-----------|------------------------|---------------------|---------------|
| 1 | file_type | Supported document     | .pdf                | Valid         |
| 2 | file_type | Supported spreadsheet  | .xlsx               | Valid         |
| 3 | file_type | Unsupported image      | .jpg                | Invalid       |
| 4 | file_type | Unsupported executable | .exe                | Invalid       |
| 5 | file_type | Empty / no extension   | (blank)             | Invalid       |
```

### For combined EP (dependent fields):

Each row in the Combined Partition Summary (Step 2d) is already a test case.
Select representative values for each combination:

```markdown
| # | Department  | Level   | Total Bonus | Valid/Invalid |
|---|-------------|---------|-------------|---------------|
| 1 | Engineering | Manager | 11%         | Valid         |
| 2 | Engineering | Leader  | 10.5%       | Valid         |
| 3 | Engineering | Normal  | 10%         | Valid         |
| 4 | Sales       | Manager | 9%          | Valid         |
| 5 | Sales       | Leader  | 8.5%        | Valid         |
| 6 | Sales       | Normal  | 8%          | Valid         |
| 7 | HR          | Manager | 8%          | Valid         |
| 8 | HR          | Leader  | 7.5%        | Valid         |
| 9 | HR          | Normal  | 7%          | Valid         |
| 10| (blank)     | Manager | —           | Invalid       |
| 11| Engineering | (blank) | —           | Invalid       |
```

## Step 4: Generate expected outputs

For each representative value (or combination), determine the expected system
behavior based on the requirement's business rules. Be specific.

### Simple EP:

```markdown
| # | Field     | Value  | Partition              | Valid/Invalid | Expected Output                            |
|---|-----------|--------|------------------------|---------------|--------------------------------------------|
| 1 | file_type | .pdf   | Supported document     | Valid         | File uploaded, preview shown               |
| 2 | file_type | .xlsx  | Supported spreadsheet  | Valid         | File uploaded, preview shown               |
| 3 | file_type | .jpg   | Unsupported image      | Invalid       | Error: "File type not supported"           |
| 4 | file_type | .exe   | Unsupported executable | Invalid       | Error: "File type not supported"           |
| 5 | file_type | (blank)| Empty / no extension   | Invalid       | Error: "Please select a file to upload"    |
```

### Combined EP:

For combined partitions, the expected output depends on the **combination**.
Use the Combined Partition Summary from Step 2d:

```markdown
| # | Department  | Level   | Valid/Invalid | Expected Output                                        |
|---|-------------|---------|---------------|--------------------------------------------------------|
| 1 | Engineering | Manager | Valid         | Bonus = salary × 11% (base 10% + plus 1%)             |
| 2 | Engineering | Leader  | Valid         | Bonus = salary × 10.5% (base 10% + plus 0.5%)         |
| 3 | Engineering | Normal  | Valid         | Bonus = salary × 10% (base 10% + plus 0%)             |
| 4 | Sales       | Manager | Valid         | Bonus = salary × 9% (base 8% + plus 1%)               |
| 5 | Sales       | Leader  | Valid         | Bonus = salary × 8.5% (base 8% + plus 0.5%)           |
| 6 | Sales       | Normal  | Valid         | Bonus = salary × 8% (base 8% + plus 0%)               |
| 7 | HR          | Manager | Valid         | Bonus = salary × 8% (base 7% + plus 1%)               |
| 8 | HR          | Leader  | Valid         | Bonus = salary × 7.5% (base 7% + plus 0.5%)           |
| 9 | HR          | Normal  | Valid         | Bonus = salary × 7% (base 7% + plus 0%)               |
| 10| (blank)     | Manager | Invalid       | Error: "Department is required"                        |
| 11| Engineering | (blank) | Invalid       | Error: "Employee level is required"                    |
```

## Step 5: Collect real-world representative data ⏸ Wait for user input (REQUIRED)

The values from Step 3 are ideal for **unit testing**. For **acceptance testing**,
we need data that reflects how real users actually use the system.

**This step is critical. Do NOT skip it. Do NOT proceed to Step 6 until the user
provides real-world data or explicitly chooses to skip.**

Ask the user for each partition: **"What value would a real customer typically use
in this situation?"**

Present the request as a table the user must fill in:

```markdown
Please provide real-world values for each partition:

| Partition              | Representative Value | Real-world Value | Source / Reason |
|------------------------|---------------------|------------------|-----------------|
| Supported document     | .pdf                | ❓ Please provide | ❓               |
| Supported spreadsheet  | .xlsx               | ❓ Please provide | ❓               |
| Unsupported image      | .jpg                | ❓ Please provide | ❓               |
| Unsupported executable | .exe                | ❓ Please provide | ❓               |
| Empty / no extension   | (blank)             | ❓ Please provide | ❓               |
```

Help the user think about it with guiding questions:
- "What file type do your customers upload most often?"
- "When users upload the wrong file type, what do they usually try?"
- "Do you have any usage logs or support tickets showing common mistakes?"

**If the user says they don't know or wants to skip**, ask one more time:
> "Real-world data is important because acceptance tests should reflect how
> customers actually use the system. Can you check with your team, product owner,
> or look at any usage reports?"

**If the user still chooses to skip**, accept it but mark the missing data clearly
with `⚠️ TBD (awaiting real-world data)` in the acceptance test table.

## Step 6: Generate final test case tables

Produce **two separate tables** using the template from `test-case-template.md`.

Both tables use **two-level column headers**:
- **Level 1** (top): repeats `Input` across input columns and `Output` across output columns
- **Level 2**: individual field names under each group

Determine input and output field names dynamically from the requirement.

---

### Table 1: Technique-based test cases (for Unit Testing)

These use the **representative values** from EP analysis. Purpose: guide developers
to write unit tests that verify partition logic in their code.

Label this table clearly: **"Unit Test Cases (EP Technique)"**

**Simple EP example:**

```markdown
#### Unit Test Cases (EP Technique)

| TC-ID | Test Case Name            | Test Description                                                       | Input    | Input     | Output | Output                             | Type    |
|       |                           |                                                                        | filename | file_type | Result | Message                            |         |
|-------|---------------------------|------------------------------------------------------------------------|----------|-----------|--------|------------------------------------|---------|
| UT-01 | Supported document        | Upload a .pdf file and system should accept and show preview           | test     | .pdf      | Accept | File uploaded, preview shown       | Valid   |
| UT-02 | Supported spreadsheet     | Upload a .xlsx file and system should accept and show preview          | data     | .xlsx     | Accept | File uploaded, preview shown       | Valid   |
| UT-03 | Unsupported image         | Upload a .jpg file and system should reject with unsupported error     | photo    | .jpg      | Reject | "File type not supported"          | Invalid |
| UT-04 | Unsupported executable    | Upload a .exe file and system should reject with unsupported error     | setup    | .exe      | Reject | "File type not supported"          | Invalid |
| UT-05 | Empty / no extension      | Upload with no file selected and system should reject with empty error | (blank)  | (blank)   | Reject | "Please select a file to upload"   | Invalid |
```

**Combined EP example (department × level → bonus):**

```markdown
#### Unit Test Cases (Combined EP Technique)

| TC-ID | Test Case Name               | Test Description                                                                       | Input       | Input   | Input      | Output | Output                               | Type    |
|       |                              |                                                                                        | employee    | dept    | level      | Rate   | Bonus Calculation                    |         |
|-------|------------------------------|----------------------------------------------------------------------------------------|-------------|---------|------------|--------|--------------------------------------|---------|
| UT-01 | Eng + Manager                | Engineering manager should receive 11% bonus (base 10% + plus 1%)                     | EMP-001     | Engineering | Manager | 11%    | Bonus = salary × 11%               | Valid   |
| UT-02 | Eng + Leader                 | Engineering leader should receive 10.5% bonus (base 10% + plus 0.5%)                  | EMP-002     | Engineering | Leader  | 10.5%  | Bonus = salary × 10.5%             | Valid   |
| UT-03 | Eng + Normal                 | Engineering normal employee should receive 10% bonus (base 10% + plus 0%)             | EMP-003     | Engineering | Normal  | 10%    | Bonus = salary × 10%              | Valid   |
| UT-04 | Sales + Manager              | Sales manager should receive 9% bonus (base 8% + plus 1%)                             | EMP-004     | Sales       | Manager | 9%     | Bonus = salary × 9%               | Valid   |
| UT-05 | Sales + Leader               | Sales leader should receive 8.5% bonus (base 8% + plus 0.5%)                          | EMP-005     | Sales       | Leader  | 8.5%   | Bonus = salary × 8.5%             | Valid   |
| UT-06 | Sales + Normal               | Sales normal employee should receive 8% bonus (base 8% + plus 0%)                     | EMP-006     | Sales       | Normal  | 8%     | Bonus = salary × 8%               | Valid   |
| UT-07 | HR + Manager                 | HR manager should receive 8% bonus (base 7% + plus 1%)                                | EMP-007     | HR          | Manager | 8%     | Bonus = salary × 8%               | Valid   |
| UT-08 | HR + Leader                  | HR leader should receive 7.5% bonus (base 7% + plus 0.5%)                             | EMP-008     | HR          | Leader  | 7.5%   | Bonus = salary × 7.5%             | Valid   |
| UT-09 | HR + Normal                  | HR normal employee should receive 7% bonus (base 7% + plus 0%)                        | EMP-009     | HR          | Normal  | 7%     | Bonus = salary × 7%               | Valid   |
| UT-10 | Unknown department           | Employee with blank department should be rejected                                       | EMP-010     | (blank)     | Manager | —      | "Department is required"           | Invalid |
| UT-11 | Unknown level                | Engineering employee with blank level should be rejected                                | EMP-011     | Engineering | (blank) | —      | "Employee level is required"       | Invalid |
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

| TC-ID | Test Case Name               | Test Description                                                                  | Input             | Input     | Output | Output                            | Type    |
|       |                              |                                                                                   | filename          | file_type | Result | Message                           |         |
|-------|------------------------------|-----------------------------------------------------------------------------------|-------------------|-----------|--------|------------------------------------|---------|
| AT-01 | Customer uploads contract    | Customer uploads contract.pdf (most common) and system should accept              | contract_2024     | .pdf      | Accept | File uploaded, preview shown       | Valid   |
| AT-02 | Customer uploads wrong type  | Customer uploads photo.heic (common mistake from iPhone) and system should reject  | IMG_2024          | .heic     | Reject | "File type not supported"          | Invalid |
```

**When the user skipped providing real-world data**, use `⚠️ TBD`:

```markdown
#### Acceptance Test Cases (Business Scenarios)

> ⚠️ **Some test data is pending.** Values marked "TBD" need real-world data
> from the team before these test cases can be executed.

| TC-ID | Test Case Name            | Test Description                                                    | Input                            | Input                            | Output | Output                            | Type    |
|       |                           |                                                                     | filename                         | file_type                        | Result | Message                           |         |
|-------|---------------------------|---------------------------------------------------------------------|----------------------------------|----------------------------------|--------|------------------------------------|---------|
| AT-01 | Typical upload            | Customer uploads ⚠️ TBD and system should accept                   | ⚠️ TBD (awaiting real-world data)| ⚠️ TBD (awaiting real-world data)| Accept | File uploaded, preview shown       | Valid   |
| AT-02 | Common wrong type         | Customer uploads ⚠️ TBD and system should reject                   | ⚠️ TBD (awaiting real-world data)| ⚠️ TBD (awaiting real-world data)| Reject | "File type not supported"          | Invalid |
```

Number these as AT-01, AT-02, etc.
