**English** | [Русский](ru/atom-anatomy.md)

# Atom Anatomy

An atom is the minimal indivisible unit of specification in the Atomic Spec methodology. Each atom is a single Markdown file with YAML frontmatter and a set of sections.

## YAML Frontmatter

The frontmatter contains atom metadata. All fields are required unless stated otherwise.

```yaml
---
id: SPEC-042
type: feature
title: "Order cancellation by buyer"
parent: SPEC-040
children: [SPEC-043, SPEC-044]
supersedes: null
see-also: [SPEC-010, SPEC-035]
emits: [order.cancelled, refund.initiated]
consumes: [order.placed, payment.completed]
actors: [Buyer, System, Payment Gateway]
tags: [orders, cancellation, refund]
open-questions:
  - id: OQ-001
    question: "Should the courier be notified on cancellation?"
    status: open
    resolution: null
  - id: OQ-002
    question: "What is the timeout for refund processing?"
    status: closed
    resolution: "72 hours per payment provider SLA"
status: draft
implementation: none
verification: none
owners:
  analyst: "ivanov"
  developer: "petrov"
  tester: "sidorova"
created: 2025-11-15
updated: 2025-11-20
---
```

### Field Descriptions

#### Identification

| Field       | Type     | Description                                                              |
|-------------|----------|--------------------------------------------------------------------------|
| `id`        | string   | Unique atom identifier. Format: `SPEC-NNN` or `DOMAIN-NNN`.             |
| `type`      | enum     | Atom type: `feature`, `rule`, `constraint`, `event`, `model`.            |
| `title`     | string   | Short name in business language. No technical jargon.                    |

#### Hierarchy and Relationships

| Field        | Type     | Description                                                              |
|-------------|----------|--------------------------------------------------------------------------|
| `parent`    | string?  | ID of the parent atom. `null` for root atoms.                            |
| `children`  | string[] | List of child atom IDs. Empty list if there are no children.             |
| `supersedes`| string?  | ID of the atom that this atom replaces. `null` if this is a new atom.    |
| `see-also`  | string[] | References to related atoms (non-hierarchical). For navigation and context. |

#### Events

| Field     | Type     | Description                                                               |
|-----------|----------|---------------------------------------------------------------------------|
| `emits`   | string[] | Domain events produced by this atom (e.g., `order.cancelled`).            |
| `consumes`| string[] | Domain events this atom reacts to (e.g., `order.placed`).                 |

#### Participants

| Field    | Type     | Description                                                                 |
|----------|----------|-----------------------------------------------------------------------------|
| `actors` | string[] | Actors involved in the scenario. Must match actors in the AC.               |
| `tags`   | string[] | Tags for search and grouping. Arbitrary strings.                            |

#### Open Questions

| Field                          | Type   | Description                                                     |
|--------------------------------|--------|-----------------------------------------------------------------|
| `open-questions[].id`         | string | Unique question ID within the atom (e.g., `OQ-001`).           |
| `open-questions[].question`   | string | Question text.                                                  |
| `open-questions[].status`     | enum   | `open` or `closed`.                                             |
| `open-questions[].resolution` | string?| Answer to the question. `null` while the question is open.      |

#### Three Status Axes

| Field            | Type | Values                                            | Description                   |
|-----------------|------|---------------------------------------------------|-------------------------------|
| `status`        | enum | `draft`, `active`, `deprecated`                   | Semantic status of the atom   |
| `implementation`| enum | `none`, `in-progress`, `done`                     | Implementation status         |
| `verification`  | enum | `none`, `in-progress`, `passed`, `failed`         | Verification status           |

For more on the status axes see [Methodology](methodology.md).

#### Owners and Dates

| Field             | Type   | Description                                          |
|-------------------|--------|------------------------------------------------------|
| `owners.analyst`  | string | Login of the analyst responsible for the atom.        |
| `owners.developer`| string | Login of the developer.                               |
| `owners.tester`   | string | Login of the tester.                                  |
| `created`         | date   | Atom creation date (YYYY-MM-DD).                      |
| `updated`         | date   | Last update date (YYYY-MM-DD).                        |

## Atom Sections

After the frontmatter come Markdown sections. Each section belongs to a specific role.

### Analyst Sections

#### Intent

The essence of the atom in 1-3 sentences. Business meaning only, no technical details.

```markdown
## Intent

The buyer can cancel an order within 30 minutes of placing it,
provided the order has not yet been handed off to delivery.
```

**Rule**: if Intent mentions a database, framework, API, or endpoint, that is a violation.

#### Domain Rules

Domain rules. Each rule has a unique ID and a description of the violation consequence.

```markdown
## Domain Rules

- **DR-001**: Cancellation is only possible in statuses `placed` and `confirmed`.
  Violation: `ORDER_NOT_CANCELLABLE`.
- **DR-002**: Cancellation window is 30 minutes from the `order.placed` event.
  Violation: `CANCELLATION_WINDOW_EXPIRED`.
- **DR-003**: On cancellation, a refund is initiated to the original payment method.
  Violation: not applicable — system rule.
```

#### Acceptance Criteria

Gherkin scenarios. Technology-neutral. At least one positive and one negative scenario.

```markdown
## Acceptance Criteria

**AC-001**: Successful cancellation
Given an order in status "placed" created 10 minutes ago
When the Buyer requests cancellation
Then the order transitions to status "cancelled"
And the system emits an `order.cancelled` event
And a refund is initiated

**AC-002**: Cancellation after window expiration
Given an order created 40 minutes ago
When the Buyer requests cancellation
Then the system rejects the request
And returns error "CANCELLATION_WINDOW_EXPIRED"
```

#### Decision Matrix (DMT)

A decision table for complex condition combinations.

```markdown
## Decision Matrix (DMT)

| Order Status  | Time Since Creation | Result                |
|---------------|---------------------|-----------------------|
| placed        | < 30 min            | Cancellation allowed  |
| placed        | >= 30 min           | Cancellation denied   |
| confirmed     | < 30 min            | Cancellation allowed  |
| shipped       | any                 | Cancellation denied   |
```

#### Constraints

Non-functional requirements: performance, security, idempotency.

```markdown
## Constraints

- **PERF**: The cancellation operation must complete in < 500ms (p99).
- **SEC**: Only the order owner can cancel the order.
- **IDMP**: A repeated cancellation request for an already cancelled order is idempotent (200 OK).
```

#### Open Questions

Expanded description of open questions (duplicated from frontmatter for readability).

#### Decision Log

A chronological log of decisions made, with rationale.

```markdown
## Decision Log

- **2025-11-15**: Cancellation window set to 30 minutes (not 60). Reason: logistics
  begins processing after 30 minutes; cancellation after that incurs additional costs.
- **2025-11-18**: Refund to the original payment method (not to account balance). Reason:
  legal requirement under consumer protection law.
```

### Developer Sections

#### Tech Spec

API contracts, data schemas, migrations.

```markdown
## Tech Spec

### Model Changes

Added field `cancelled_at: timestamp?` to the `orders` table.
Migration: `V042__add_cancelled_at_to_orders.sql`.

### Dependencies

- `payment-gateway` service for initiating the refund.
- `refund.initiated` event is published to the Kafka topic `payments`.
```

#### Platform API

Specific endpoints, request body, responses with types.

```markdown
## Platform API

### POST /api/v1/orders/{orderId}/cancel

**Request Body**: empty

**Response 200**:
```json
{
  "orderId": "string",
  "status": "cancelled",
  "cancelledAt": "ISO-8601"
}
```

**Response 409**:
```json
{
  "error": "ORDER_NOT_CANCELLABLE",
  "message": "string"
}
```

**Response 410**:
```json
{
  "error": "CANCELLATION_WINDOW_EXPIRED",
  "message": "string"
}
```
```

#### Implementation Notes

Trade-offs, dependencies, limitations, file map.

```markdown
## Implementation Notes

### File Map
- `src/orders/cancel-order.usecase.ts` — main use case
- `src/orders/order.entity.ts` — added `cancel()` method
- `migrations/V042__add_cancelled_at.sql` — migration

### Trade-offs
- Refund is asynchronous: the `refund.initiated` event is processed
  by a separate consumer.

### Limitations
- If payment-gateway is unavailable, the `refund.initiated` event goes to the DLQ.
```

### Tester Sections

#### Test Plan

Specific test cases with test data.

```markdown
## Test Plan

| TC ID  | Description                       | Input Data                     | Expected Result               |
|--------|-----------------------------------|--------------------------------|-------------------------------|
| TC-001 | Successful cancellation           | order.status=placed, age=10min | 200, status=cancelled         |
| TC-002 | Cancellation of expired order     | order.status=placed, age=40min | 410, WINDOW_EXPIRED           |
| TC-003 | Cancellation of shipped order     | order.status=shipped           | 409, NOT_CANCELLABLE          |
| TC-004 | Repeated cancellation (idempotency)| order.status=cancelled        | 200, status=cancelled         |
```

#### Platform Tests

Automated tests linked to TC.

#### Coverage Matrix

Mapping between AC, test cases, and domain rules.

```markdown
## Coverage Matrix

| AC     | TC              | DR              |
|--------|-----------------|-----------------|
| AC-001 | TC-001          | DR-001, DR-003  |
| AC-002 | TC-002          | DR-002          |
| —      | TC-003          | DR-001          |
| —      | TC-004          | IDMP constraint |
```

## Who Writes What

| Section              | Owner         | Gate      |
|----------------------|---------------|-----------|
| Intent               | Analyst       | Gate A    |
| Domain Rules         | Analyst       | Gate A    |
| Acceptance Criteria  | Analyst       | Gate A    |
| Decision Matrix      | Analyst       | Gate A    |
| Constraints          | Analyst       | Gate A    |
| Open Questions       | Analyst       | Gate A    |
| Decision Log         | Analyst       | Gate A    |
| Tech Spec            | Developer     | Gate B    |
| Platform API         | Developer     | Gate B    |
| Implementation Notes | Developer     | Gate B    |
| Test Plan            | Tester        | Gate C    |
| Platform Tests       | Tester        | Gate C    |
| Coverage Matrix      | Tester        | Gate C    |

## Progressive Disclosure Rule

Sections are filled in strictly in pipeline order:

1. **Analyst** fills in their sections and passes Gate A.
2. **Developer** reads the Analyst's sections as an input contract, fills in their own sections, and passes Gate B.
3. **Tester** reads the Analyst's and Developer's sections as an input contract, fills in their own sections, and passes Gate C.

Backward modification (e.g., a Developer editing Intent) is only permitted through the [amendment](ru/amendments.md) process.

## Related Documents

- [Methodology](methodology.md)
- [Roles and Pipeline](roles-and-pipeline.md)
- [Gate Validation](ru/gate-validation.md)
