# Decision Catalog: Family accounts and kid registration

| # | Question | Decision | Rationale |
|---|----------|----------|-----------|
| 1 | Which feature should be briefed first? | T-04 family accounts | Every other theme needs to know which parent or kid is acting. |
| 2 | How does the first parent and the family come into existence? | The first sign-up on a fresh install founds the family as an active parent, and that path then closes | This needs no admin seeding and no setup outside the app. |
| 3 | How does a second parent join? | An existing parent creates the account directly, and it is active immediately | Parents are trusted, and self-registration as a parent would need its own approval path. |
| 4 | What happens to a rejected registration's username? | The registration is kept on record as rejected, and its username is freed for reuse | A kid can try again, and the kid can be told that the registration was rejected. |
| 5 | Notify parents of pending registrations in this feature? | No. Parents see a pending list with a count badge, and notifications come with T-06 | This keeps the feature inside the `family` context and the sprint. |
| 6 | Username and password rules? | Username 3–20 chars of letters, digits and underscore, unique regardless of case. Password at least 8 chars, with no composition rules | The rules have to work for kids and still be reasonable for a private family app. |
| 7 | How long does a login last? | Until logout or until the system ends it, with no idle timeout | Phone-first use in short moments makes re-login friction costly. |
| 8 | How is a forgotten password handled? | A parent sets a new one, and no email is involved | The app stores no email addresses, and parents are in control. |
| 9 | Can parents remove members? | Parents can deactivate and reactivate members, and history is kept | This removes access without losing chores and points history. |
| 10 | What does a pending or deactivated member see at login? | A specific message when the password matches. Wrong credentials get a generic message | This gives clear feedback to the right person without revealing which usernames exist. |
| 11 | Throttle failed logins? | 5 consecutive failures lock the username for 15 minutes | Light protection against guessing, at a cost a family can live with. |
| 12 | Display name besides username? | Yes, 1–30 chars and not unique | Leaderboards and lists read better with real names. |
| 13 | Can members change their own password? | Yes, by giving their current password | This is basic self-service, and it removes reset requests to parents. |
| 14 | Does a username stay reserved while its registration is pending? | Yes [inferred] | This prevents two kids from competing for the same username. |
| 15 | What happens to sessions on deactivation or parent reset? | All of that member's sessions end [inferred] | Logins never expire, so access has to end at the account change. |
| 16 | Can the last active parent be deactivated? | No [inferred] | Otherwise nobody could approve registrations or manage the family. |
| 17 | Draft a BPMN for the kid-registration flow change? | No, the body describes it only | The change is two renamed steps and needs no new diagram. |
| 18 | How do we stop a kid from founding the family on a fresh install? | Setup asks the person to confirm "I am a parent setting up this family", and the remaining risk is accepted | One family installs the app once, so a confirmation is proportionate. |
| 19 | What does a rejected kid see at login? | The message that a parent rejected the registration | Without it, the kid only ever sees "incorrect" and cannot tell what happened. |
| 20 | Does lockout apply to usernames that don't exist? | Yes. Failed attempts are counted per submitted username | Otherwise the lockout would reveal which usernames exist. |
| 21 | What happens to your other sessions when you change your own password? | They end, and the session used for the change stays | A password change usually means a device should lose access. |
| 22 | When input breaks several rules, what does the refusal name? | Every broken rule | The person can fix everything in one go. |
