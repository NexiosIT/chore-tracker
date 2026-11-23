# Feature Shaping — FT-001: Family Accounts

## Problem Statement

Nobody in the family can use Chore Tracker yet, because the system cannot tell who is acting. Every later theme (chores, points, approvals, reports) needs to know whether a parent or a kid is behind each action, and parents need to control who gets into the family space. This feature delivers roadmap theme T-04: "Every family member has their own login, with parents in control of who joins".

## Proposed Solution

Every family member gets their own username-and-password login. On a fresh install the first sign-up founds the family as its first parent, and that path then closes. Kids register themselves, and their registration stays pending until a parent approves or rejects it from a pending list with a count badge. Parents add further parents directly, reset any member's password, and deactivate or reactivate members without deleting their data. Logins last until logout or until a password change, reset or deactivation ends them, and 5 consecutive failed attempts lock a submitted username for 15 minutes.

## Scope

### In Scope

- **C1 — Found the family as its first parent** (brief Req 1). Setup screen only while no family exists; confirmation "I am a parent setting up this family"; family and first active parent created together and logged in; every broken input rule named. Display-name rule message (1.7, inferred, ratified).
- **C2 — Kid registers themselves** (brief Req 2). Registration link on the login screen once the family exists; a pending registration is created and the kid is told a parent must approve it. Pending registrations reserve their username (2.5, inferred, ratified); registration refused while no family exists (2.7, inferred, ratified); a username from a rejected registration can be reused (2.8).
- **C3 — Parent reviews pending registrations** (brief Req 3). Pending list oldest first with username, display name and requested date; count badge for every active parent; approve creates an active kid, reject keeps the record and frees the username; decisions on non-pending, missing or unauthorized calls refused.
- **C4 — Log in and log out** (brief Req 4). Case-insensitive username; session survives app restarts with no idle timeout; generic message for wrong credentials; specific messages for pending, rejected and deactivated accounts when the password matches; lockout after 5 consecutive failures per submitted username for 15 minutes; a successful login resets the count (4.9, inferred, ratified).
- **C5 — Parent adds another parent** (brief Req 5). Active immediately, no approval step; same username and input rules; kids and anonymous callers refused.
- **C6 — Change or reset a password** (brief Req 6). Self-service change with the current password, keeping the session used and ending all others; parent reset without the old password; reset ends all of that member's sessions and clears their lockout (6.6, inferred, ratified).
- **C7 — Deactivate and reactivate a member** (brief Req 7). Members list of all members with role and status; deactivation deletes nothing and ends all their sessions (7.3, inferred, ratified); reactivation restores login with the existing password; the last active parent cannot be deactivated (7.6, inferred, ratified).

### Out of Scope

- Notifying parents of pending registrations — deferred to theme T-06; parents see the pending list and badge instead.
- Parents registering themselves — an existing parent creates further parent accounts directly (decision 3).
- Email-based or self-service forgotten-password recovery — the app stores no email addresses; a parent resets passwords.
- Session expiry or idle timeout — sessions last until logout or until a password change, reset or deactivation ends them.
- Deleting family members — parents deactivate and reactivate; no data is removed.
- What deactivation means for chores, leaderboard places and redemptions — decided by themes T-01, T-02 and T-03.
- Home view content per role — defined by later features.
- Multiple families — the system serves a single family.

## Key Decisions

| Decision | Options Considered | Choice | Rationale |
|----------|-------------------|--------|-----------|
| Which feature goes first | T-04 accounts first, or a chores theme first | T-04 family accounts | Every other theme needs to know which parent or kid is acting. |
| How the first parent and family come into existence | Admin seeding, setup outside the app, first sign-up founds the family | First sign-up founds the family as an active parent, then that path closes | No admin seeding and no setup outside the app. |
| How a second parent joins | Self-registration with approval, created by an existing parent | An existing parent creates the account, active immediately | Parents are trusted; self-registration as a parent would need its own approval path. |
| Rejected registration's username | Keep reserved, delete the record, keep record and free the username | Kept on record as rejected, username freed | A kid can try again and can be told the registration was rejected. |
| Notify parents of pending registrations | Notify now, pending list with badge | Pending list with a count badge; notifications come with T-06 | Keeps the feature inside the family context and the sprint. |
| Username and password rules | Various lengths and composition rules | Username 3–20 letters, digits, underscore, unique ignoring case; password at least 8 characters, no composition rules | Has to work for kids and still be reasonable for a private family app. |
| How long a login lasts | Idle timeout, fixed expiry, until logout or ended by the system | Until logout or ended by the system, no idle timeout | Phone-first use in short moments makes re-login costly. |
| Forgotten password | Email reset, parent reset | A parent sets a new password | No email addresses are stored, and parents are in control. |
| Removing members | Delete, deactivate and reactivate | Deactivate and reactivate, history kept | Removes access without losing chores and points history. |
| Login message for pending or deactivated members | Generic only, specific when the password matches | Specific message when the password matches, generic for wrong credentials | Clear feedback to the right person without revealing which usernames exist. |
| Throttling failed logins | None, per account, per submitted username | 5 consecutive failures lock the submitted username for 15 minutes | Light protection against guessing at a cost a family can live with. |
| Display name besides username | Username only, separate display name | Display name 1–30 characters, not unique | Leaderboards and lists read better with real names. |
| Self-service password change | None, with current password | Yes, by giving the current password | Basic self-service; removes reset requests to parents. |
| Username reserved while pending | Reserved, first approval wins | Reserved (inferred, ratified) | Prevents two kids from competing for one username. |
| Sessions on deactivation or parent reset | Keep, end all | All of that member's sessions end (inferred, ratified) | Logins never expire, so access has to end at the account change. |
| Deactivating the last active parent | Allow, refuse | Refused (inferred, ratified) | Otherwise nobody could approve registrations or manage the family. |
| BPMN for the kid-registration flow change | Draft a new diagram, describe in the brief | Described only; applied with the flow command after delivery | Two renamed steps need no new diagram. |
| Stopping a kid from founding the family | Admin code, confirmation checkbox | Confirmation "I am a parent setting up this family", remaining risk accepted | One family installs once, so a confirmation is proportionate. |
| What a rejected kid sees at login | Generic message, specific message | "A parent rejected the registration" | Otherwise the kid only ever sees "incorrect". |
| Lockout for unknown usernames | Only existing usernames, every submitted username | Counted per submitted username | Otherwise the lockout reveals which usernames exist. |
| Own sessions after changing own password | Keep all, end all, keep the current one | Keep the session used for the change, end the others | A password change usually means a device should lose access. |
| Refusal when several rules are broken | First broken rule, every broken rule | Every broken rule is named | The person can fix everything in one go. |
| Whether the family needs its own entity | Separate family entity, implied by the first parent | Implied: a family exists once an active or deactivated parent exists | The system serves one family; a separate row adds nothing. |
| Domain entities for logins | No entity, session only, session and login lockout | Request `session` and `login-lockout` in the family context | Sessions are ended by several use cases and lockouts are shared by login and reset, so both are real domain state. |
| UC split for C1 found family | One found-family UC; found-family plus a family-status read | found-family (UC-001) plus get-family-status (UC-002) | Founding is a write by an anonymous actor; the entry-screen choice is a separate anonymous read. |
| UC split for C2 kid registers | One register-kid UC; one generic create-member UC | register-kid (UC-006) | One actor and one goal; registrations have their own rules, unlike members. |
| UC split for C3 review registrations | approve plus reject UCs; one decide-registration UC | list-pending-registrations (UC-007), approve-registration (UC-008), reject-registration (UC-009) | Approve creates a kid, reject frees the username: different outcomes in separate transactions. |
| UC split for C4 log in and out | log-in plus log-out; one manage-session UC; plus a current-member read | log-in (UC-003), log-out (UC-004), get-current-member (UC-005) | Login carries lockout and status rules; the current-member read lets the app find its role after a restart and detect an ended session. |
| UC split for C5 add parent | add-parent UC; generic create-member UC | add-parent (UC-010) | Only parents are ever created directly by another member. |
| UC split for C6 passwords | Two UCs; one set-password UC | change-own-password (UC-011), reset-member-password (UC-012) | Different actors and rules: current password and session kept, versus parent-only, all sessions ended and lockout cleared. |
| UC split for C7 members | Two UCs for status; one set-member-status UC | list-members (UC-013), deactivate-member (UC-014), reactivate-member (UC-015) | Deactivation carries the last-parent guard and ends sessions; reactivation does neither. |
| Screen split | 6, 8 or 9 screens | 8 screens: setup, login, kid registration, home, pending registrations, members (with a reset dialog), add parent, change password | Each form has its own screen; reset is a short action on a chosen member, so it is a dialog on the members screen. |
| Pattern types | Reuse existing keys; mint new ones | All new (the hub has none yet): domain-logic-with-rules, query-api-endpoint, authentication-session; screens form-screen, list-with-actions-screen, app-shell-screen | First feature; keys are kept generic so later features can reuse them. |

| Read use case names | Task names such as check-family-setup; keep get and list | Keep get-family-status, get-current-member, list-pending-registrations, list-members | They are pure reads with no business rule beyond the parent-only check; a task name would only disguise the read. |
| Session token transport | Bearer header, cookie | Opaque token in the response body, sent as a Bearer header; sessions stored server-side | Fits a Flutter app, and the server can end sessions on logout, password change, reset and deactivation. |
| Username storage | Keep as typed, lowercase | Stored and shown as typed, compared ignoring case | Kids see the name they chose. |
| Password maximum | 64, 128, none | 64 characters | Caps hashing cost and fits bcrypt. |
| API path style | Unversioned, versioned | Unversioned resource paths under /api | A single app and a single API ship together. |
| Login attempts with a matching password on a pending, rejected or deactivated account | Count, do not count | Not counted, and the count is not reset | The password was right; the lockout is against guessing. |
| Lockout status code | 429, 401, 403 | 429 with retryAt | Clear for the app. Needs a rate-limit category in STD-002. |
| What the fifth failure returns | Incorrect message, lockout message | Lockout message with retryAt | The person learns at once when to try again. |
| Wrong current password on change own password | Field error, separate 422, counts toward the lockout | Field error on currentPassword; not counted toward the lockout | Shown with the other field errors; the caller already holds a session. |
| Parent resetting their own password | Allow, refuse | Refused; use change password | The brief scopes reset to another member, and change password checks the current one. |
| Members list order | By role then name, by name only | Parents first, then kids, each by display name then username | Parents manage the list and look for kids by name. |
| Deactivating an already deactivated member | 409, idempotent 200 | 409 | Tells the parent that nothing changed. |
| Parent deactivating themselves | Allow while another parent is active, refuse | Allowed while another active parent remains | The last-active-parent guard is enough protection. |
| Ended sessions | Delete the row, keep the row with an end time, delete and keep an audit record | The session row is removed and a session end record keeps the member, start time, end time and reason | The engineer wants an audit of why sessions ended, without live and ended sessions mixed in one table. |
| Registration decision record | Record the decision, record nothing | Each decision records the deciding parent and the decision time; an approval also records the created member | The registration is kept on record anyway, and it now says who decided it. |
| Timestamps | None, created and updated only, plus attribution and device | Members carry created_at and updated_at; sessions carry created_at | Enough for support questions without storing more personal data than needed. |
| Concurrency mechanism | Name it in the specs, require atomicity only | A username-reservation table keyed by the lower-case username, and one family-wide advisory lock for founding and the last-active-parent guard | Username uniqueness spans two tables and there is no family row to lock, so the guarantee needs a named mechanism. |
| Username reservation link | The reservation names its holder; the holders point at the reservation | The reservation names its holder (family member or registration, exactly one); approval re-points it, rejection deletes it | A rejected registration is kept but must hold no reservation; deleting the reservation does that cleanly. |
| Registry entry for the username reservation | Technical table, domain entity | Technical table, no registry entry | It only enforces the uniqueness rule of family members and registrations. |

## Registry Change Requests

Ratified at the feature-approval gate by the domain registry curator. Nothing is written to the domain model until then.

| Name | Context | Definition | Aliases | Projection of | Justification |
|------|---------|------------|---------|---------------|---------------|
| session | family | A family member's logged-in state on one device; it lasts until logout or until a password change, a password reset or a deactivation ends it. | — | — | Several use cases end sessions (logout, password change, reset, deactivation), so it is shared domain state, not a login internal. |
| login-lockout | family | The count of consecutive failed login attempts for one submitted username, and the 15-minute lockout that 5 failures start; it exists for usernames that match no member too. | failed-login-count | — | Shared by log-in (counts and resets) and password reset (clears), and keyed by submitted username rather than by member. |
| session-end-record | family | An audit record of one ended session: which member it belonged to, when it started and ended, and why it ended (logout, password change, password reset or deactivation). | — | — | The engineer chose to keep an audit of ended sessions; four use cases write it. |

## Context & Trade-offs

This is the first feature on a greenfield product, so it also establishes the API's authentication approach, which the tech stack leaves open ("token or session mechanism not decided"). The brief fixes the behaviour (sessions without idle timeout that the server can end), which rules out stateless tokens that cannot be revoked; the concrete mechanism is chosen during specification and generation.

The family has a single Android app (Flutter) and a single Spring Boot API. All behaviour lives in the family context owned by the API service.

The brief also changes the kid-registration flow: "Notify parents of pending registration" becomes "Show registration in parents' pending list", and "Activate kid's login" is renamed "Create kid's account". That is a flow edit, not a spec, and it is applied with the flow command once this feature is delivered.

Accepted risk: a kid could found the family on a fresh install by ticking the parent confirmation (decision on setup). One family installs the app once, so this is accepted.

## Affected Specs

- UC-001 — Found family (create) · pattern domain-logic-with-rules (new)
- UC-002 — Get family status (create) · pattern query-api-endpoint (new)
- UC-003 — Log in (create) · pattern authentication-session (new)
- UC-004 — Log out (create) · pattern authentication-session (new)
- UC-005 — Get current member (create) · pattern query-api-endpoint (new)
- UC-006 — Register kid (create) · pattern domain-logic-with-rules (new)
- UC-007 — List pending registrations (create) · pattern query-api-endpoint (new)
- UC-008 — Approve registration (create) · pattern domain-logic-with-rules (new)
- UC-009 — Reject registration (create) · pattern domain-logic-with-rules (new)
- UC-010 — Add parent (create) · pattern domain-logic-with-rules (new)
- UC-011 — Change own password (create) · pattern domain-logic-with-rules (new)
- UC-012 — Reset member password (create) · pattern domain-logic-with-rules (new)
- UC-013 — List members (create) · pattern query-api-endpoint (new)
- UC-014 — Deactivate member (create) · pattern domain-logic-with-rules (new)
- UC-015 — Reactivate member (create) · pattern domain-logic-with-rules (new)

## Affected Screens

- SCR-001 — Setup (create) · pattern form-screen (new) · C1
- SCR-002 — Login (create) · pattern form-screen (new) · C4
- SCR-003 — Kid registration (create) · pattern form-screen (new) · C2
- SCR-004 — Home (create) · pattern app-shell-screen (new) · C3, C4, C6
- SCR-005 — Pending registrations (create) · pattern list-with-actions-screen (new) · C3
- SCR-006 — Members (create) · pattern list-with-actions-screen (new) · C6, C7
- SCR-007 — Add parent (create) · pattern form-screen (new) · C5
- SCR-008 — Change password (create) · pattern form-screen (new) · C6
