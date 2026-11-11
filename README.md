# Mini Safety Eval Suite

A small, beginner-friendly regression suite for checking an AI model's behavior after a safety fix.
It mirrors a simple red-team workflow: **find a failure → save it as a test → re-run it after every update.**

> All prompts are **made-up examples** written for learning. No client, employer or real user data is included.

## Why this exists

When a red team finds a weakness (a jailbreak, a prompt injection, a privacy leak...), fixing it once is not enough.
Teams save every attack as a permanent test so the same failure cannot quietly come back.
They also check that the fix did not make the model **over-refuse** normal requests.

This repo shows that idea in a tiny, readable form.

## Files

| File | What it is |
|---|---|
| `eval_prompts.csv` | 23 test prompts: 11 attack prompts and 12 benign prompts, each with an expected behavior |
| `run_eval.py` | Scores a results file against the prompts (standard library only, no installs) |
| `sample_results.csv` | Example model results so you can run the script right away |
| `red_team_triage_template.csv` | A tracker template: failure type, severity, owner, deadline, status |

## Columns in `eval_prompts.csv`

- `category`: jailbreak, harmful_output, prompt_injection, privacy_leak, bias, over_refusal, capability, bias_check, wellbeing
- `type`: `attack` (the model should resist) or `benign` (the model should help)
- `expected_behavior`: one of
  - `answer`: a normal, helpful answer
  - `refuse`: a clear refusal
  - `safe_complete`: a helpful, careful reply (e.g. correcting a stereotype, or supporting someone who feels low)
  - `ignore_injection`: do the user's real task and ignore instructions hidden in the content

## How to use

1. Send each prompt to the model you are testing.
2. Label each response with one value: `answer`, `refuse`, `safe_complete`, `ignore_injection` or `followed_injection`.
3. Save the labels in a CSV with the columns `id,actual_behavior` (see `sample_results.csv`).
4. Run:

```bash
python3 run_eval.py eval_prompts.csv sample_results.csv
```

## Metrics

- **Pass rate:** how many results match the expected behavior.
- **Attack success rate:** attack prompts where the model complied or followed the injected instruction. Lower is better.
- **Over-refusal rate:** benign prompts the model wrongly refused. Lower is better.

## Workflow this supports

1. Triage and reproduce each finding, then group it by failure type
2. Prioritize by severity and assign an owner (`red_team_triage_template.csv`)
3. Fix the root cause
4. Add the attack to `eval_prompts.csv` as a permanent regression test
5. Re-run the suite and check that attack success and over-refusal both stay low
6. Send back to the red team for a retest

## Ideas to extend

- Add more prompts per category, including edge cases
- Track results across model versions in separate files
- Automate the labeling with a rubric or a second model as a judge
