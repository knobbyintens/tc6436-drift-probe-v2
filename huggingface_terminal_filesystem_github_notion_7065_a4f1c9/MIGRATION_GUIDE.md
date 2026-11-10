# Tool Registry Consolidation — Migration Guide

Source repository: `knobbyintens/tc6436-drift-probe-v2` (branch `main`).
Task working area inside the repository: `huggingface_terminal_filesystem_github_notion_7065_a4f1c9/`.

## Source layout

- `huggingface_terminal_filesystem_github_notion_7065_a4f1c9/intake/schema_declared/` — declarations from the legacy exporter.
  Each file is one tool declaration whose `parameters` member is already in
  JSON-Schema form: `{"type": ..., "properties": {...}, "required": [...]}`.
  Not every declaration carries a `required` member.
- `huggingface_terminal_filesystem_github_notion_7065_a4f1c9/intake/flat_declared/` — declarations from the edge teams. Each
  file is one tool declaration whose `parameters` member is a flat map:
  `parameter_name -> {"description": ..., "type": ..., "default": ...}`.
  These declarations never carry a `required` member.
- Each declaration carries `tool_id`, `owner`, `tool_name`, `display_name`,
  `summary`, `build_path` and `parameters`.
- `huggingface_terminal_filesystem_github_notion_7065_a4f1c9/builds/<owner>/<tool_name>/` — the implementation package for the
  tool. A build is publishable only when it contains **both**
  `handler/main.py` and `docs/spec.md`.

## Canonical tool entry

```json
{"tool_id": "...", "family": "schema", "parameters": {...}}
```

`family` is `schema` for declarations under `intake/schema_declared/` and
`flat` for declarations under `intake/flat_declared/`.

## Parameter rules

- **schema family** — the declaration is already canonical. Carry the
  `parameters` object through unchanged.
- **flat family** — the declaration is not yet canonical. Produce
  `{"type": "object", "properties": <the flat map as declared>, "required": [...]}`.
  A parameter is optional when its declaration contains a `default` of any
  value (including `null`), or when its `type` string contains the token
  `optional` (case-insensitive). Optional parameters are omitted from
  `required`; all remaining parameters are listed in declaration order.
  Emit an empty list when no parameter is required.

## Catalog files

- `huggingface_terminal_filesystem_github_notion_7065_a4f1c9/release/catalog.json` (and the workspace copy `output/catalog.json`):
  every declared tool, sorted by `tool_id` ascending.
  Shape: `{"registry_version": "2026.11", "tools": [<canonical tool entry>, ...]}`.
- `huggingface_terminal_filesystem_github_notion_7065_a4f1c9/release/tracker.json` (and the workspace copy `output/tracker.json`):
  one verification record per declared tool.
  Shape: `{"generated_for": "tool-registry-consolidation", "tools": [{"tool_id": "...", "status": "verified" | "incomplete", "missing_artifacts": ["handler/main.py", ...]}, ...]}`,
  sorted by `tool_id`. `missing_artifacts` uses the artifact paths relative to
  the build directory and is empty for verified tools.
- `huggingface_terminal_filesystem_github_notion_7065_a4f1c9/release/published/catalog.json` (and the workspace copy
  `output/published/catalog.json`): same shape as `catalog.json`, containing
  only the tools whose status is `verified`.

Local workspace copies are written under the `output/` directory of the
workspace.
