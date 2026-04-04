---
dun:
  id: F-012
  depends_on:
    - helix.prd
    - F-003
---
# Feature Spec: F-012 External Plugin Loading

## Summary

Allow users to extend Dun with external plugins loaded from user-level and
project-level directories. Project plugins override user plugins with the
same ID, enabling per-repo customization.

## Requirements

- Load external plugins from user directory (`~/.dun/plugins/*/plugin.yaml`).
- Load external plugins from project directory (`.dun/plugins/*/plugin.yaml`).
- Project plugins override user plugins with the same plugin ID.
- Skip invalid plugin directories with a warning log, without failing the
  overall load.
- Require `id` and `version` fields in plugin manifests.
- Merge external plugins with built-in plugins during discovery.

## Inputs

- User home directory for user-level plugin path.
- Project working directory for project-level plugin path.
- Plugin manifest files (`plugin.yaml`) in each plugin subdirectory.

## Acceptance Criteria

- Plugins in `~/.dun/plugins/*/plugin.yaml` are loaded and available.
- Plugins in `.dun/plugins/*/plugin.yaml` are loaded and available.
- A project plugin with the same ID as a user plugin replaces it.
- Invalid plugin directories are skipped with a warning, not an error.
- Missing plugin directories are silently ignored.
- Loaded external plugins participate in normal discovery and plan building.

## Gaps & Conflicts

- No gaps identified. The implementation in `plugin_loader.go` fully
  satisfies the requirements above.
- No conflicts with governing artifacts.

## Traceability

- Supports PRD scope: "Extensible plugin system for future workflows."
- Supports F-003 Plugin System: "Provide an extensible plugin system."
- Implementation: `internal/dun/plugin_loader.go:LoadExternalPlugins`.
- Tests: `internal/dun/plugin_loader_test.go:TestLoadExternalPlugins`.
