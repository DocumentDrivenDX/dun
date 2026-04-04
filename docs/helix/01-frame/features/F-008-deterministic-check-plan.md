---
dun:
  id: F-008
  depends_on:
    - helix.prd
    - F-001
    - F-003
---
# Feature Spec: F-008 Deterministic Check Plan

## Summary

Produce a stable, deterministic ordering of checks so that repeated runs
against the same repo state yield identical check plans. This ensures
predictable output for agents and operators.

## Requirements

- Sort the check plan deterministically after discovery and filtering.
- Order by plugin priority (manifest `priority` field, default 50).
- Break ties by check priority (check-level `priority` field, default 50).
- Break further ties by HELIX phase order (frame < design < test < build <
  deploy < iterate).
- Break final ties alphabetically by check ID.
- Produce identical ordering for the same set of active plugins and checks.

## Inputs

- Active plugins with their manifest priority values.
- Planned checks with per-check priority, phase, and ID fields.

## Acceptance Criteria

- `sortPlan` produces identical ordering across multiple runs for the same
  input.
- Plugin priority takes precedence over check priority.
- Check priority takes precedence over phase order.
- Phase order takes precedence over alphabetical ID.
- Default priority (50) is applied when priority is unset.
- Empty or single-element plans do not panic.

## Gaps & Conflicts

- No gaps identified. The implementation in `engine.go:sortPlan` fully
  satisfies the requirements above.
- No conflicts with governing artifacts.

## Traceability

- Supports PRD goal for "deterministic, agent-friendly output that is easy
  to parse" by ensuring stable check ordering.
- Supports F-001 Auto-Discovery acceptance criterion: "Check IDs and ordering
  are stable across runs for the same repo state."
- Supports F-003 Plugin System acceptance criterion: "Plugin discovery and
  ordering are deterministic for a given repo state."
- Implementation: `internal/dun/engine.go:sortPlan`.
- Tests: `internal/dun/engine_sort_test.go`.
