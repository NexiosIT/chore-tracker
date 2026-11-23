# Chore Tracker — Product Hub

Product-hub for **Chore Tracker**, managed with the [Intent Engineering SDLC](https://github.com/rp-one/ai-sdlc) (v0.4).

> **New here?** The workflow map and which-command-when live in [product-hub-docs/WORKFLOW.md](product-hub-docs/WORKFLOW.md). Agents auto-load [CLAUDE.md](CLAUDE.md).

## Getting Started

```bash
cd chore-tracker
ie status # Project overview
ie doctor # Run diagnostics
ie feature create my-first-feature # Create a feature
ie spec create "My Spec" --domain <domain> --service <service> # Create a use case spec
```

## Structure

| Directory | Purpose |
|-----------|---------|
| `product/` | Product context (mission, tech stack, glossary) |
| `domain-model.yml` | Domain registry — context-scoped entity vocabulary. Run `ie domain init`. |
| `integrations/` | Integrations registry — external-service facts. Seeded empty; add entries or run `ie integrations init`. |
| `specs/` | Use case specifications, organized by domain |
| `screens/` | Screen/UI specifications, organized by app then domain |
| `design-system/` | Design tokens — shared `_base` + per-app themes |
| `features/` | Feature lifecycle (active + archive) |
| `standards/` | Coding standards and patterns |
| `services/` | Service registry |
| `product-hub-docs/` | Operator workflow guide (`WORKFLOW.md`) |

## Organization

- **Organization:** kbo
- **Methodology:** Intent Engineering SDLC v0.4
