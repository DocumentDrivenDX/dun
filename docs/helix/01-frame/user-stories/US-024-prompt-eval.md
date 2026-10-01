---
dun:
  id: US-024
  depends_on:
  - F-024
---
# US-024: Evaluate Prompt Quality

As a prompt author, I want to evaluate whether my prompts produce good results
so I can iterate on them with evidence instead of guesswork.

## Acceptance Criteria

- `dun eval skills/helix-design/SKILL.md` runs the prompt through a harness,
  judges the output, and reports a score.
- A rubric file defines quality dimensions with weights and a pass threshold.
- When no rubric is provided, criteria are extracted from the prompt's own
  completion criteria or quality checks sections.
- `--generate-only` produces a test output without judging it.
- `--judge-only --output out.md` judges a pre-existing output against the
  rubric.
- `--batch "skills/*/SKILL.md"` evaluates multiple prompts in one invocation.
- Multi-judge consensus via `--harnesses claude,gemini --quorum majority`
  increases confidence in scores.
- Results include per-dimension scores, evidence excerpts, overall score, and a
  pass/marginal/fail recommendation.
- Results are available in prompt, LLM, and JSON output formats.

## Example Usage

```bash
# Evaluate a single prompt with default rubric extraction
dun eval skills/helix-design/SKILL.md

# Evaluate with an explicit rubric
dun eval skills/helix-design/SKILL.md --rubric rubrics/skill.yaml

# Generate output only (inspect manually before judging)
dun eval skills/helix-design/SKILL.md --generate-only

# Judge a previously generated output
dun eval skills/helix-design/SKILL.md --judge-only --output eval-outputs/design.md

# Batch evaluate all skills with multi-judge consensus
dun eval --batch "skills/*/SKILL.md" \
  --harnesses claude,gemini --quorum majority

# Include in autonomous loop
dun loop --only "eval-*"
```

## Design Notes

- The generate and judge phases are independent — this enables workflows where
  a human reviews generated output before scoring, or where output is generated
  once and judged by many evaluators.
- Rubrics are intentionally simple (name, description, weight, threshold) so
  they're easy to author and version alongside prompts.
- Weighted scoring surfaces which dimensions are weak, guiding targeted prompt
  improvements rather than vague "make it better" iteration.
- Batch mode enables regression testing: after editing a prompt, re-evaluate
  the full suite to check for regressions in other prompts.
