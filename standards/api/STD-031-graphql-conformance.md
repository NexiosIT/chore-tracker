---
id: STD-031
name: "GraphQL Code-to-Contract Conformance"
version: "1.0.0"
category: api
status: active
last_reviewed: "2026-07-03"
---

# GraphQL Code-to-Contract Conformance

> Producer services exposing a GraphQL API MUST prove, at test time, that every contract root field is served correctly and that errors follow a stable, machine-readable taxonomy — using GraphQL's **native `{data, errors}` envelope**. The REST conformance oracle (STD-024's envelope unwrap + camelCase→snake_case projection) does **not** apply.

## Motivation

For a UC with `contract.graphql`, the SDL is the source of truth. Unlike REST, much of the structural proof is free in schema-first stacks: Spring for GraphQL (and peers) fail at startup when a resolver is missing or mistyped against the SDL, and a spec-compliant server cannot emit a response that violates the schema's shape. What remains — and what this standard mandates — is the **semantic** layer: is every contract operation actually resolved and tested, do errors carry a stable code, and is nullability authored deliberately rather than inherited by accident.

This is the GraphQL analog of STD-024, following the same split established for gRPC: structural conformance rides the toolchain; the framework mandates the universal semantic checks; the stack realization is authored per service stack.

## Where the boundary is

The wire format is GraphQL's own: a top-level `{data, errors}` response, **camelCase field names as written in the SDL**. There is no `{data, error, meta}` envelope (STD-001) and no snake_case wire (STD-010) — do not wrap GraphQL responses in the REST envelope, and do not project casing in test oracles. STD-024 Rules 2–4 do not apply to GraphQL operations.

## Rules

1. **Every contract root field MUST have a conformance test.** For each `Query` and `Mutation` field in `contract.graphql`, an integration test MUST execute the operation against the running schema and assert the business outcome. Coverage is keyed by the contract operation ids `query {field}` / `mutation {field}` (`ie contracts coverage` enumerates them). A root field with no test is a coverage gap that blocks feature completion.

2. **Errors carry a stable machine code.** Every error path defined in the UC's scenarios MUST surface as a GraphQL error with a stable, documented code at `errors[].extensions.code` (the GraphQL analog of STD-002's error taxonomy). Raw exception class names or messages MUST NOT leak into `errors[].message`. A test per distinct error scenario asserts the code.

3. **Partial-data semantics are intentional.** For each top-level error path, a test MUST assert the intended `data` outcome — `data: null` (operation failed atomically) or partial data with the failed field `null` (partial success is meaningful). Whichever the UC intends, prove it; do not leave it to nullability accident.

4. **Nullability is authored, not inherited.** Contract-required inputs are non-null (`!`) in the SDL — the server then rejects violations on the wire for free (the inverse of proto3's silent zero-defaults). Response fields are non-null only where the resolver can always supply them: a non-null field whose resolver errors bubbles the null up to the nearest nullable parent and can wipe sibling data. Review nullability at spec time; conformance tests for error paths (Rule 3) catch bubbling surprises.

5. **No subscriptions at P1.** Async pushes are modeled in `contract.asyncapi.yml` and validated per STD-024 Rule 5 / STD-021/022. `ie spec validate` warns on `Subscription` root fields.

6. **Generate the helper once; reuse it.** As STD-024 Rule 6: one small conformance helper per service (execute an operation document, assert `errors[].extensions.code` / data shape), generated per the stack's test-pattern standard and reused across every root field. IE ships no helper code.

7. **Read the contract from the materialized snapshot.** Operation documents and expected shapes derive from `.flowforge/contracts/{UC}/contract.graphql` (materialized at handover) so tests run standalone; snapshot↔hub drift is `ie manifest verify`'s job.

## Realization (per stack)

This standard is stack-agnostic and framework-shipped; **the per-stack realization is authored by the adopting organisation** (mirrors the gRPC split). Illustrative for Spring Boot + Spring for GraphQL:

- Schema-first: ship `contract.graphql`'s types in the service's schema files; boot fails on resolver/schema mismatch (structural conformance).
- `@GraphQlTest` / `GraphQlTester` integration tests per root field (Rule 1), asserting business outcome + `errors[].extensions.code` via `.errors().satisfy(...)` (Rules 2–3).
- A `DataFetcherExceptionResolver` maps domain exceptions to the error taxonomy — the single place Rule 2 is implemented.

## Applicability

- **Service types:** backend, gateway
- **Domains:** all
- **Checkpoints:** CP1 (SDL types → DTO/domain awareness), CP3 (the per-root-field conformance tests). Bound via the `api` category (see `/ie/generate` standards-to-checkpoint binding).
- **Exceptions:** introspection and health fields (no contract); event-only UCs have no GraphQL surface.

## References

- STD-024: Code-to-Contract Conformance (the REST sibling; its envelope/casing rules do NOT apply here)
- STD-002: Error Handling Strategy (the taxonomy `errors[].extensions.code` mirrors)
- STD-021 / STD-022: Event Publisher / Consumer (where async pushes live instead of subscriptions)
