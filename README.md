# Aike

A technology-enabled delivery marketplace connecting merchants and customers with distributed delivery capacity.

**Aike** — from the Hausa word for *errand* — starts in Kano, Nigeria, with a merchant-first delivery model supporting professional dispatch riders, part-time riders, private vehicle owners, fleet operators, and other eligible delivery partners.

The long-term vision is larger than a conventional dispatch service: Aike is intended to become a marketplace for distributed delivery capacity, allowing existing vehicles, journeys, and unused carrying capacity to participate in the logistics network.

---

## Product Vision

Aike connects:

* Merchants and businesses that need deliveries
* Customers receiving deliveries
* Delivery partners with available transport capacity
* The platform coordinating matching, dispatch, tracking, authorization, settlement, and operations

The initial delivery flow is:

```text
Merchant / Customer
        ↓
Delivery Request
        ↓
Eligibility & Matching
        ↓
Dispatch
        ↓
Partner Acceptance
        ↓
Pickup
        ↓
Transit
        ↓
Delivery
        ↓
Proof of Delivery
        ↓
Settlement / Completion
```

Aike does **not** assume that every delivery partner is a professional dispatch rider.

Partners may include:

* Professional dispatch riders
* Part-time riders
* Private motorcycle owners
* Car owners
* Van owners
* Fleet operators
* Other future transport-capable partners

---

## Initial Market

### Launch Market

Kano, Nigeria.

The geographic model must remain configurable rather than hardcoded to Kano.

```text
Country
  └── Region
       └── City
            └── Service Zone
```

Service zones may define:

* Service availability
* Operating hours
* Vehicle eligibility
* Pricing
* Delivery areas
* Partner eligibility
* Operational rules

---

## Core Product Areas

Aike is being designed around the following modules:

* Identity & Authentication
* Merchants
* Merchant Users
* Customers
* Delivery Partners
* Vehicles
* Shipments
* Deliveries
* Assignments
* Matching
* Dispatch
* Tracking
* Service Zones
* Pricing
* Settlement
* Payments
* Proof of Delivery
* Notifications
* Authorization
* Audit & Event History
* Operations
* Analytics

---

## Architecture

### Modular Monolith First

Aike will begin as a modular monolith.

```text
Application
├── Identity
├── Merchant
├── Partner
├── Shipment
├── Delivery
├── Matching
├── Dispatch
├── Tracking
├── Pricing
├── Settlement
├── Authorization
└── Operations
```

Modules must maintain clear ownership boundaries so that individual modules can be extracted into independent services later if actual scale requires it.

Microservices are **not** a V1 requirement.

---

## Event-Driven Internals

Important business operations should produce intentional domain events.

For example:

```text
PackagePickedUp
        ↓
 ┌──────┼───────────────┐
 ↓      ↓               ↓
Notify  Analytics       Settlement
```

Domain events are distinct from:

* Security/audit events
* High-frequency telemetry
* Generic database updates

Not every database mutation is a domain event.

---

## Transactional Integrity

Important state changes should use a transactional outbox pattern:

```text
Command
   ↓
Application / Domain Service
   ↓
PostgreSQL Transaction
   ├── Business State
   ├── Domain Event
   └── Outbox Record
            ↓
        Message Queue
            ↓
       Async Workers
```

This ensures business state and the corresponding event/outbox record are committed atomically.

---

## Infrastructure Strategy

Aike will use managed infrastructure during V1 where doing so reduces operational complexity.

Supabase may initially provide:

* PostgreSQL
* Authentication
* Storage
* Realtime
* Queues
* Edge Functions
* Scheduled jobs

However:

> **Supabase is an implementation choice, not the architecture.**

The application must avoid unnecessary vendor lock-in.

Infrastructure capabilities should have clean boundaries where provider replacement is reasonably likely.

For example:

```ts
interface MessageBus {
  publish(message: DomainMessage): Promise<void>;
  consume(queue: string): Promise<Message>;
  acknowledge(message: Message): Promise<void>;
}
```

The initial implementation may use PostgreSQL/pgmq.

A future implementation could use:

* NATS
* RabbitMQ
* Kafka
* Cloud messaging infrastructure
* Another appropriate broker

without requiring the domain model to change.

The same principle applies to:

* Object storage
* Authentication
* Realtime
* Telemetry
* Caching
* Background processing

---

## PostgreSQL

PostgreSQL is the primary transactional system of record.

Aike should prefer standard PostgreSQL capabilities wherever practical.

The schema should remain portable across:

* Supabase PostgreSQL
* Managed PostgreSQL
* Self-hosted PostgreSQL

Supabase-specific functionality should not unnecessarily become part of the domain model.

---

## Authorization & Security

Security is a first-class architectural concern.

Aike must not rely on frontend restrictions to protect data.

Authorization should consider:

```text
Actor
Role
Resource
Relationship
Action
Context
Sensitivity
```

The system should support:

* Authentication
* RBAC
* Contextual / attribute-based authorization
* Object-level authorization
* Field/data-level access control
* Merchant/tenant isolation
* Scoped administrative access
* Audit logging
* Sensitive-data protection
* Rate limiting
* Session/device security
* Retention policies

Example:

```text
Merchant
  → Accesses their own deliveries

Assigned Partner
  → Accesses information required to perform the assigned delivery

Operations
  → Accesses operational information within their authorized scope

Security / Investigation
  → May access restricted historical information when explicitly authorized
```

An `admin` role must not automatically mean unrestricted access.

---

## Data Classification

Sensitive data should be classified and exposed according to need.

Initial conceptual classification:

```text
PUBLIC
INTERNAL
OPERATIONAL
CONFIDENTIAL
RESTRICTED
```

Examples:

| Data                    | Classification |
| ----------------------- | -------------- |
| Delivery status         | Operational    |
| Pickup/dropoff location | Confidential   |
| Live partner location   | Confidential   |
| Package contents        | Confidential   |
| Package value           | Confidential   |
| Partner earnings        | Restricted     |
| Payment information     | Restricted     |
| Fraud/risk information  | Restricted     |

The backend must enforce these boundaries.

---

## Tracking

Tracking is a private platform capability.

There is **no anonymous/public tracking API by default**.

Tracking access is authenticated and authorized according to the actor's relationship with the delivery.

Aike should not expose unnecessary historical movement information.

### Telemetry

Raw GPS telemetry is not equivalent to a business event.

The intended pipeline is:

```text
Partner Device
      ↓
Location Updates
      ↓
Telemetry Processing
      ↓
Filtering / Deduplication
      ↓
Downsampling / Aggregation
      ↓
Operational Tracking Data
```

The system should avoid permanently storing every GPS ping unless operational, security, or analytical requirements justify it.

Telemetry storage should be abstracted sufficiently to allow migration to specialized time-series infrastructure later.

---

## Domain Model

A shipment represents the thing being transported.

A delivery represents the operational fulfillment of that shipment.

An assignment represents the relationship between a delivery and a selected partner.

```text
Shipment
   ↓
Delivery
   ↓
Assignment
   ↓
Delivery Partner
```

These concepts must remain distinct.

This allows future support for:

* Dedicated delivery
* Shared routes
* Partner journeys
* Bundled deliveries
* Multi-leg fulfillment
* Distributed capacity matching

without prematurely implementing those models.

---

## Matching & Dispatch

Matching and dispatch are separate concerns.

### Matching

Matching determines which partners are eligible and suitable.

```text
Eligibility
    ↓
Candidate Partners
    ↓
Scoring
    ↓
Ranked Candidates
```

Potential factors include:

* Availability
* Location
* Service zone
* Vehicle capability
* Package compatibility
* Account status
* Reliability
* Acceptance history
* Zone familiarity

### Dispatch

Dispatch determines how delivery opportunities are offered.

Possible strategies include:

* Sequential offers
* Parallel offers
* Broadcast
* Timed offers
* Escalation

Matching answers:

> **Who can do this?**

Dispatch answers:

> **How do we offer it?**

---

## Pricing

Pricing must be configuration-driven rather than hardcoded throughout the application.

Pricing configurations should support:

* Versioning
* Effective dates
* Service zones
* Vehicle types
* Delivery characteristics
* Applicable pricing rules

Historical deliveries must retain a pricing snapshot so historical charges remain explainable after pricing rules change.

---

## Financial Model

Financial operations must not be represented solely by a `delivery.price` field.

The architecture should support:

* Customer charges
* Merchant charges
* Partner earnings
* Platform fees
* Discounts
* Refunds
* Adjustments
* Settlements
* Payouts
* Ledger entries

Financial history must remain auditable and reconstructable.

---

## Domain Events

Examples include:

```text
ShipmentCreated
DeliveryRequested
PartnerAssigned
AssignmentAccepted
AssignmentRejected
PartnerArrivedAtPickup
PackagePickedUp
DeliveryStarted
DeliveryAttempted
DeliveryCompleted
ProofOfDeliverySubmitted
DeliveryCancelled
SettlementCreated
PayoutCreated
```

Security/audit events are a separate concept.

Examples:

```text
UserViewedShipment
AdminViewedPackageValue
AdminChangedPricingRule
UserModifiedDelivery
```

Important historical events should be durable and reconstructable.

---

## Delivery Lifecycle

Initial lifecycle:

```text
DRAFT
  ↓
REQUESTED
  ↓
SEARCHING
  ↓
ASSIGNED
  ↓
ACCEPTED
  ↓
AT_PICKUP
  ↓
PICKED_UP
  ↓
IN_TRANSIT
  ↓
ARRIVING
  ↓
DELIVERED
  ↓
COMPLETED
```

Exceptional states include:

```text
CANCELLED
FAILED
RETURN_REQUIRED
RETURNED
DISPUTED
```

State transitions must be controlled by domain operations.

Clients must not be able to arbitrarily mutate delivery status.

---

## Development Philosophy

> **Minimum infrastructure that works. Clean boundaries that allow growth.**

Do not introduce infrastructure merely because it is popular.

Do not introduce microservices because the architecture diagram looks better.

Do not introduce Kafka because the system has events.

Do not introduce Redis because a backend supposedly needs Redis.

Do not introduce Kubernetes because the platform may eventually scale.

Build the simplest system that correctly solves the current problem while preserving the boundaries required for future evolution.

Infrastructure should be introduced or replaced when actual workload, reliability, operational, scaling, or cost requirements justify it.

---

## Initial Technology Direction

### V1

* TypeScript
* PostgreSQL
* Supabase
* Supabase Auth
* Supabase Storage
* Supabase Realtime
* Supabase Queues / pgmq
* TypeScript backend functions/workers
* TypeScript web application

### Possible Future Infrastructure

Depending on actual requirements, Aike may evolve toward:

* Dedicated application servers
* Background workers
* NATS / RabbitMQ / Kafka
* Redis / Valkey
* S3-compatible object storage
* Dedicated realtime infrastructure
* TimescaleDB or specialized telemetry storage
* Dedicated search infrastructure
* Independently scalable services

These technologies should only be introduced when justified by actual requirements.

---

## Repository Structure

The repository will evolve toward a structure similar to:

```text
aike/
├── apps/
│   ├── web/
│   └── api/
│
├── packages/
│   ├── domain/
│   ├── application/
│   ├── authorization/
│   ├── infrastructure/
│   ├── contracts/
│   └── shared/
│
├── database/
│   ├── migrations/
│   ├── seeds/
│   └── functions/
│
├── docs/
│   ├── architecture/
│   ├── domain/
│   ├── security/
│   ├── api/
│   └── decisions/
│
└── README.md
```

The exact structure may change as implementation begins.

Architecture should follow actual boundaries rather than forcing code into a predetermined directory structure.

---

## Initial Implementation Scope

The first implementation phase focuses on establishing:

1. Core domain model
2. PostgreSQL schema
3. Authentication
4. Authorization model
5. Merchant model
6. Partner model
7. Vehicle model
8. Shipment/delivery model
9. Assignment model
10. Delivery lifecycle
11. Matching engine
12. Dispatch mechanism
13. Event/outbox infrastructure
14. Basic tracking
15. Proof of delivery
16. Pricing
17. Settlement foundations
18. Operational APIs
19. Web application foundations

Advanced distributed-capacity marketplace functionality is intentionally deferred until the core delivery platform is operational.

---

## Long-Term Direction

Aike is ultimately intended to evolve from a conventional delivery platform into a marketplace for distributed logistics capacity.

Instead of assuming:

```text
Delivery Request
      ↓
Professional Rider
```

the mature model can become:

```text
Delivery Requirement
        ↓
Available Delivery Capacity
        ↓
Matching
        ↓
Route / Journey / Vehicle Capacity
        ↓
Fulfillment
```

This may eventually allow Aike to coordinate:

* Existing journeys
* Unused vehicle capacity
* Shared deliveries
* Route-based fulfillment
* Multi-leg delivery
* Fleet capacity
* Opportunistic delivery partners

The V1 architecture should preserve this possibility without prematurely implementing it.

---

## Status

Early architecture and implementation phase.

The repository should become the source of truth for the technical implementation as decisions are finalized.
