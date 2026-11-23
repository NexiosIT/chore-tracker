# Flutter Standards

Stack-coupled standards for services with `framework: flutter` in their `services/*.yml` entry. The manifest assembler injects this directory's content only when the touched service declares `framework: flutter`, which here is `chore-tracker-app`.

## What Goes Here

Conventions that depend on Flutter and Dart idioms: Material 3 theming, widget and folder structure, how the app calls the backend API, local notifications and background work on Android, and widget tests.

Stack-agnostic concerns (response envelopes, error categories, NFR budgets, naming) live in `standards/api/`, `standards/nfr/` and `standards/general/`.

## Naming Convention

`STD-{NNN}-{kebab-name}.md` for promoted standards, `{descriptive-name}.md` for drafts.

## Status

Seeded with a single starter standard (STD-043) during bootstrap. More Flutter standards are expected to come out of the pathfinder feature.
