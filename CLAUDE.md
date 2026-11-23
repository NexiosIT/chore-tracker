# Intent Engineering SDLC Product Hub — Operator Notes

> Auto-loaded by Claude Code every session. **Workflow map + which-command-when: [product-hub-docs/WORKFLOW.md](product-hub-docs/WORKFLOW.md).**

This is an **Intent Engineering SDLC product-hub**. Specs are the primary artifact; code is generated from them.

## Where things are
- **First feature, end-to-end** → [product-hub-docs/WALKTHROUGH.md](product-hub-docs/WALKTHROUGH.md) — one worked example through every phase
- **Workflow + command map** → [product-hub-docs/WORKFLOW.md](product-hub-docs/WORKFLOW.md)
- **Per-command detail** → `.claude/commands/ie/<command>.md`
- **Greenfield service setup (Phase 0b)** → ask the person to run `/ie/bootstrap`; it emits the Foundation runbook
- **CLI** → `ie <group> --help` (every command supports `--json`)

## Cardinal rules
- Specs are folders, not files — each format (Gherkin, OpenAPI, Mermaid, Component YAML) is its own file.
- Fix defects in the spec and regenerate; never hand-patch generated code.
- Shaping soft-warns on incomplete service foundation; `/ie/generate` hard-blocks until required gates pass.

<!-- Project-specific rules go below. This file is yours — `ie update` will not overwrite it. -->
