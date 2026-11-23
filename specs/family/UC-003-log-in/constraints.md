# Constraints — UC-003-log-in

## Security

- Passwords are compared against the stored salted hash only. A password is never logged or returned.
- An unknown username and a wrong password take comparable time to answer, so the response time does not reveal which usernames exist.
- The session token is an opaque random value of at least 128 bits. The server stores only a hash of it.
- The 429 status for a lockout is not yet a category in STD-002 (Error Handling Strategy). This use case depends on an amendment to STD-002 that adds a rate-limit category mapped to 429.

## Business Rules

- The failed-attempt count is keyed by the submitted username in lower case, whether or not that username exists.
- Only an unknown username or a wrong password counts as a failure. A missing field, a lockout refusal, or a matching password for a pending, rejected or deactivated account does not change the count.
- When the fifth consecutive failure happens, the username is locked for 15 minutes from that moment, and that fifth attempt already gets the lockout answer. When the lockout ends, the count restarts at zero.
- A member can hold several sessions, one per device. Sessions have no idle timeout.
- When several rejected registrations share the username, any of them with a matching password gives the rejected answer.
- A session stores its member, the hash of its token and its created_at time. When a session ends, its row is removed and a session end record is written (see UC-004, UC-011, UC-012 and UC-014).
