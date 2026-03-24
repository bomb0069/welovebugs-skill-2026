# Eval: Workflow Routing

Tests that the skill correctly auto-detects which workflow to recommend based on
the requirement content.

---

### EVAL-R01: Single numeric condition → BVA

**Requirement:**
> Customers must be between 18 and 65 years old to register for the service.

**User Responses:**
1. Workflow selection → "A"

**Evaluation Criteria:**
- [ ] Identifies exactly 1 condition (age)
- [ ] Classifies it as numeric
- [ ] Recommends technique: BVA
- [ ] Recommends workflow A with ★ marker
- [ ] Shows all four workflow options (A, B, C, D)
- [ ] Condition table has columns: #, Condition, Field, Type, Technique

---

### EVAL-R02: Single non-numeric condition → EP

**Requirement:**
> The system accepts documents in PDF, DOCX, and XLSX formats. All other file types
> should be rejected with an error message.

**User Responses:**
1. Workflow selection → "B"

**Evaluation Criteria:**
- [ ] Identifies exactly 1 condition (file format/type)
- [ ] Classifies it as non-numeric (category)
- [ ] Recommends technique: EP
- [ ] Recommends workflow B with ★ marker
- [ ] Shows all four workflow options (A, B, C, D)

---

### EVAL-R03: Multiple mixed conditions → Multi-Condition

**Requirement:**
> Loan eligibility: applicant must be 21-60 years old, have an income of at least
> 30,000 THB per month, and hold either Thai nationality or permanent residency.

**User Responses:**
1. Workflow selection → "C"

**Evaluation Criteria:**
- [ ] Identifies 3 conditions (age, income, nationality/residency)
- [ ] Classifies age as numeric (BVA), income as numeric (BVA), nationality as non-numeric (EP)
- [ ] Recommends workflow C with ★ marker
- [ ] Condition table shows all 3 conditions with correct types and techniques

---

### EVAL-R04: State/status language → State Transition

**Requirement:**
> A leave request starts as Draft. The employee submits it, changing status to
> Pending Approval. The manager can Approve or Reject it. If rejected, the employee
> can edit and resubmit. Approved requests move to Confirmed.

**User Responses:**
1. Workflow selection → "D"

**Evaluation Criteria:**
- [ ] Detects state/transition language (Draft, Pending Approval, Approved, Rejected, Confirmed)
- [ ] Recommends workflow D with ★ marker
- [ ] Shows state summary table (not condition table) with states and transitions
- [ ] Counts states and transitions correctly in the recommendation line

---

### EVAL-R05: Ambiguous — two numeric conditions → Multi-Condition (not BVA)

**Requirement:**
> Product pricing: items weighing between 0.5 kg and 30 kg cost 50 THB for shipping.
> Items priced over 5,000 THB get free shipping regardless of weight.

**User Responses:**
1. Workflow selection → "C"

**Evaluation Criteria:**
- [ ] Does NOT recommend workflow A (there are 2 conditions, not 1)
- [ ] Identifies 2 conditions (weight, price)
- [ ] Classifies both as numeric (BVA)
- [ ] Recommends workflow C with ★ marker

---

### EVAL-R06: User overrides recommendation

**Requirement:**
> Customers must be between 18 and 65 years old to register for the service.

**User Responses:**
1. Workflow selection → "C" (overriding recommended A)

**Evaluation Criteria:**
- [ ] Originally recommends workflow A
- [ ] Accepts user override to workflow C without complaint
- [ ] Proceeds with multi-condition workflow (loads multi-condition-workflow.md)

---

### EVAL-R07: Informal business language — extract testable behavior

**Requirement:**
> We need to make sure that only Gold and Platinum members can access the VIP lounge.
> Silver and Basic members should see a message saying they need to upgrade.

**User Responses:**
1. Workflow selection → "B"

**Evaluation Criteria:**
- [ ] Extracts testable condition from informal language (membership level)
- [ ] Classifies as non-numeric (category)
- [ ] Recommends workflow B (EP)
- [ ] Does not assume additional unstated rules
