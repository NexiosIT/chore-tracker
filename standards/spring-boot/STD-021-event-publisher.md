---
id: STD-021
name: "Event Publisher (Spring Kafka)"
version: "1.0.0"
category: spring-boot
status: active
last_reviewed: "2026-05-29"
discovery_layer: "manual"
---

# Event Publisher (Spring Kafka)

> Backend services that declare a `send` operation in a `contract.asyncapi.yml` MUST publish that event through a broker-agnostic publisher port with a Spring Kafka adapter, a CloudEvents envelope, a deterministic partition key, and an at-least-once delivery guarantee via the transactional outbox pattern.

## Motivation

The async event contract (`contract.asyncapi.yml`) is broker-agnostic by design: it declares *what* is published (channel address + payload schema + `action: send`) but not *how*. This standard fills the "how" for the Spring stack without leaking broker choice into the spec. Without it, generated publishers couple domain logic to Kafka, drop events on transaction rollback, and emit payloads that drift from the contract — exactly the producer↔consumer drift Iron Dome guards against. Keeping the broker behind a port also lets a service retarget brokers without touching the spec.

## When this applies

Apply only to UCs whose `contract.asyncapi.yml` declares an `operation` with `action: send`. A UC with no async contract, or only `receive` operations, does not exercise this standard.

## Rules

1. **Publish through a port, not a `KafkaTemplate` directly.** Define a domain-facing `EventPublisher` interface in the application layer; the Spring Kafka implementation lives in the infrastructure layer. Domain/service code depends on the port only.

2. **The payload type is generated from the contract.** The message payload class MUST be generated from the `components.schemas` entry referenced by the channel's message (CP1, Domain Model & Contracts). Field names, types, and required-ness MUST match the schema exactly — this is what keeps the producer aligned with consumers.

3. **The channel `address` is the topic.** Map the AsyncAPI channel `address` (e.g. `orders.placed`) to the broker topic. Never hardcode a topic string elsewhere; resolve it from a single configuration source keyed by the channel address.

4. **Wrap every event in a CloudEvents envelope.** Set `id` (UUID), `source` (the owning service name), `type` (the channel address or message name), `time` (UTC), and `datacontenttype: application/json`. The domain payload is the CloudEvents `data`.

5. **Set a deterministic partition key.** Use the aggregate identifier (e.g. `order_id`) as the Kafka message key so all events for one aggregate preserve order on a single partition. Document the key field in `constraints.md` when ordering matters.

6. **Guarantee at-least-once delivery with a transactional outbox.** Within the same database transaction that commits the domain state change, write the event to an `outbox` table. A relay publishes outbox rows to Kafka and marks them sent. Never publish directly inside the domain transaction (a broker failure must not roll back domain state, and a domain rollback must not emit a phantom event).

7. **Do not invent delivery semantics in code.** At-least-once is the default. If a UC needs ordering or exactly-once, it MUST be stated in `constraints.md`; the publisher implements what the constraint specifies, nothing more.

## Checkpoint binding

- **CP1 (Domain Model & Contracts):** generate the payload class(es) from `components.schemas`; define the `EventPublisher` port.
- **CP3 (Interface Layer):** generate the Spring Kafka adapter, the outbox write, and the relay; wire publish calls at the point the domain state change commits.

## Verification

- A published event deserializes back into the contract payload type with no missing required fields.
- Rolling back the domain transaction leaves no event in the outbox / on the topic.
- All events for one aggregate id land on the same partition (key assertion in an integration test).
