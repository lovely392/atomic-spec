**English** | [Русский](ru/amendments.md)

# Amendments

Amendments are the mechanism for making changes to atoms that have already passed one or more gates. Amendments ensure change traceability and protect contractual boundaries between roles.

## Three Amendment Scenarios

### Scenario 1: Atom in Draft Status (before passing all gates)

The atom is still in progress and has not reached `active` status. Changes are made directly to the original atom file.

**When:** The Analyst discovered an error in a DR before passing Gate A, or the Developer found a contradiction in AC during implementation.

**Process:**
1. The author of the change creates a branch `amend/<ID>-<description>`.
2. Makes changes to the original atom file.
3. Updates the `updated` field in frontmatter.
4. Adds an entry to the Decision Log with justification for the change.
5. Creates a PR with the checklist of the affected gate.

**Important:** if the change affects sections belonging to a previous role (e.g., the Developer edits AC), review from the owner of those sections is required.

### Scenario 2: Atom Returned for Revision (pulled back)

The atom did not pass the gate and was returned to the previous role.

**When:** The reviewer at Gate B discovered that the Tech Spec contradicts AC. Or at Gate C the tester found a missing DR.

**Process:**
1. The PR review specifies the exact checklist items that were not passed.
2. The author fixes the atom in the same branch.
3. If changes to sections of another role are needed — an amendment file is created.
4. The PR undergoes re-review.

### Scenario 3: Atom is Sprint-Locked

The atom has `active` status, its implementation and tests are complete. A change is required.

**When:** Business requirements changed, a defect was found in production, or functionality needs to be extended.

**Process:**
1. An **amendment file** (a separate amendment file) is created.
2. The amendment file goes through the full pipeline (Gate A -> Gate B -> Gate C).
3. After passing all gates — the changes are applied to the original atom.
4. If the scope of the change is significant — a new atom with `supersedes` is created.

## Amendment File Format

An amendment is a separate Markdown file with its own YAML frontmatter.

```yaml
---
id: SPEC-042a
type: amendment
title: "Extend order cancellation window to 60 minutes"
target: SPEC-042
change-type: ParameterChange
reason: "Analytics showed that 78% of cancellations occur in the 30-45 minute interval"
status: draft
implementation: none
verification: none
owners:
  analyst: "ivanov"
  developer: "petrov"
  tester: "sidorova"
created: 2025-12-10
updated: 2025-12-10
---

## What Changes

### Domain Rules

- **DR-002** (change): Cancellation window — ~~30 minutes~~ **60 minutes** from `order.placed`.

### Acceptance Criteria

- **AC-002** (change): Given the order was created ~~40~~ **70** minutes ago

### Constraints

No changes.

## Justification

According to Q3 2025 analytics, 78% of cancellation requests fall within the
30-45 minute interval after placing the order. The current 30-minute limit leads to
increased support load (manual cancellation).

## Impact Analysis

- **Affected atoms**: SPEC-042 (target), SPEC-043 (child — notifications)
- **Breaking**: No (window expansion, not contraction)
- **Migrations**: Not required (configuration parameter)

## Decision Log

- **2025-12-10**: Decision to extend the window from 30 to 60 minutes.
  Initiator: Product Owner. Basis: Q3 2025 analytics.
```

## Naming Convention

Amendment files are named by adding a suffix to the target atom ID:

```
SPEC-042.md       — original atom
SPEC-042a.md      — first amendment
SPEC-042b.md      — second amendment
SPEC-042c.md      — third amendment
```

The suffix is a lowercase Latin letter, added sequentially.

**File location:**

```
specs/
  orders/
    SPEC-042.md
    SPEC-042a.md
    SPEC-042b.md
```

Amendment files are placed next to the target atom in the same directory.

## Branches and Commits for Amendments

```
Branch:  amend/SPEC-042a-extend-window
Commit:  amend(SPEC-042a): extend cancellation window to 60 minutes
PR:      [Amend] SPEC-042a: Extend cancellation window
```

## Principles for Working with Amendments

### 1. Atoms Are Never Deleted

An atom is never deleted from the repository. Instead of deletion, `status: deprecated` is used. The new atom specifies `supersedes: SPEC-042` in frontmatter.

```yaml
# Old atom SPEC-042
status: deprecated

# New atom SPEC-042-v2
supersedes: SPEC-042
status: active
```

### 2. Add Leaves, Modify Branches Minimally

When changing the atom hierarchy, prefer adding child atoms (leaves) over modifying existing ones (branches). This reduces the risk of cascading changes.

```
# Preferred: add a leaf
SPEC-042
  └── SPEC-042-notify (new child atom)

# Avoid: rewrite the branch
SPEC-042 (completely reworked)
```

### 3. Scope Does Not Expand

An amendment cannot expand the scope of the original atom. If an amendment causes the atom to start describing a new scenario — this is a signal for decomposition.

One amendment — one specific change.

### 4. Domain Rules: Append-Only

Domain rules are not deleted or modified directly. Instead:

- **Adding a new DR**: simply add the new rule.
- **Changing an existing DR**: create an amendment with explicit indication of what exactly changes.
- **Removing a DR**: mark the rule as `[DEPRECATED]` and specify the reason in the Decision Log.

```markdown
- **DR-002** [DEPRECATED]: Cancellation window — 30 minutes.
  Replaced by DR-005 (see SPEC-042a).
- **DR-005**: Cancellation window — 60 minutes from `order.placed`.
```

### 5. Open Questions Block Implementation, Not Documentation

An open question (`status: open`) blocks passing Gate A and starting implementation, but does not block the Analyst from working on documentation. The Analyst can continue filling in sections, noting assumptions.

```yaml
open-questions:
  - id: OQ-003
    question: "Is double authorization needed for canceling large orders?"
    status: open
    resolution: null
```

While OQ-003 is open, Gate A cannot be passed. But the Analyst can write AC assuming an answer and note this in the Decision Log.

### 6. No Domain Changes Without an Explicit Decision

Every change to a domain rule, scenario, or constraint must be recorded in the Decision Log with:

- Who made the decision
- On what basis (data, regulatory requirement, incident)
- What alternatives were considered

```markdown
## Decision Log

- **2025-12-10**: Extend cancellation window from 30 to 60 minutes.
  Decision: Product Owner (Ivanov I.I.)
  Basis: Q3 2025 analytics, 78% of cancellations in the 30-45 min interval.
  Alternatives: 45 minutes (rejected — insufficient margin),
  no limit (rejected — logistics risks).
```

## Related Documents

- [Change Types](change-types.md)
- [Git Conventions](git-conventions.md)
- [Gate Validation](gate-validation.md)
- [Atom Anatomy](atom-anatomy.md)
