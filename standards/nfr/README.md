# NFR Standards

Non-functional requirement patterns that apply across features and domains.

## What Goes Here

Standards for repeatable non-functional concerns:
- **Performance budgets** — response times, load times, bundle sizes, rendering thresholds
- **Accessibility rules** — WCAG compliance, keyboard navigation, screen reader support
- **Interaction patterns** — focus management, virtualization thresholds, debounce behavior
- **Error resilience** — graceful degradation, offline behavior, retry strategies
- **Observability** — logging standards, metrics collection, tracing requirements

## What Does NOT Go Here

Feature-specific NFRs belong in `constraints.md` within the feature folder, not here. Standards capture **repeatable** patterns that apply to every feature.

## Naming Convention

`STD-{NNN}-{kebab-name}.md` for promoted standards, `{descriptive-name}.md` for drafts.

## File Format

Each standard uses YAML front matter + Markdown body. See existing standards in sibling directories for the template.
