# Tech Stack

## Languages

| Tier | Language | Version |
|------|----------|---------|
| Backend | Kotlin | 2.x on JVM 21 (LTS) |
| Mobile (Android) | Dart | 3.x (bundled with Flutter stable) |
| Database migrations | SQL | PostgreSQL dialect |

## Frameworks

| Tier | Framework | Version |
|------|-----------|---------|
| Backend | Spring Boot | 4.0.x |
| Mobile (Android) | Flutter | current stable channel |

The family uses only the Android app; there is no web client at this stage. Flutter was chosen over native Kotlin/Jetpack Compose so that an iPhone app can be added later from the same codebase.

## Infrastructure

- **Database:** PostgreSQL, accessed from the backend through Spring Data JPA / Hibernate. Schema changes go through Flyway migrations.
- **API:** the Android app talks to the Spring Boot backend over a REST/JSON API.
- **Authentication:** parents and kids register with a username and password. A kid's registration is only activated after a parent approves it. The token or session mechanism is not decided yet.
- **Notifications:** kids and parents receive in-app notifications (chore assigned, chore due soon, chore needs approval, chore completed). The delivery mechanism is not decided yet.
- **Deployment:** TBD. The hosting target for the backend and database has not been chosen.
- **CI/CD:** TBD. The code repositories will be on GitHub.
- **App distribution:** TBD.

## Constraints

- **Android support:** the app must run on Android 10 (API level 29) and newer.
- **Family privacy:** nobody outside the family can see any data. Within the family, parents see everything, and kids see their own chores, points and reports as well as the family leaderboard.
- **No private data:** the system stores no personal data beyond what the app needs to work (usernames, chores, points and rewards). No real names, contact details or payment details are stored.
- **Single family:** the system serves one family. Multi-family support is out of scope for now.
- **Future iOS:** choices on the mobile side must not rule out an iPhone app built from the same Flutter codebase.
