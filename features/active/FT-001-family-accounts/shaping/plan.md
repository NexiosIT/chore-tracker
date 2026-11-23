# Implementation Plan — FT-001: Family Accounts

## Approach

All behaviour lives in the family context of the chore-tracker-api service (Kotlin, Spring Boot, PostgreSQL). The Android app (chore-tracker-app, Flutter) renders eight screens on top of it. No other service and no external system is involved.

**Entities touched**

| Entity | Status | Written by | Read by |
|--------|--------|-----------|---------|
| family-member | registered | UC-001 (create), UC-008 (create), UC-010 (create), UC-011, UC-012 (password), UC-014, UC-015 (status) | UC-002, UC-003, UC-005, UC-006, UC-007, UC-013, and every authorized UC to check the caller's role |
| registration | registered | UC-006 (create), UC-008, UC-009 (decide) | UC-003 (pending and rejected messages), UC-006, UC-010 (username uniqueness), UC-007 |
| session | change request (shape) | UC-001, UC-003 (start), UC-004, UC-011, UC-012, UC-014 (end) | every authenticated UC, UC-005 |
| login-lockout | change request (shape) | UC-003 (count, lock, reset), UC-012 (clear) | UC-003 |
| session-end-record | change request (shape) | UC-004, UC-011, UC-012, UC-014 (create) | — |

"A family exists" is not an entity: it means at least one parent exists (Key Decisions). Username uniqueness spans family-member and pending registration, so UC-001, UC-006, UC-008 and UC-010 share one rule.

**Call chains.** No UC calls another at runtime; each is one transaction in the API. Shared behaviour (the username and password rules, the parent-only check, ending a member's sessions) is a common domain rule referenced from each spec, not a UC dependency. Every screen depends only on UCs in this feature.

**Authentication.** Session-based: the API issues an opaque session token at UC-001 and UC-003 and stores the session server-side, so UC-004, UC-011, UC-012 and UC-014 can end sessions. The app keeps the token across restarts, and any refused call with an ended session sends it to the Login screen. The exact token transport is fixed in the UC-003 contract.

**External dependencies.** None: no other feature, spec or external system.

## Spec Authoring Order

1. UC-001 — Found family — creates the first family-member and session, and sets the username, display-name and password rules every later spec reuses.
2. UC-002 — Get family status — the anonymous read that decides between Setup and Login; depends only on family-member.
3. UC-003 — Log in — defines session start, login-lockout and the member/registration status messages.
4. UC-004 — Log out — ends the session UC-003 starts.
5. UC-005 — Get current member — reads the session and member; the template for the authenticated, ended-session response every later UC reuses.
6. UC-006 — Register kid — creates the registration entity and the pending-username reservation.
7. UC-007 — List pending registrations — first parent-only UC; reads registration.
8. UC-008 — Approve registration — turns a registration into a family-member.
9. UC-009 — Reject registration — the other decision; frees the username.
10. UC-010 — Add parent — reuses the uniqueness rule across members and pending registrations.
11. UC-011 — Change own password — first UC that ends sessions selectively.
12. UC-012 — Reset member password — ends all sessions and clears login-lockout.
13. UC-013 — List members — reads every family-member for the Members screen.
14. UC-014 — Deactivate member — ends sessions and carries the last-active-parent guard.
15. UC-015 — Reactivate member — the reverse of UC-014.
16. SCR-001 — Setup — uses UC-001, UC-002.
17. SCR-002 — Login — uses UC-002, UC-003.
18. SCR-003 — Kid registration — uses UC-006.
19. SCR-004 — Home — uses UC-005, UC-007 (badge), UC-004.
20. SCR-005 — Pending registrations — uses UC-007, UC-008, UC-009.
21. SCR-006 — Members — uses UC-013, UC-014, UC-015, UC-012.
22. SCR-007 — Add parent — uses UC-010.
23. SCR-008 — Change password — uses UC-011.

## Risks & Unknowns

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| The session mechanism is still open in the tech stack and every UC depends on it | High | High | Fix it in the UC-003 contract (opaque server-side session token) before authoring UC-004 onward; recorded as a decision in UC-003. |
| The registry change requests for session and login-lockout are not accepted as worded | Medium | Medium | Specs use the requested names; the curator decides at feature approval, and a mapped name is a rename in the specs. |
| Username uniqueness across members and pending registrations races under concurrent sign-ups | Low | Medium | A username-reservation table keyed by the lower-case username gives one unique key over both; the loser gets "username taken". |
| Founding and the last-active-parent guard race without a family row to lock | Low | High | One family-wide advisory lock serializes both. |
| Lockout per submitted username lets someone lock a family member out by guessing their username | Medium | Low | Accepted in the brief (decision on throttling); a parent reset clears the lockout. |
| A kid founds the family on a fresh install | Low | Medium | Accepted risk; the setup confirmation is required. |
| Passwords stored insecurely | Low | High | Spec requires a salted, slow hash and never returning a password; checked in review. |

## Estimated Complexity

- **Estimate:** set at the Phase 3 Estimation Ritual; this hub has no delivered feature to calibrate DF Credits against yet.
- **Basis:** 15 new use cases (9 with business rules, 4 reads, 2 session operations) and 8 new screens in one backend and one app, with no external systems; the first feature also sets up authentication for the whole product.
- **Confidence:** Low
