# Test Case Table Template

Use this tabular format for the final test case output in Step 6.
Generate **two separate tables**: one for unit testing, one for acceptance testing.

## Common format

Both tables use **two-level column headers**:
- **Level 1** (top row): repeats `Input` across all input field columns and `Output`
  across all output field columns — this indicates grouping. When copied to a spreadsheet,
  merge the repeated cells into one.
- **Level 2** (second row): individual field names under each group.

Determine the input and output field names dynamically from the requirement.

## Column guide

- **TC-ID**: Sequential identifier (UT-01, UT-02... for unit tests; AT-01, AT-02... for acceptance tests)
- **Test Case Name**: Short name identifying the boundary condition (e.g., "Age at lower boundary")
- **Test Description**: A human-readable sentence describing what input is given and
  what the system should do. Write it so a non-technical person can understand the test
  without looking at the data columns.
- **Input (repeated)**: One column per input field. Level 1 shows `Input` on every
  input column. Level 2 shows the field name.
- **Output (repeated)**: One column per output field. Level 1 shows `Output` on every
  output column. Level 2 shows the field name.
- **Type**: `Valid` or `Invalid`

---

## Table 1: Unit Test Cases (BVA Technique)

Uses **exact boundary values** from the BVA analysis. Purpose: guide developers to
write unit tests that verify boundary logic in code.

```markdown
#### Unit Test Cases (BVA Technique)

| TC-ID | Test Case Name                  | Test Description                                                        | Input     | Input | Input          | Output | Output                                    | Type    |
|       |                                 |                                                                         | name      | age   | email          | Result | Message                                   |         |
|-------|---------------------------------|-------------------------------------------------------------------------|-----------|-------|----------------|--------|-----------------------------------------  |---------|
| UT-01 | Age at minimum boundary (18)    | Register with age 18 and system should accept and show welcome page     | Sarah Lee | 18    | sarah@test.com | Accept | Registration accepted, welcome page shown | Valid   |
| UT-02 | Age below minimum boundary (17) | Register with age 17 and system should reject with minimum age error    | Tom Park  | 17    | tom@test.com   | Reject | "You must be at least 18 to register"     | Invalid |
| UT-03 | Age just above minimum (19)     | Register with age 19 and system should accept and show welcome page     | Amy Chen  | 19    | amy@test.com   | Accept | Registration accepted, welcome page shown | Valid   |
| UT-04 | Age just below maximum (64)     | Register with age 64 and system should accept and show welcome page     | Bob Grant | 64    | bob@test.com   | Accept | Registration accepted, welcome page shown | Valid   |
| UT-05 | Age at maximum boundary (65)    | Register with age 65 and system should accept and show welcome page     | Lisa Wang | 65    | lisa@test.com  | Accept | Registration accepted, welcome page shown | Valid   |
| UT-06 | Age above maximum boundary (66) | Register with age 66 and system should reject with maximum age error    | Mark Hall | 66    | mark@test.com  | Reject | "Age must be between 18 and 65"           | Invalid |
```

---

## Table 2: Acceptance Test Cases (Business Scenarios)

Uses **real-world representative data** collected from the user in Step 5. Purpose:
validate end-to-end business process with data that reflects actual customer behavior.

The input values are NOT boundary values — they are realistic amounts, ages, or
quantities that real customers would actually use, as provided by the user.

```markdown
#### Acceptance Test Cases (Business Scenarios)

| TC-ID | Test Case Name               | Test Description                                                              | Input     | Input | Input          | Output | Output                                    | Type    |
|       |                              |                                                                               | name      | age   | email          | Result | Message                                   |         |
|-------|-----------------------------|-------------------------------------------------------------------------------|-----------|-------|----------------|--------|-----------------------------------------  |---------|
| AT-01 | Typical young adult register | A 25-year-old registers and system should accept and show welcome page        | Sarah Lee | 25    | sarah@test.com | Accept | Registration accepted, welcome page shown | Valid   |
| AT-02 | Senior customer register     | A 60-year-old registers and system should accept and show welcome page        | Bob Grant | 60    | bob@test.com   | Accept | Registration accepted, welcome page shown | Valid   |
| AT-03 | Minor attempts to register   | A 15-year-old attempts to register and system should reject with age error    | Tom Park  | 15    | tom@test.com   | Reject | "You must be at least 18 to register"     | Invalid |
```
