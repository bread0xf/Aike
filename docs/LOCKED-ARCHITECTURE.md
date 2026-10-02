# Aike — Locked Architecture

> **Status: LOCKED**
>
> These decisions define the architectural constraints for Aike V1.
> They may only be changed through an explicit architectural decision.

---

## 1. Product Architecture

Aike V1 is a **modular monolith**.

Do not introduce microservices unless there is a demonstrated requirement for independently scaling, deploying, or operating a specific module.

The application must maintain clear module boundaries so individual modules can be extracted later.

---

## 2. Infrastructure Philosophy

Use the minimum infrastructure required to make the current system:

* Correct
* Secure
* Observable
* Maintainable

Managed infrastructure is preferred during early development and initial production deployment.

Supabase may be used as the initial infrastructure platform.

**Supabase is not the architecture.**

The application must not unnecessarily depend on Supabase-specific semantics.

---

## 3. Database

PostgreSQL is the primary transactional system of record.

Prefer standard PostgreSQL capabilities.

Database design should remain portable across:

* Supabase PostgreSQL
* Managed PostgreSQL
* Self-hosted PostgreSQL

Do not introduce a second primary transactional database without a demonstrated requirement.

---

## 4. Infrastructure Boundaries

Infrastructure capabilities that are reasonably likely to be replaced should be accessed through explicit boundaries.

Potential boundaries include:

* `Repository`
* `MessageBus`
* `ObjectStorage`
* `IdentityProvider`
* `RealtimePublisher`
* `TelemetryStore`
* `Cache`
* `Scheduler`

Do not create abstractions merely for the sake of abstraction.

Abstract meaningful infrastructure capabilities, not every library call.

---

## 5. Messaging

Durable asynchronous messaging is required for appropriate asynchronous business processing.

The initial implementation may use PostgreSQL/pgmq/Supabase Queues.

The domain model must not depend on pgmq-specific semantics.

A future implementation may replace it with:

* NATS
* RabbitMQ
* Kafka
* Cloud messaging infrastructure
* Another appropriate broker

Do not introduce an external broker until actual requirements justify it.

---

## 6. Events

Use intentional domain events.

Do **not** convert every database update into an event.

Maintain a clear distinction between:

1. Domain events
2. Security/audit events
3. Telemetry observations

Important business events must be durably recorded.

Use a transactional outbox where reliable asynchronous processing is required.

---

## 7. Realtime

Realtime client updates are not a message queue.

Use realtime infrastructure for authorized client-facing state updates.

Use durable queues for backend asynchronous processing.

Never use realtime delivery as a substitute for durable business events.

---

## 8. Authorization

Security is a cross-cutting architectural requirement.

Authorization must be enforced server-side.

Frontend visibility restrictions are not security controls.

Authorization decisions should consider:

* Actor
* Role
* Resource
* Relationship
* Action
* Context
* Data sensitivity

Use RBAC together with contextual/attribute-based authorization where required.

Object-level and field/data-level authorization are required where sensitivity demands them.

An `admin` role must not automatically imply unrestricted access.

---

## 9. Tracking

Tracking is private by default.

There is no anonymous/public tracking API unless explicitly introduced as a future product capability.

Tracking access must be authenticated and authorized according to the actor's relationship with the delivery.

Do not expose unnecessary historical movement information.

---

## 10. Telemetry

Raw GPS telemetry is not a domain event.

Telemetry should be processed through appropriate:

* Filtering
* Deduplication
* Downsampling
* Aggregation

Do not permanently store every location update unless a concrete operational, security, or analytical requirement justifies it.

Telemetry storage must remain replaceable so specialized time-series infrastructure can be introduced later if required.

---

## 11. Domain Model

The following concepts must remain distinct:

* Shipment
* Delivery
* Assignment
* Delivery Partner
* Vehicle

Conceptual relationship:

```text
Shipment
   ↓
Delivery
   ↓
Assignment
   ↓
Delivery Partner
```

The model must preserve the possibility of future:

* Shared routes
* Partner journeys
* Bundled deliveries
* Multi-leg fulfillment
* Distributed capacity matching

without implementing these prematurely.

---

## 12. Matching

Matching and dispatch are separate concerns.

Matching determines:

> Who can do this?

Dispatch determines:

> How do we offer it?

V1 matching should be deterministic and explainable.

Potential matching factors include:

* Availability
* Location
* Service zone
* Vehicle capability
* Package compatibility
* Account status
* Reliability
* Acceptance history
* Zone familiarity

---

## 13. Geography

Geography must be configurable.

Do not hardcode Kano throughout business logic.

Use a hierarchy capable of representing:

```text
Country
  ↓
Region
  ↓
City
  ↓
Service Zone
```

Service zones may determine:

* Availability
* Pricing
* Operating hours
* Vehicle eligibility
* Partner eligibility
* Delivery areas

---

## 14. Pricing

Pricing must be configuration-driven.

Pricing rules must support:

* Versioning
* Effective dates
* Applicable zones
* Vehicle types
* Delivery characteristics

Historical deliveries must preserve the pricing information necessary to explain historical charges.

Today's configuration must not be required to reconstruct yesterday's transaction.

---

## 15. Financial Data

Do not model the financial system as a single `delivery.price` field.

The architecture must support:

* Charges
* Partner earnings
* Platform fees
* Discounts
* Refunds
* Adjustments
* Settlements
* Payouts
* Ledger entries

Financial history must remain auditable.

---

## 16. Delivery Lifecycle

Initial lifecycle:

```text
DRAFT
REQUESTED
SEARCHING
ASSIGNED
ACCEPTED
AT_PICKUP
PICKED_UP
IN_TRANSIT
ARRIVING
DELIVERED
COMPLETED
```

Exceptional states:

```text
CANCELLED
FAILED
RETURN_REQUIRED
RETURNED
DISPUTED
```

State transitions must be controlled by domain operations.

Clients must not arbitrarily mutate delivery status.

---

## 17. Data Minimization

Follow:

> Store the minimum necessary.
> Retain the minimum necessary.
> Expose the minimum necessary.

However, immutable business history required for:

* Disputes
* Accountability
* Financial reconciliation
* Security investigations
* Operational reconstruction
* Auditability

must not be discarded merely to minimize storage.

---

## 18. Initial Technology

Preferred V1 stack:

* TypeScript
* PostgreSQL
* Supabase
* Supabase Auth
* Supabase Storage
* Supabase Realtime
* Supabase Queues / pgmq
* TypeScript backend functions/workers
* TypeScript web application

---

## 19. Deferred Infrastructure

Do not introduce the following without a concrete requirement:

* Kafka
* RabbitMQ
* NATS
* Redis
* Valkey
* Kubernetes
* Microservices
* Specialized telemetry databases
* Dedicated search infrastructure

These technologies are valid future options.

They are simply not assumed to be necessary for V1.

---

## 20. Coding Principles

Prefer:

* Explicit code
* Simple designs
* Strong domain boundaries
* Testable business logic
* Measurable behavior
* Maintainable implementations

Avoid:

* Clever abstractions
* Premature optimization
* Infrastructure fashion
* Framework-driven architecture
* Unnecessary indirection

Business rules belong in the domain/application layer.

Infrastructure-specific behavior belongs in infrastructure implementations.

Infrastructure details should not leak into domain logic unnecessarily.

---

## 21. Architectural Change Control

A locked decision may only be changed when:

1. The user explicitly requests reconsideration; or
2. The current decision creates a demonstrated technical contradiction.

When proposing a change:

1. Identify the locked decision.
2. Explain the concrete problem.
3. Present the proposed alternative.
4. Explain the migration impact.
5. Explain why the existing decision is no longer sufficient.

Never silently alter a locked architectural decision.

Do not make architectural decisions merely because a framework, tutorial, agent, or library recommends them.

Aike should evolve from measured requirements rather than technology fashion.
