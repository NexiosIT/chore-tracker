# Constraints — UC-012-reset-member-password

## Security

- The new password is stored only as a salted, slow one-way hash and is never logged or returned.
- The password replacement, the ending of the target's sessions and the clearing of the lockout happen in one transaction.

## Business Rules

- A parent cannot reset their own password here. They use change password (UC-011), which asks for the current one.
- The lockout cleared is the one keyed by the target member's username in lower case.
- A deactivated member's password can be reset. The member stays deactivated.
