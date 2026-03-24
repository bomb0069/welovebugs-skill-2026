# Evaluation Tests: wlb-test-engineer

Manual evaluation suite for the `wlb-test-engineer` skill. Each eval file contains
structured test scenarios with sample requirements, simulated user responses, and
evaluation criteria (rubrics).

## Structure

| File | Covers |
|------|--------|
| `eval-routing.md` | Workflow auto-detection and selection |
| `eval-bva-workflow.md` | Workflow A: Boundary Value Analysis |
| `eval-ep-workflow.md` | Workflow B: Equivalence Partitioning |
| `eval-multi-condition-workflow.md` | Workflow C: Multi-Condition (Decision Table + Sequential) |
| `eval-state-transition-workflow.md` | Workflow D: State Transition Testing |
| `eval-common-rules.md` | Cross-workflow rules: language, pause, save, output format |

## How to run

1. Start a new conversation with the `wlb-test-engineer` skill loaded
2. Paste the **Requirement** from a test scenario
3. Provide the **User Responses** at each pause point
4. Check the output against the **Evaluation Criteria**
5. Mark each criterion as Pass / Fail

## Eval scenario format

Each scenario follows this structure:

```
### EVAL-XX: <scenario name>

**Requirement:**
> (the requirement text to paste)

**User Responses:**
1. Step N → "response text"
2. Step M → "response text"

**Evaluation Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2
```

## Scoring

- **Pass**: All criteria met
- **Partial**: 1-2 minor criteria missed (e.g., formatting, but logic correct)
- **Fail**: Key criteria missed (wrong workflow, missing test cases, wrong boundaries)
