# Constraints — UC-011-change-own-password

## Security

- Passwords are compared and stored only as salted, slow one-way hashes. Neither password is logged or returned.
- A wrong current password does not count toward the login lockout of UC-003, because the caller already holds a session.

## Business Rules

- The password replacement and the ending of the other sessions happen in one transaction.
- The new password may equal the current one; there is no history rule.
