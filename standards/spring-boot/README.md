# Spring Boot Standards

Stack-coupled standards for services with `framework: spring-boot` in their `services/*.yml` entry. The manifest assembler injects this directory's content only when the touched service declares `framework: spring-boot`.

## Language Note (Chore Tracker)

`chore-tracker-api` is written in **Kotlin**, not Java. The code examples in these standards are Java; apply the same rules with Kotlin idioms (data classes, null-safety instead of `Optional`, `val` by default). Rewriting the examples in Kotlin can be done later through `/ie/shape-standard`.

## What Goes Here

Conventions that depend on Spring Boot's idioms — `@Service` + `@Transactional` boundaries, `@RestController` patterns, JPA entity/repository design, Mockito + `@SpringBootTest` test scaffolding.

Stack-agnostic concerns (response envelopes, error categories, NFR budgets, naming) live in `standards/api/`, `standards/nfr/`, `standards/general/`.

## Naming Convention

`STD-{NNN}-{kebab-name}.md` for promoted standards, `{descriptive-name}.md` for drafts.
