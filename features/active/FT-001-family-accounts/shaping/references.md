# References — FT-001: Family Accounts

## Internal Paths

- `features/briefs/family-accounts/requirements.md` — the brief's requirements and WHEN/SHALL acceptance criteria; source of the In Scope capabilities
- `features/briefs/family-accounts/decisions.md` — the brief's decision catalog; source of the Key Decisions table
- `product/roadmap.md` — theme T-04 that this feature delivers
- `product/tech-stack.md` — Kotlin/Spring Boot API, Flutter Android app, PostgreSQL; authentication mechanism left open
- `product/flows/` — the kid-registration flow the brief changes
- `domain-model.yml` — domain registry; family-member and registration entities, and the target of the registry change requests
- `services/chore-tracker-api.yml` — backend service owning the family domain
- `services/chore-tracker-app.yml` — Android app service rendering the family screens
- `specs/family/` — the 15 use case specs UC-001 to UC-015 authored by this feature
- `screens/chore-tracker/family/` — the 8 screen specs SCR-001 to SCR-008 authored by this feature
- `screens/chore-tracker/_sitemap.yml` — routes of the 8 screens
- `design-system/_base/` and `design-system/chore-tracker/` — tokens and components the screens bind to
- `standards/api/error-handling.md` — STD-002; its `rate_limit` category (1.2.0) covers the 429 lockout answer of UC-003
- `standards/api/response-envelope.md` — STD-001; envelope applied to every contract
- `standards/api/STD-024-contract-conformance.md` — STD-024; shared schemas defined identically across contracts
- `config/readiness-rubric.yml` — readiness rubric the specs are scored against

## External URLs

- None.

## Upstream Ticket

- **T-04** — Roadmap theme "Every family member has their own login, with parents in control of who joins"; briefed in the family-accounts brief (merged via NexiosIT/chore-tracker#3).

## Prior Art

- None. This is the first feature on a greenfield product.
