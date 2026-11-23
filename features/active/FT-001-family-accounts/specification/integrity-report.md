# Integrity Report — FT-001

**Feature:** Family Accounts
**Date:** 2026-11-23
**Grade:** WARNING
**Scope:** 15 specs checked

## How to review this feature

**Review these** — the deliverable:
- `specs/family/UC-001-found-family` — UC-001 (create)
- `specs/family/UC-002-get-family-status` — UC-002 (create)
- `specs/family/UC-003-log-in` — UC-003 (create)
- `specs/family/UC-004-log-out` — UC-004 (create)
- `specs/family/UC-005-get-current-member` — UC-005 (create)
- `specs/family/UC-006-register-kid` — UC-006 (create)
- `specs/family/UC-007-list-pending-registrations` — UC-007 (create)
- `specs/family/UC-008-approve-registration` — UC-008 (create)
- `specs/family/UC-009-reject-registration` — UC-009 (create)
- `specs/family/UC-010-add-parent` — UC-010 (create)
- `specs/family/UC-011-change-own-password` — UC-011 (create)
- `specs/family/UC-012-reset-member-password` — UC-012 (create)
- `specs/family/UC-013-list-members` — UC-013 (create)
- `specs/family/UC-014-deactivate-member` — UC-014 (create)
- `specs/family/UC-015-reactivate-member` — UC-015 (create)
- `screens/chore-tracker/family/SCR-001-setup.yml` — SCR-001 (create)
- `screens/chore-tracker/family/SCR-002-login.yml` — SCR-002 (create)
- `screens/chore-tracker/family/SCR-003-kid-registration.yml` — SCR-003 (create)
- `screens/chore-tracker/family/SCR-004-home.yml` — SCR-004 (create)
- `screens/chore-tracker/family/SCR-005-pending-registrations.yml` — SCR-005 (create)
- `screens/chore-tracker/family/SCR-006-members.yml` — SCR-006 (create)
- `screens/chore-tracker/family/SCR-007-add-parent.yml` — SCR-007 (create)
- `screens/chore-tracker/family/SCR-008-change-password.yml` — SCR-008 (create)
- The findings in this report — each conflict and how it was resolved.
- `shaping/shape.md` — the **Key Decisions** table (boundary & decomposition rationale).

**Skim for context:** `shaping/plan.md` (the implementation plan).

**Not spec-review material** (tool-generated indexes): `manifest.yml`, `shaping/references.md`.

## Warnings (Non-Blocking)

- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-001-setup.yml' (app: chore-tracker) references modified UC 'specs/family/UC-002-get-family-status' — verify data shape assumptions
  - Specs: UC-002
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-002-9739535d` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-001-setup.yml' (app: chore-tracker) references modified UC 'specs/family/UC-001-found-family' — verify data shape assumptions
  - Specs: UC-001
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-001-134cd7c6` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-002-login.yml' (app: chore-tracker) references modified UC 'specs/family/UC-002-get-family-status' — verify data shape assumptions
  - Specs: UC-002
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-002-8c9235b1` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-002-login.yml' (app: chore-tracker) references modified UC 'specs/family/UC-003-log-in' — verify data shape assumptions
  - Specs: UC-003
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-003-85817088` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-003-kid-registration.yml' (app: chore-tracker) references modified UC 'specs/family/UC-006-register-kid' — verify data shape assumptions
  - Specs: UC-006
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-006-b6d3bf78` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-004-home.yml' (app: chore-tracker) references modified UC 'specs/family/UC-005-get-current-member' — verify data shape assumptions
  - Specs: UC-005
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-005-8d9d4b25` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-004-home.yml' (app: chore-tracker) references modified UC 'specs/family/UC-007-list-pending-registrations' — verify data shape assumptions
  - Specs: UC-007
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-007-cb38d7cf` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-004-home.yml' (app: chore-tracker) references modified UC 'specs/family/UC-004-log-out' — verify data shape assumptions
  - Specs: UC-004
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-004-ab4f79bb` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-005-pending-registrations.yml' (app: chore-tracker) references modified UC 'specs/family/UC-007-list-pending-registrations' — verify data shape assumptions
  - Specs: UC-007
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-007-b2662989` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-005-pending-registrations.yml' (app: chore-tracker) references modified UC 'specs/family/UC-008-approve-registration' — verify data shape assumptions
  - Specs: UC-008
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-008-7740e171` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-005-pending-registrations.yml' (app: chore-tracker) references modified UC 'specs/family/UC-009-reject-registration' — verify data shape assumptions
  - Specs: UC-009
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-009-2d8ab477` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-006-members.yml' (app: chore-tracker) references modified UC 'specs/family/UC-013-list-members' — verify data shape assumptions
  - Specs: UC-013
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-013-b2d3d4a1` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-006-members.yml' (app: chore-tracker) references modified UC 'specs/family/UC-014-deactivate-member' — verify data shape assumptions
  - Specs: UC-014
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-014-0ba9f919` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-006-members.yml' (app: chore-tracker) references modified UC 'specs/family/UC-015-reactivate-member' — verify data shape assumptions
  - Specs: UC-015
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-015-191ffead` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-006-members.yml' (app: chore-tracker) references modified UC 'specs/family/UC-012-reset-member-password' — verify data shape assumptions
  - Specs: UC-012
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-012-687ae925` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-007-add-parent.yml' (app: chore-tracker) references modified UC 'specs/family/UC-010-add-parent' — verify data shape assumptions
  - Specs: UC-010
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-010-64bc23e2` to `iron_dome_suppress` in meta.yml
- **[screen-composition]** Screen 'screens/chore-tracker/family/SCR-008-change-password.yml' (app: chore-tracker) references modified UC 'specs/family/UC-011-change-own-password' — verify data shape assumptions
  - Specs: UC-011
  - To suppress (if reviewed & acceptable): add `finding_id: screen-composition-general-UC-011-d45dcd40` to `iron_dome_suppress` in meta.yml

## Reviewer Sign-Off

Completed by the Phase 3 reviewer after evaluating all findings.

- [ ] All conflicts resolved or justified
- [ ] All warnings evaluated and accepted
- **Reviewer:**
- **Date:**
- **Notes:** _(tag any reviewer-added finding with its defect-class slug — module-ownership, seam-existence, prose-scenario, invariant-strength, sibling-parity, rendered-binding, flow-parse — never a bare letter code, so recurrence per class stays measurable)_
