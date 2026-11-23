# Next.js Standards

Stack-coupled standards for services with `framework: nextjs` in their `services/*.yml` entry. The manifest assembler injects this directory's content only when the touched service declares `framework: nextjs`.

## What Goes Here

Conventions that depend on Next.js idioms — App Router page structure, React Query hooks, form patterns with React Hook Form + zod, component composition with forwardRef + CVA, dialog/combobox/toast primitives.

Stack-agnostic concerns (response envelopes, NFR budgets, naming) live in `standards/api/`, `standards/nfr/`, `standards/general/`.

## Naming Convention

`STD-{NNN}-{kebab-name}.md` for promoted standards, `{descriptive-name}.md` for drafts.
