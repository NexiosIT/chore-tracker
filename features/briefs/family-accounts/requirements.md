---
title: "Family accounts and kid registration"
description: "Parents and kids each get their own username-and-password login; the first sign-up founds the family as a parent, kids self-register pending parent approval, and parents add parents, reset passwords and deactivate members."
domains: [family]
flows: [kid-registration]
---

# Requirements Document

## Introduction

This feature gives every member of the family their own login, with parents in control of who joins. It delivers roadmap theme **T-04 — "Every family member has their own login, with parents in control of who joins"** (the roadmap has themes, not epics, so no `epic` is set). On a fresh install the first person to sign up founds the family and becomes its first parent. Kids then register themselves, and their registration stays pending until a parent approves it. Parents can add further parents directly, reset any member's password, and deactivate or reactivate members.

The roadmap says parents and kids each register. This brief deliberately has an existing parent create further parent accounts instead (decision 3). Every other theme (chores, points, approvals, reports) depends on knowing who is acting, so this feature comes first. Notifications about pending registrations are left to theme T-06. In this feature, parents see pending registrations in a list in the app.

Later themes must decide what deactivation means for their own data: a deactivated kid's open and group-assigned chores (T-01), their leaderboard place and outstanding redemptions (T-02, T-03). This feature only guarantees that deactivation deletes nothing.

Requirements tagged **[inferred]** were not stated explicitly by the roadmap or the engineer. They were proposed during the discussion and accepted as defaults, and they must be ratified during shaping.

## Glossary

- **Family**: The single household the system serves. It exists once its first parent has signed up.
- **Family_Member**: A parent or kid with their own Username and Password. It has a role (*parent* or *kid*) and a status (*active* or *deactivated*).
- **Parent**: A Family_Member with the parent role. Parents approve registrations and manage Family_Members.
- **Kid**: A Family_Member with the kid role.
- **Registration**: A kid's sign-up request, with status *pending*, *approved* or *rejected*. An approved Registration becomes an active Kid. A rejected Registration is kept on record, but its Username is free for reuse.
- **Username**: The Family_Member's login name. It is 3–20 characters of letters, digits or underscore, and unique across the family regardless of case.
- **Display_Name**: The name shown to the family, for example on the leaderboard. It is 1–30 characters and does not need to be unique.
- **Password**: A secret of at least 8 characters. There are no composition rules.
- **Session**: A logged-in state on one device. It lasts until the Family_Member logs out, or until it is ended by a password change, a password reset or a deactivation, as the requirements below state.
- **Lockout**: A 15-minute period during which login for a submitted Username is refused after 5 consecutive failed attempts with that Username.
- **Account_Service**: The part of the system that owns Family_Members, Registrations, Sessions and Passwords.
- **Setup_Screen**: The app screen for founding the family. It is offered only while no family exists.
- **Registration_Screen**: The app screen where a kid requests an account.
- **Login_Screen**: The app screen where a Family_Member logs in. It is the entry screen once the family exists.
- **Home_View**: The first screen after login. Its content per role is defined by later features.
- **Pending_Registrations_List**: The parent view listing pending Registrations, with a count badge.
- **Members_List**: The parent view of all Family_Members with their role and status.

---

## Requirements

### Requirement 1: Found the family as its first parent

**User Story:** As the first parent on a fresh install, I want to create my account and the family in one step, so that I can start inviting the rest of the family.

#### Acceptance Criteria

1. WHILE no family exists, THE app SHALL offer the Setup_Screen as its entry screen and SHALL NOT offer the Registration_Screen.
2. WHILE a family exists, THE app SHALL NOT offer the Setup_Screen.
3. WHEN a person submits the Setup_Screen with a valid Username, Display_Name and Password and confirms "I am a parent setting up this family", THE Account_Service SHALL create the family and an active Parent with those details.
4. WHEN the family and first Parent are created, THE Account_Service SHALL start a Session for that Parent.
5. IF the "I am a parent setting up this family" confirmation is not given, THEN THE Account_Service SHALL refuse the setup.
6. IF the Username, Display_Name or Password breaks one or more of its rules, THEN THE Account_Service SHALL refuse the setup and name every broken rule.
7. IF the Display_Name is empty or longer than 30 characters, THEN THE Account_Service SHALL refuse the setup with the message that the display name needs 1 to 30 characters. **[inferred]**
8. IF a family already exists when setup is submitted, THEN THE Account_Service SHALL refuse it and create no account.

### Requirement 2: Kid registers themselves

**User Story:** As a kid, I want to create my own account with a username and password, so that I can start using the app once a parent lets me in.

#### Acceptance Criteria

1. WHILE a family exists and nobody is logged in, THE Login_Screen SHALL offer a link to the Registration_Screen.
2. WHEN a kid submits the Registration_Screen with a valid Username, Display_Name and Password, THE Account_Service SHALL create a Registration with status *pending*.
3. WHEN the Registration is created, THE app SHALL tell the kid that a parent must approve the account before they can log in.
4. WHEN the Registration is created, THE Pending_Registrations_List SHALL show it to every active Parent.
5. IF the Username is already used by a Family_Member or a pending Registration, ignoring case, THEN THE Account_Service SHALL refuse the registration with the message that the username is taken. **[inferred: pending registrations reserve their username]**
6. IF the Username, Display_Name or Password breaks one or more of its rules, THEN THE Account_Service SHALL refuse the registration and name every broken rule.
7. IF no family exists yet, THEN THE Account_Service SHALL refuse a kid registration. **[inferred]**
8. WHEN a kid registers with a Username whose earlier Registration was rejected, THE Account_Service SHALL accept it as a new pending Registration.

### Requirement 3: Parent reviews pending registrations

**User Story:** As a parent, I want to approve or reject kids' registrations, so that only my own kids can join the family.

#### Acceptance Criteria

1. WHEN a Parent opens the Pending_Registrations_List, THE app SHALL show each pending Registration with its Username, Display_Name and requested date.
2. THE Pending_Registrations_List SHALL order pending Registrations oldest first.
3. WHILE at least one Registration is pending, THE app SHALL show the number of pending Registrations as a badge for every active Parent.
4. WHEN a Parent approves a pending Registration, THE Account_Service SHALL create an active Kid with its Username, Display_Name and Password, and mark the Registration *approved*.
5. WHEN a Parent rejects a pending Registration, THE Account_Service SHALL mark it *rejected* and free its Username for a new Registration.
6. IF the Registration is no longer pending, THEN THE Account_Service SHALL refuse the decision and tell the Parent whether it was already approved or rejected.
7. IF the caller is a Kid or is not logged in, THEN THE Account_Service SHALL refuse to approve or reject a Registration.
8. IF the Registration does not exist, THEN THE Account_Service SHALL refuse the decision with a not-found message.

### Requirement 4: Log in and log out

**User Story:** As a family member, I want to log in with my username and password and stay logged in on my phone, so that I don't have to type them every time.

#### Acceptance Criteria

1. WHEN an active Family_Member submits a matching Username (in any case) and Password, THE Account_Service SHALL start a Session and THE app SHALL open the Home_View.
2. WHILE a Session exists, THE app SHALL keep the Family_Member logged in across app restarts, with no idle timeout.
3. WHEN a Family_Member logs out, THE Account_Service SHALL end that Session and THE app SHALL show the Login_Screen.
4. IF the Username is unknown or the Password does not match, THEN THE Account_Service SHALL refuse the login with the message "Username or password incorrect".
5. IF the Username and Password match a pending Registration, THEN THE Account_Service SHALL refuse the login with the message that a parent has not approved the account yet.
6. IF the Username and Password match only a rejected Registration, THEN THE Account_Service SHALL refuse the login with the message that a parent rejected the registration.
7. IF the Username and Password match a deactivated Family_Member, THEN THE Account_Service SHALL refuse the login with the message that the account is deactivated.
8. IF 5 consecutive login attempts with the same submitted Username fail, whether or not that Username exists, THEN THE Account_Service SHALL refuse every login with it for 15 minutes and say when to try again.
9. WHEN a login succeeds, THE Account_Service SHALL reset that Username's failed-attempt count to zero. **[inferred]**

### Requirement 5: Parent adds another parent

**User Story:** As a parent, I want to create an account for my partner, so that they can manage chores and approvals too.

#### Acceptance Criteria

1. WHEN a Parent submits a valid Username, Display_Name and initial Password for a new parent, THE Account_Service SHALL create an active Parent immediately, with no approval step.
2. WHEN the new Parent is created, THE Members_List SHALL show them as an active parent.
3. WHEN the new Parent logs in with the initial Password, THE Account_Service SHALL start a Session for them.
4. IF the Username is already used by a Family_Member or a pending Registration, ignoring case, THEN THE Account_Service SHALL refuse with the message that the username is taken.
5. IF the Username, Display_Name or Password breaks one or more of its rules, THEN THE Account_Service SHALL refuse and name every broken rule.
6. IF the caller is a Kid or is not logged in, THEN THE Account_Service SHALL refuse to create a parent account.

### Requirement 6: Change or reset a password

**User Story:** As a family member, I want to change my own password, and as a parent I want to reset anyone's forgotten password, so that nobody is locked out of the family for good.

#### Acceptance Criteria

1. WHEN a logged-in Family_Member submits their current Password and a valid new Password, THE Account_Service SHALL replace the Password.
2. WHEN a Family_Member changes their own Password, THE Account_Service SHALL keep the Session used for the change and end all their other Sessions.
3. IF the current Password given does not match, THEN THE Account_Service SHALL refuse the change.
4. IF the new Password is shorter than 8 characters, THEN THE Account_Service SHALL refuse it with the message that the password needs at least 8 characters.
5. WHEN a Parent sets a new Password for another Family_Member, THE Account_Service SHALL replace it without asking for the old one.
6. WHEN a Parent resets another Family_Member's Password, THE Account_Service SHALL end all of that Family_Member's Sessions and clear any Lockout on their Username. **[inferred]**
7. IF the caller is a Kid or is not logged in, THEN THE Account_Service SHALL refuse to reset another Family_Member's Password.
8. IF the Family_Member to reset does not exist, THEN THE Account_Service SHALL refuse with a not-found message.

### Requirement 7: Deactivate and reactivate a member

**User Story:** As a parent, I want to deactivate a member's account and reactivate it later, so that I control who can use the family space without losing anyone's history.

#### Acceptance Criteria

1. WHEN a Parent opens the Members_List, THE app SHALL show every Family_Member with Display_Name, Username, role and status.
2. WHEN a Parent deactivates an active Family_Member, THE Account_Service SHALL set their status to *deactivated*, delete none of their data, and keep them in the Members_List.
3. WHEN a Family_Member is deactivated, THE Account_Service SHALL end all of their Sessions, and their next action on any device SHALL show the Login_Screen. **[inferred]**
4. WHEN a Parent reactivates a deactivated Family_Member, THE Account_Service SHALL set their status to *active*.
5. WHEN a reactivated Family_Member logs in with their existing Password, THE Account_Service SHALL start a Session.
6. IF the deactivation would leave the family with no active Parent, THEN THE Account_Service SHALL refuse it with the message that the family needs at least one active parent. **[inferred]**
7. IF the caller is a Kid or is not logged in, THEN THE Account_Service SHALL refuse to deactivate or reactivate a Family_Member.
8. IF the Family_Member does not exist, THEN THE Account_Service SHALL refuse with a not-found message.

---

## Flow changes

### kid-registration

- **Changed — "Notify parents of pending registration"** (system, `notifications`): until theme T-06 delivers notifications, this step becomes **"Show registration in parents' pending list"** (system, `family`). The step before is "Create account with username and password" (Kid). The step after is "Review registration" (Parent). When T-06 is briefed, it restores a notification step here.
- **Changed — "Activate kid's login"** (system, `family`): this step is renamed **"Create kid's account"**, because approval creates the active Kid from the Registration (Requirement 3.4). The step before is the "Approve registration?" gateway (yes). The step after is "Kid can log in".
- **Unchanged:** "Create account with username and password", "Review registration", the "Approve registration?" gateway, and the end events "Kid can log in" and "Registration rejected". On the rejected path the Registration is kept on record and its Username is freed (Requirement 3.5), which changes no step.

The family-founding setup (Requirement 1), parent-created parent accounts (Requirement 5), password changes (Requirement 6) and deactivation (Requirement 7) are single-actor actions. They add no step to any flow.
