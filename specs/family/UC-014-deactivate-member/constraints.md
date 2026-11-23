# Constraints — UC-014-deactivate-member

## Business Rules

- Deactivation deletes nothing. Later features decide what a deactivated member's chores, points and redemptions mean.
- The check that at least one active parent remains and the status change happen under the family-wide lock: the PostgreSQL transaction-scoped advisory lock shared with founding (UC-001). Two parents therefore cannot deactivate each other at the same moment.
- The status change, the removal of the member's sessions and the writing of their session end records happen in one transaction.
