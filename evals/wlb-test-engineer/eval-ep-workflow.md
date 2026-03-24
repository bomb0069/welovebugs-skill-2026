# Eval: EP Workflow (Workflow B)

Tests for Equivalence Partitioning workflow — non-numeric inputs with categories.

---

### EVAL-EP01: Simple category field — independent

**Requirement:**
> The system supports three payment methods: Credit Card, Bank Transfer, and QR Payment.
> Any other payment method should be rejected.

**User Responses:**
1. Workflow selection → "B"
2. Partition confirmation → confirm the proposed partitions
3. Independent/Combined → "Independent" (single field)
4. Real-world data → "Most customers use QR Payment. Some use credit cards. Occasionally someone tries PayPal or cash on delivery."

**Evaluation Criteria:**

Step 1 — Identify fields:
- [ ] Identifies 1 input field: payment method

Step 2 — Partitions:
- [ ] Valid partition: {Credit Card, Bank Transfer, QR Payment}
- [ ] Invalid partition: {any other method}
- [ ] Asks user to confirm partitions

Step 3 — Partition diagram:
- [ ] Produces nested box ASCII diagram
- [ ] Shows valid territory and invalid territory
- [ ] Valid box contains the 3 accepted methods
- [ ] Invalid box shows examples of rejected methods

Step 4 — Representative values:
- [ ] Selects one representative from each partition
- [ ] Valid: picks one of the three (e.g., Credit Card)
- [ ] Invalid: picks a realistic non-accepted method (e.g., PayPal)

Step 5 — Expected outputs:
- [ ] Valid payment → specific success behavior
- [ ] Invalid payment → specific rejection message

Step 6 — Real-world data:
- [ ] Pauses (⏸) to collect real-world data

Step 7 — Final tables:
- [ ] UT table covers all valid partitions + at least one invalid partition
- [ ] AT table uses user-provided data (QR Payment, Credit Card, PayPal, cash on delivery)
- [ ] Two-level column headers present
- [ ] Type column: Valid for accepted methods, Invalid for rejected

---

### EVAL-EP02: Multiple fields — combined partitions

**Requirement:**
> User role determines access level:
> - Admin users can access all modules
> - Editor users can access Content and Media modules only
> - Viewer users can only access Content module in read-only mode
> The system also has a maintenance mode. In maintenance mode, only Admin users can log in.

**User Responses:**
1. Workflow selection → "B"
2. Partition confirmation → confirm
3. Independent/Combined → "Combined" (role × mode interact)
4. Real-world data → "Most users are Editors during normal operation. Maintenance happens monthly."

**Evaluation Criteria:**

Step 2b — Dependency check:
- [ ] Asks whether role and mode are independent or combined
- [ ] User confirms combined (maintenance mode affects access by role)

Combined partition analysis:
- [ ] Builds combined partition summary table (role × mode combinations)
- [ ] Shows overlapping sets diagram (Venn-style)
- [ ] Identifies all valid combinations (Admin+normal, Admin+maintenance, Editor+normal, Viewer+normal)
- [ ] Identifies invalid combinations (Editor+maintenance, Viewer+maintenance)

Step 7 — Final tables:
- [ ] UT table covers both valid and invalid combinations
- [ ] Each row shows both role AND mode input values
- [ ] Output columns describe specific access behavior per combination

---

### EVAL-EP03: Boolean/binary partition

**Requirement:**
> Premium subscribers can download content in HD quality.
> Free users can only stream in standard quality and cannot download.

**User Responses:**
1. Workflow selection → "B"
2. Partition confirmation → confirm
3. Real-world data → "90% of users are Free. Premium users download 3-4 times per week."

**Evaluation Criteria:**
- [ ] Identifies subscription type as the input field
- [ ] Valid partitions: {Premium} and {Free} (both are valid inputs, different behaviors)
- [ ] Does NOT treat Free as "invalid" — both are valid subscription types with different outputs
- [ ] UT table shows different expected behaviors for each partition
- [ ] Output columns describe: access level (HD/Standard), download capability (Yes/No)

---

### EVAL-EP04: Field with many categories

**Requirement:**
> Supported file upload types: PNG, JPG, GIF, SVG for images; MP4, AVI, MOV for video;
> PDF, DOCX for documents. Maximum file size is 10 MB for images, 100 MB for video,
> and 25 MB for documents. Unsupported types should show an error.

**User Responses:**
1. Workflow selection → "B" (if recommended) or "C" (if detected as multi-condition)

**Evaluation Criteria:**
- [ ] Recognizes this may be multi-condition (file type + file size interact)
- [ ] If user selects B: groups file types into meaningful partitions (Image, Video, Document, Unsupported)
- [ ] Does not create one partition per file extension — groups by category
- [ ] If detected as C: correctly identifies file type (EP) and file size (BVA) as separate conditions
