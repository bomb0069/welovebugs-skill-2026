# Eval: Common Rules

Tests for cross-workflow behaviors: language support, pause behavior, file saving,
output format, and general guidelines.

---

## Language Support

### EVAL-CR01: Thai language requirement

**Requirement:**
> ระบบรับสมัครสมาชิก ผู้สมัครต้องมีอายุระหว่าง 18-60 ปี

**User Responses:**
1. Workflow selection → "A"
2. One unit → "1" (1 ปี)
3. Real-world data → "ผู้สมัครส่วนใหญ่อายุ 25-40 ปี มีบางคนอายุ 15 ปีมาลองสมัคร"

**Evaluation Criteria:**
- [ ] All output text is in Thai (matching the requirement language)
- [ ] Condition table labels in Thai
- [ ] Boundary diagram labels in Thai (e.g., ✓ ได้ / ✗ ไม่ได้)
- [ ] Test case names and descriptions in Thai
- [ ] Column headers may remain in English (TC-ID, Type) but field names follow Thai terms
- [ ] Does NOT switch to English unless user asks

---

### EVAL-CR02: English requirement — stays in English

**Requirement:**
> Minimum password length is 8 characters, maximum is 128 characters.

**User Responses:**
1. Workflow selection → "A"
2. One unit → "1" (1 character)

**Evaluation Criteria:**
- [ ] All output in English
- [ ] No unexpected language switches

---

## Pause Behavior

### EVAL-CR03: Only pauses at ⏸ steps

**Requirement:**
> The system accepts deposits between 500 and 100,000 THB.

**User Responses:**
1. Workflow selection → "A"
2. One unit → "1" (0.01 THB)

**Evaluation Criteria:**
- [ ] After workflow selection, runs Steps 1–4 continuously without pausing
- [ ] Presents boundary extraction, boundary values, diagram, and test possibilities together
- [ ] First pause is at "one unit" confirmation (Step 2 in BVA — marked with ⏸)
- [ ] After unit confirmation, continues through Steps 3–4 without pausing
- [ ] Next pause is at real-world data collection (Step 5 — marked with ⏸)
- [ ] Does NOT pause between diagram and test possibility listing

---

## Output Format

### EVAL-CR04: Two-level column headers

**Requirement:**
> Registration requires name (text) and age (18–65). Valid registrations show a
> welcome message. Invalid ones show an error.

**User Responses:**
1. Workflow selection → "A"
2. Continue through full workflow

**Evaluation Criteria:**
- [ ] Tables have two header rows
- [ ] Level 1 row: `Input` repeated across name and age columns, `Output` repeated across result columns
- [ ] Level 2 row: actual field names (name, age, Result, Message, etc.)
- [ ] Alignment separator row (|---|) appears after Level 2
- [ ] Format is consistent between UT and AT tables

---

### EVAL-CR05: TC-ID numbering scheme

**Requirement:**
> (Use any single-condition requirement)

**Evaluation Criteria:**
- [ ] Unit Test Cases use UT-01, UT-02, UT-03, ... (zero-padded, sequential)
- [ ] Acceptance Test Cases use AT-01, AT-02, AT-03, ...
- [ ] IDs do not skip numbers
- [ ] IDs restart per table (UT starts at 01, AT starts at 01)

---

### EVAL-CR06: Type column values

**Requirement:**
> (Use any requirement)

**Evaluation Criteria:**
- [ ] Type column contains exactly `Valid` or `Invalid` (not "Pass"/"Fail", "Positive"/"Negative")
- [ ] Valid = input meets all requirements
- [ ] Invalid = input violates at least one requirement

---

## File Saving

### EVAL-CR07: Auto-save after generation

**Requirement:**
> Age verification: users must be 20 years or older.

**User Responses:**
1. Complete full workflow

**Evaluation Criteria:**
- [ ] After generating tables, automatically saves to `test-cases/<feature-name>.md`
- [ ] Feature name is kebab-case (e.g., `test-cases/age-verification.md`)
- [ ] Creates `test-cases/` directory if it doesn't exist
- [ ] File contains all sections per the template in SKILL.md (Requirement, Analysis, Real-world Data, Unit Test Cases, Acceptance Test Cases)
- [ ] Tells the user the file path after saving

---

### EVAL-CR08: File already exists — ask user

**Setup:** Create a file at `test-cases/age-verification.md` before running.

**Requirement:**
> Age verification: users must be 20 years or older.

**User Responses:**
1. Complete full workflow

**Evaluation Criteria:**
- [ ] Detects existing file
- [ ] Asks user whether to overwrite or append
- [ ] Does NOT silently overwrite

---

## Gotchas

### EVAL-CR09: Does not assume unstated rules

**Requirement:**
> The discount applies to orders over 500 THB.

**User Responses:**
1. Workflow selection → "A"
2. Continue through workflow

**Evaluation Criteria:**
- [ ] Only tests the stated condition (order amount > 500)
- [ ] Does NOT invent additional rules (e.g., maximum discount, membership requirement)
- [ ] May ask clarifying questions if ambiguity exists (e.g., "Is 500 THB included or excluded?")
- [ ] Does NOT assume inclusive/exclusive without asking

---

### EVAL-CR10: Informal language extraction

**Requirement:**
> We need to check that people aren't too young or too old for the gym membership.
> I think the rules say something like 16 to 70 years old.

**User Responses:**
1. Workflow selection → "A"

**Evaluation Criteria:**
- [ ] Extracts testable condition: age range 16–70
- [ ] Does NOT dismiss the informal language — treats it as a valid requirement
- [ ] May flag uncertainty (e.g., "you mentioned 'I think' — should I confirm the range is 16–70?")
- [ ] Proceeds with BVA workflow using 16–70 as boundaries
