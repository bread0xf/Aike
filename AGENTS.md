# AI AGENT — Aike Architecture Lock

You are implementing **Aike**, a technology-enabled delivery marketplace.

Treat the repository documentation and locked architecture as authoritative.

## Non-negotiable constraints

* V1 is a modular monolith.
* TypeScript is the primary application language.
* PostgreSQL is the system of record.
* Supabase is initial infrastructure, not the architecture.
* Avoid unnecessary vendor lock-in.
* Keep plausible infrastructure replacement points behind clean boundaries.
* Do not introduce microservices prematurely.
* Do not introduce Kafka, RabbitMQ, NATS, Redis, Valkey, Kubernetes, or specialized databases without a concrete requirement.
* Use intentional domain events rather than generic database-change events.
* Use a transactional outbox where reliable asynchronous processing is required.
* Queues are for durable backend asynchronous work.
* Realtime is for authorized client-facing updates.
* Realtime is not a replacement for durable messaging.
* Authorization is enforced server-side.
* Object-level and field/data-level authorization matter.
* Private tracking is the default.
* GPS telemetry is not a domain event.
* GPS telemetry should be filtered, deduplicated, and downsampled where appropriate.
* Shipment, Delivery, Assignment, Partner, and Vehicle are distinct concepts.
* Matching and Dispatch are distinct concepts.
* Geography is configurable.
* Pricing is configuration-driven and historically explainable.
* Financial history must remain auditable.
* Domain logic must not depend unnecessarily on Supabase-specific APIs.

## Implementation behavior

Before implementing a feature:

1. Identify the relevant domain module.
2. Identify the business rules involved.
3. Identify authorization requirements.
4. Identify state transitions.
5. Identify domain events, if any.
6. Identify asynchronous side effects, if any.
7. Determine whether the feature belongs in the domain, application, or infrastructure layer.
8. Implement the simplest design satisfying the requirements.

Do not introduce infrastructure simply because it may become useful later.

Do not implement future marketplace capabilities before they are required.

Do not silently modify locked architectural decisions.

If a requested implementation conflicts with the locked architecture, identify the conflict and propose the architectural decision that would need to change before implementing it.

## Design principle

> Build the simplest system that works today while preserving the boundaries required to evolve tomorrow.

Optimize for correctness, security, maintainability, and measured scalability—not architectural fashion.
