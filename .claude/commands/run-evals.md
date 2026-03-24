# Run Skill Evaluations

Run evaluation tests for a specified skill using its `evals/evals.json` configuration.

## Arguments

`$ARGUMENTS` — format: `[skill-name] [eval-id ...]`

- All arguments are optional
- If **no arguments**: discover all skills that have evals and run all of them
- If **first argument matches a skill name** (has `evals/<name>/evals.json`): run evals for that skill
- If **first argument does NOT match a skill name**: treat all arguments as eval IDs and run them across all skills
- Remaining arguments after skill name: specific eval IDs to run

Examples:
- `/run-evals` — run ALL evals for ALL skills that have evals
- `/run-evals wlb-test-engineer` — run all evals for wlb-test-engineer
- `/run-evals wlb-test-engineer BVA01` — run single eval
- `/run-evals wlb-test-engineer R01 R02 R03` — run specific set
- `/run-evals wlb-sdet JAVA01 PY01` — run evals for wlb-sdet skill

## Setup

1. Parse `$ARGUMENTS`:
   - If empty: scan `evals/` directory for all subdirectories containing `evals.json`. Collect all skill names found (e.g., `wlb-test-engineer`, `wlb-sdet`). Run all evals for each.
   - If first word matches an `evals/<word>/evals.json`: use it as skill name, remaining words are eval IDs
   - Otherwise: scan all skills and filter by the provided eval IDs across all skills
2. For each skill to evaluate:
   a. Verify the skill exists at `skills/<skill-name>/SKILL.md` — warn and skip if not found
   b. Read `evals/<skill-name>/evals.json` and parse `eval_config`
   c. Resolve the skill path from `eval_config.skill_path` (default: `skills/<skill-name>`)
   d. Create the results directory: `evals/<skill-name>/results/<timestamp>/`
   e. If eval IDs were specified, filter to only those IDs

## Processing each eval

For each eval in the `evals` array:

### Single-turn evals (`type: "single_turn"`)

1. Spawn a subagent with the skill loaded from `skills/<skill-name>`
2. Send the `prompt` as a single message
3. Collect the full response
4. Run assertions against the response
5. Save results

### Multi-turn evals (`type: "multi_turn"`)

1. Spawn a subagent with the skill loaded from `skills/<skill-name>`
2. Process the `turns` array sequentially:
   - For each `user` turn: send the `content` as a message
   - For each `assistant` turn: collect the response and store it keyed by the `label`
3. After all turns complete, run `turn_assertions` and `final_assertions`
4. Save results

**Important**: Each subagent must start with a clean context. The skill must be the only loaded skill.

## Running assertions

### Deterministic assertions

Execute these directly with code — do NOT use LLM judgment:

| `check` | Logic |
|---------|-------|
| `response_contains` | Case-insensitive search for `value` in the turn's response text |
| `response_not_contains` | Verify `value` is NOT in the turn's response text |
| `response_contains_question_mark` | Check if response text contains `?` |
| `response_matches_regex` | Match `pattern` (regex) against the turn's response text |
| `file_exists` | Glob for `pattern` in the working directory, pass if ≥1 match |
| `file_contains` | Find file matching `file` glob, search for `value` in content |
| `file_matches_regex` | Find file matching `file` glob, match `pattern` (regex) against content |

For `turn_assertions`: the response text is the assistant response stored under the `turn` label.
For `final_assertions`: check across the full conversation output and any generated files.

### LLM-graded assertions

For each `llm_graded` assertion:

1. Construct a grading prompt:

```
You are a strict eval grader. Evaluate whether this assertion is satisfied by the output.

**Assertion**: {assertion.check}

**Output to evaluate**:
{the assistant response for the specified turn, or full conversation for final_assertions}

Respond with ONLY valid JSON:
{
  "passed": true/false,
  "evidence": "specific quote or observation from the output (max 200 chars)"
}

Rules:
- Require concrete evidence for PASS — do not give benefit of the doubt
- If the assertion is about something NOT being present, verify it is truly absent
- If unsure, mark as FAIL
```

2. Run this grading prompt 3 times (as configured in `eval_config.grader_runs`)
3. Use **majority vote**: if 2 out of 3 say PASS → PASS, otherwise FAIL
4. Record all 3 evidence strings

## Saving results

For each eval, save `grading.json` to `results/<timestamp>/<eval-id>/grading.json`:

```json
{
  "eval_id": "BVA01",
  "eval_name": "BVA: basic age range with integer boundaries",
  "type": "multi_turn",
  "assertion_results": [
    {
      "turn": "boundaries_and_realworld_ask",
      "type": "deterministic",
      "check": "response_contains",
      "value": "17",
      "passed": true,
      "evidence": "Found '17' in response"
    },
    {
      "turn": "final_output",
      "type": "llm_graded",
      "check": "both tables have two-level column headers",
      "passed": true,
      "evidence": "Row 1 shows '| Input | Input | Output | Output |', Row 2 shows field names",
      "grader_votes": [true, true, false],
      "grader_evidence": ["evidence1", "evidence2", "evidence3"]
    }
  ],
  "summary": {
    "passed": 15,
    "failed": 3,
    "total": 18,
    "pass_rate": 0.83,
    "deterministic_passed": 10,
    "deterministic_total": 12,
    "llm_graded_passed": 5,
    "llm_graded_total": 6
  }
}
```

## After all evals complete

1. Aggregate results into `results/<timestamp>/benchmark.json`:

```json
{
  "timestamp": "2026-03-25T14:30:00",
  "skill_name": "<skill-name>",
  "total_evals": 29,
  "evals_run": 29,
  "overall": {
    "pass_rate": 0.85,
    "deterministic_pass_rate": 0.92,
    "llm_graded_pass_rate": 0.78,
    "total_assertions": 178,
    "total_passed": 152
  },
  "by_category": {
    "<category>": { "pass_rate": 0.90, "evals": ["ID1", "ID2"] }
  },
  "per_eval": [
    { "id": "R01", "name": "...", "pass_rate": 1.0, "passed": 5, "failed": 0 }
  ]
}
```

Categories are auto-detected by stripping trailing digits from eval IDs (e.g., `BVA01` → `bva`, `R03` → `routing`, `CR05` → `common_rules`).

2. Print a summary table per skill. When running multiple skills, print one table per skill then a grand total:

```
╔══════════════════════════════════════════════════════╗
║           <skill-name> Eval Results                  ║
╠══════════════════════════════════════════════════════╣
║ Category         │ Pass Rate │ Passed │ Failed       ║
║──────────────────┼───────────┼────────┼──────────────║
║ Routing (7)      │    90%    │  18/20 │ 2            ║
║ BVA (4)          │    88%    │  35/40 │ 5            ║
║ ...              │    ...    │  ...   │ ...          ║
║──────────────────┼───────────┼────────┼──────────────║
║ OVERALL (29)     │    85%    │ 152/178│ 26           ║
╚══════════════════════════════════════════════════════╝

Results saved to: evals/<skill-name>/results/<timestamp>/
```

When multiple skills were run, also print:

```
╔══════════════════════════════════════════════════════╗
║              All Skills Summary                      ║
╠══════════════════════════════════════════════════════╣
║ Skill                │ Pass Rate │ Passed │ Failed   ║
║──────────────────────┼───────────┼────────┼──────────║
║ wlb-test-engineer    │    85%    │ 152/178│ 26       ║
║ wlb-sdet             │    90%    │  45/50 │ 5        ║
║──────────────────────┼───────────┼────────┼──────────║
║ TOTAL                │    86%    │ 197/228│ 31       ║
╚══════════════════════════════════════════════════════╝
```

3. If any eval has failures, list the top failed assertions with their evidence.

## Error handling

- If a subagent fails or times out, mark all assertions for that eval as FAIL with evidence "subagent error: {message}"
- If a file glob matches no files for `file_exists`/`file_contains`/`file_matches_regex`, mark as FAIL with evidence "no files matching pattern: {pattern}"
- Continue to the next eval on failure — do not stop the entire run

## Important notes

- Each eval subagent runs in isolation — do not share state between evals
- Clean up any `test-cases/` files generated by previous evals before starting a new one
- The skill path is resolved from `eval_config.skill_path` or defaults to `skills/<skill-name>`
- For multi-turn evals, the turn `label` is the key used to look up the response when evaluating `turn_assertions`
- When running all skills (`/run-evals` with no args), process them sequentially to avoid resource contention
