# Spring Boot Standards

Stack-coupled standards for services with `framework: spring-boot` in their `services/*.yml` entry. The manifest assembler injects this directory's content only when the touched service declares `framework: spring-boot`.

## What Goes Here

Conventions that depend on Spring Boot's idioms — `@Service` + `@Transactional` boundaries, `@RestController` patterns, JPA entity/repository design, Mockito + `@SpringBootTest` test scaffolding.

Stack-agnostic concerns (response envelopes, error categories, NFR budgets, naming) live in `standards/api/`, `standards/nfr/`, `standards/general/`.

## Naming Convention

`STD-{NNN}-{kebab-name}.md` for promoted standards, `{descriptive-name}.md` for drafts.
