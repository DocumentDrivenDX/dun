---
dun:
  id: F-024
  depends_on:
    - helix.prd
    - F-002
    - F-017
    - F-018
---
# Feature Spec: F-024 Prompt Evaluation

## Summary

Add a prompt evaluation capability to Dun so teams can measure whether their
prompts (skills, workflow instructions, system prompts) actually produce the
intended results. Capture a prompt, define evaluation criteria, generate test
outputs, and score them — all in a repeatable loop.

## Motivation

Helix ships ~40 skill definitions and ~40 workflow prompts. These prompts are
hand-authored and evolve over time, but there is no systematic way to answer:
are they actually good? Do they produce artifacts that meet their own completion
criteria? Prompt evaluation is a quality check — it fits Dun's domain.

## Requirements

- `dun eval` accepts a prompt file and a rubric file (evaluation criteria).
- The rubric is a markdown or YAML file listing quality dimensions, each with a
  description and weight. Rubrics may also reference the prompt's own completion
  criteria when present.
- Evaluation runs in two phases:
  1. **Generate**: run the prompt through a harness to produce a test output.
     Optionally accept a pre-generated output file instead.
  2. **Judge**: run the output + rubric through one or more evaluator harnesses
     to produce scored assessments.
- Support multi-agent judging via the quorum system (F-018): dispatch to
  multiple evaluator harnesses, synthesize scores.
- Each evaluation produces a structured result: per-dimension scores, overall
  score, evidence excerpts, and a recommendation (pass / marginal / fail).
- Results are emitted in Dun's standard output formats (prompt envelope, LLM
  summary, JSON) per F-002.
- Support batch mode: evaluate multiple prompts against the same or different
  rubrics in a single invocation.
- Support iteration: `dun loop` can include eval checks so prompt quality is
  part of the autonomous feedback loop (F-017).
- Evaluations are deterministic for the same prompt, rubric, and harness set.
- Rubrics can be stored alongside prompts (e.g. `prompt.md` + `rubric.md`) or
  in a shared rubrics directory.

## CLI Surface

```
dun eval <prompt> [--rubric <file>] [--output <file>] [options]
dun eval <prompt> --generate-only    # produce output, skip judging
dun eval <prompt> --judge-only --output <file>  # judge existing output
dun eval --batch <glob>              # evaluate multiple prompts
```

Flags inherited from existing commands:
- `--harness` / `--harnesses` — harness(es) for generation and judging
- `--quorum` — quorum strategy for multi-judge consensus
- `--automation` — manual/plan/auto/yolo
- `--format` — output format (prompt/llm/json)
- `--verbose` — print prompts and responses

## Rubric Format

```yaml
# rubric.md or rubric.yaml
dimensions:
  - name: completeness
    description: All required sections are present and substantive
    weight: 3
  - name: actionability
    description: Output contains concrete next steps, not vague guidance
    weight: 2
  - name: consistency
    description: Output is internally consistent and aligned with inputs
    weight: 2
  - name: conciseness
    description: No filler, no redundant sections
    weight: 1

pass_threshold: 7  # out of 10
```

When no rubric file is provided, Dun extracts evaluation criteria from the
prompt's own completion criteria, quality checks, or acceptance criteria
sections if present.

## Eval Result Shape

```json
{
  "prompt": "skills/helix-design/SKILL.md",
  "rubric": "skills/helix-design/rubric.yaml",
  "overall_score": 7.2,
  "recommendation": "pass",
  "dimensions": [
    {
      "name": "completeness",
      "score": 8,
      "weight": 3,
      "evidence": "All required sections present..."
    }
  ],
  "judges": ["claude", "gemini"],
  "quorum": "majority",
  "generated_output": "eval-outputs/helix-design-2026-04-06T12:00:00.md"
}
```

## Acceptance Criteria

- `dun eval prompt.md --rubric rubric.yaml` produces a scored evaluation.
- `dun eval prompt.md` without `--rubric` extracts criteria from the prompt's
  own completion/quality sections.
- `dun eval prompt.md --generate-only` produces output without judging.
- `dun eval prompt.md --judge-only --output out.md` judges a pre-existing output.
- `dun eval --batch "skills/*/SKILL.md"` evaluates multiple prompts.
- Multi-judge mode via `--harnesses claude,gemini --quorum majority` synthesizes
  scores across evaluators.
- Results are emitted in prompt, LLM, and JSON formats.
- Overall score is a weighted average of dimension scores.
- Recommendation is derived from pass_threshold in rubric.
- Eval checks can be included in `dun loop` iterations.

## Gaps & Conflicts

- Depends on F-018 (quorum) for multi-judge synthesis — can operate in
  single-judge mode without it.
- Depends on F-002 (output formats) for structured result emission.
- Depends on F-017 (autonomous iteration) for loop integration.
- Generation phase requires a harness; if no harness is available, only
  `--judge-only` with a pre-existing output is possible.
- Rubric authoring guidance and starter rubrics for common prompt types
  (skills, workflow instructions, system prompts) would improve adoption
  but are not blocking.

## Traceability

- Extends PRD goal: "One command that discovers and runs the right checks" to
  cover prompt quality as a check type.
- Supports PRD goal: "Deterministic, agent-friendly output."
- Builds on existing `dun review` pattern (multi-agent review with synthesis
  and scoring) applied to prompts instead of documents.
