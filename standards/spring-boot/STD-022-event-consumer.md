---
id: STD-022
name: "Event Consumer (Spring Kafka)"
version: "1.0.0"
category: spring-boot
status: deprecated
last_reviewed: "2026-05-29"
discovery_layer: "manual"
---

# Event Consumer (Spring Kafka)

> Backend services that declare a `receive` operation in a `contract.asyncapi.yml` MUST consume that event with a `@KafkaListener`, validate the payload against the contract schema, handle it idempotently, and route poison messages to a dead-letter topic after a bounded retry.

## Motivation

A consumer that trusts the wire blindly is the failure mode Iron Dome's producer↔consumer drift check exists to prevent — but runtime defence is still required for malformed or replayed messages. This standard makes generated consumers idempotent (brokers deliver at-least-once, so duplicates are normal), schema-checked (reject payloads that violate the contract), and resilient (retry then dead-letter rather than block the partition).

## When this applies

Apply only to UCs whose `contract.asyncapi.yml` declares an `operation` with `action: receive`. A UC with no async contract, or only `send` operations, does not exercise this standard.

## Rules

1. **Bind the listener to the channel `address`.** The `@KafkaListener` topic MUST resolve from the AsyncAPI channel `address`, from the same single configuration source the publisher uses. The consumer group id is the consuming service name.

2. **The payload type is generated from the contract.** Deserialize into the payload class generated from the `components.schemas` entry referenced by the channel's message (CP1). The consumer's expected shape is the contract's shape — do not hand-edit it.

3. **Validate before handling.** Reject (do not process) a message whose payload is missing a required field or has a wrong-typed field. A validation failure is a poison message → route to the dead-letter topic; it is never silently dropped.

4. **Handle idempotently.** At-least-once delivery means the same event may arrive more than once. Use the CloudEvents `id` (or an aggregate-scoped dedupe key) to make handling idempotent — a processed-events table or an upsert keyed on the event id. Re-processing a duplicate MUST be a no-op.

5. **Retry then dead-letter.** Transient failures (downstream timeout) retry with backoff up to a bounded count; after that, publish to a `<topic>.DLT` dead-letter topic with the failure reason. Never block the partition indefinitely on one message.

6. **Preserve ordering by partition.** Process messages for one aggregate in partition order; do not parallelize within a partition. If the UC tolerates out-of-order handling, state that in `constraints.md`.

7. **Do not implement business rules the spec did not state.** The consumer wires the event to a domain service; the domain rules live in the service layer (STD-006), driven by the Gherkin scenarios — not in the listener.

## Checkpoint binding

- **CP1 (Domain Model & Contracts):** generate the payload class from `components.schemas`; define the idempotency/dedupe store.
- **CP3 (Interface Layer):** generate the `@KafkaListener`, payload validation, retry/dead-letter config, and the call into the domain service.

## Verification

- Delivering the same event twice produces a single state change (idempotency integration test).
- A payload missing a required field lands on the dead-letter topic, not in domain state.
- A transient downstream failure retries and recovers without dropping the message.
