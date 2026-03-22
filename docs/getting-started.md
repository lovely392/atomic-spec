**English** | [Русский](ru/getting-started.md)

# Quick Start

This step-by-step guide will help you create your first spec atom using the Atomic Spec methodology.

## Prerequisites

To work with Atomic Spec you only need two tools:

- **Text editor** — any editor that supports Markdown (VS Code, Sublime Text, vim, etc.)
- **Git** — for versioning atoms and moving through the pipeline

No special frameworks, generators, or CLI utilities are required. The methodology operates at the level of files and conventions.

## Step 1. Create the Directory Structure

Create a root directory for specifications in your project:

```
mkdir -p specs/
```

Recommended structure:

```
specs/
  domain-a/
    SPEC-001.md
    SPEC-002.md
  domain-b/
    SPEC-010.md
```

Each domain gets its own subdirectory. The directory name corresponds to the Bounded Context of your domain.

## Step 2. Copy the Template

Copy the atom template file into the appropriate directory:

```
cp templates/atom-template.md specs/domain-a/SPEC-001.md
```

Or create the file manually with the following structure:

```markdown
---
id: SPEC-001
type: feature
title: ""
parent: null
children: []
supersedes: null
see-also: []
emits: []
consumes: []
actors: []
tags: []
open-questions: []
status: draft
implementation: none
verification: none
owners:
  analyst: ""
  developer: ""
  tester: ""
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

## Intent

## Domain Rules

## Acceptance Criteria

## Decision Matrix (DMT)

## Constraints

## Open Questions

## Decision Log
```

## Step 3. Fill in the Atom as an Analyst

The Analyst role is first in the pipeline. You fill in the following sections:

### 3.1. Frontmatter

Fill in the metadata in the YAML block:

- `id` — unique identifier (e.g., `SPEC-001`)
- `type` — atom type (`feature`, `rule`, `constraint`, `event`, `model`)
- `title` — short name in business-friendly language
- `actors` — who participates in the scenario (e.g., `[Buyer, System]`)
- `emits` / `consumes` — events that this atom produces or consumes
- `status: draft` — initial status

### 3.2. Intent

Describe the essence of the atom in 1-3 sentences. No technical details. Business meaning only.

```markdown
## Intent

The buyer can cancel an order within 30 minutes of placing it,
provided the order has not yet been handed off to delivery. Upon
cancellation, funds are returned to the original payment method.
```

### 3.3. Domain Rules

List the domain rules. Each rule has a unique ID and a description of the violation consequence.

```markdown
## Domain Rules

- **DR-001**: Cancellation is only possible in status `placed` or `confirmed`.
  Violation: the system rejects the request with error `ORDER_NOT_CANCELLABLE`.
- **DR-002**: The cancellation window is 30 minutes from the `order.placed` event.
  Violation: the system rejects the request with error `CANCELLATION_WINDOW_EXPIRED`.
```

### 3.4. Acceptance Criteria

Write at least one Gherkin scenario. Scenarios must be technology-neutral.

```markdown
## Acceptance Criteria

**AC-001**: Successful order cancellation
Given an order in status "placed" created 10 minutes ago
When the Buyer requests order cancellation
Then the order transitions to status "cancelled"
And funds are returned to the original payment method

**AC-002**: Cancellation after window expiration
Given an order in status "placed" created 40 minutes ago
When the Buyer requests order cancellation
Then the system rejects the request with error "CANCELLATION_WINDOW_EXPIRED"
```

### 3.5. Constraints

Specify non-functional requirements: performance, security, idempotency, etc.

### 3.6. Open Questions

Record open questions. Blocking questions must be closed before passing the gate.

## Step 4. Passing Gate A

Gate A is the checkpoint between the Analyst and the Developer. Make sure that:

1. Intent is filled in (1-3 sentences, no technical details)
2. There is at least one domain rule with an ID and a violation consequence
3. At least one Gherkin scenario is written
4. Actors are specified in the frontmatter and in the AC
5. Events (emits/consumes) are filled in
6. Intent and Domain Rules contain no technical decisions
7. All blocking Open Questions are closed
8. The change type is specified
9. For every happy-path there is at least one negative AC

Create a PR with the branch `spec/SPEC-001` and go through review.

## Step 5. Developer Phase (brief)

After passing Gate A, the Developer:

- Fills in the Tech Spec, Platform API, and Implementation Notes sections
- Writes the implementation code
- Passes Gate B

## Step 6. Tester Phase (brief)

After passing Gate B, the Tester:

- Fills in the Test Plan and Platform Tests sections
- Builds the Coverage Matrix (AC -> Test Case -> DR)
- Verifies that all AC and DR are covered by tests
- Passes Gate C

After passing Gate C the atom is considered complete: `status: active`, `implementation: done`, `verification: passed`.

## What's Next

- [Full methodology description](methodology.md)
- [Atom anatomy](atom-anatomy.md)
- [Roles and pipeline](roles-and-pipeline.md)
- [Gate validation](ru/gate-validation.md)
