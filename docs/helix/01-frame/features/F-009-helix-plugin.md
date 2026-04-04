---
dun:
  id: F-009
  depends_on:
    - helix.prd
    - F-003
---
# Feature Spec: F-009 Helix Plugin

## Summary

Provide a built-in Helix plugin that validates HELIX workflow artifacts and
enforces phase gates when `docs/helix/` is present. The plugin ships embedded
in the Dun binary and activates automatically via auto-discovery.

## Requirements

- Activate when `docs/helix/` exists in the repository root.
- Validate HELIX workflow state progression via state rules.
- Validate HELIX phase gates (input and exit gates for all six phases).
- Provide agent prompts for architecture creation, feature spec creation,
  spec alignment, and stack reconciliation.
- Verify spec-to-implementation bindings via spec-binding checks.
- Detect missing or stale Helix docs via doc-DAG checks.
- Verify downstream updates when upstream docs change via change-cascade
  checks.
- Verify integration contracts and dependencies.
- Detect overlapping agent work via conflict-detection checks.

## Inputs

- Repository file tree (trigger: `docs/helix/` path exists).
- HELIX artifact files: PRD, architecture, feature specs, user stories,
  solution designs, technical designs, test plans.
- Phase gate definitions in `gates/` YAML files.
- State rules in `state-rules.yml`.
- Agent prompt templates in `prompts/`.

## Acceptance Criteria

- Helix plugin activates when `docs/helix/` exists.
- State-rules check validates workflow progression.
- Gate checks validate input and exit gates for all six HELIX phases
  (frame, design, test, build, deploy, iterate).
- Agent prompts emit prompt envelopes for architecture creation, feature spec
  creation, spec alignment, and stack reconciliation.
- Spec-binding check verifies user story implementations.
- Doc-DAG check detects missing or stale docs.
- Change-cascade check detects drift when PRD changes.
- Plugin runs without custom configuration.

## Gaps & Conflicts

- No gaps identified. The plugin manifest at
  `internal/plugins/builtin/helix/plugin.yaml` defines all checks listed
  above.
- Integration-contract and conflict-detection checks depend on optional
  artifacts (`integration-map.yaml`, `work-in-progress.yaml`) and activate
  conditionally.
- No conflicts with governing artifacts.

## Traceability

- Supports PRD scope: "Built-in Helix plugin for doc and gate validation."
- Supports F-003 Plugin System: "Include a built-in Helix plugin for
  documentation and gate validation."
- Supports F-001 Auto-Discovery: "Detect Helix workflow via `docs/helix/`."
- Implementation: `internal/plugins/builtin/helix/plugin.yaml` and
  associated gate, state-rule, and prompt files.
- Tests: plugin activation tested via `internal/dun/engine_test.go`.
