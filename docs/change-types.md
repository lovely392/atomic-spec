**English** | [Русский](ru/change-types.md)

# Change Types

Each atom in Atomic Spec has a change type (Change Type) that determines the scope of impact, the list of stakeholders, and the approval process.

## Six Change Types

### 1. ParameterChange

Change to a constant value or configuration parameter.

| Characteristic   | Value                           |
|------------------|---------------------------------|
| **Scope**        | Minimal                         |
| **Stakeholders** | 1 stakeholder (parameter owner) |
| **Blocking**     | No                              |
| **Requires RFC** | No                              |

**Examples:**
- Changing the order cancellation timeout from 30 minutes to 60 minutes
- Changing the minimum order amount from 500 to 300
- Updating the password attempt limit from 3 to 5

**What changes in the atom:** only the value in Domain Rules or Constraints. The atom structure is not affected.

---

### 2. RuleChange

Change to a domain rule — conditions, logic, violation consequences.

| Characteristic   | Value                           |
|------------------|---------------------------------|
| **Scope**        | Medium                          |
| **Stakeholders** | Product + Tech                  |
| **Blocking**     | Possibly                        |
| **Requires RFC** | No                              |

**Examples:**
- Allow order cancellation not only in `placed` status, but also in `processing`
- Add a new condition to the validation rule (DR-004)
- Change the violation consequence from error to warning

**What changes in the atom:** Domain Rules, possibly AC (new scenarios), DMT.

---

### 3. FlowChange

Change to a step or branching in a use case scenario.

| Characteristic   | Value                           |
|------------------|---------------------------------|
| **Scope**        | Medium                          |
| **Stakeholders** | Product                         |
| **Blocking**     | Possibly                        |
| **Requires RFC** | No                              |

**Examples:**
- Add a confirmation step before order cancellation
- Introduce an alternative scenario when the payment gateway is unavailable
- Change the order of steps in the checkout process

**What changes in the atom:** AC (new/modified scenarios), possibly Domain Rules, DMT.

---

### 4. ModelChange

Change to an aggregate, field, event, or data structure.

| Characteristic   | Value                           |
|------------------|---------------------------------|
| **Scope**        | Significant                     |
| **Stakeholders** | Product + Tech + Architect      |
| **Blocking**     | Yes                             |
| **Requires RFC** | No (but requires Architect review) |

**Examples:**
- Add a `cancellation_reason` field to the Order aggregate
- Create a new domain event `order.partially_cancelled`
- Change the payload structure of the `order.cancelled` event
- Split the Order aggregate into Order and OrderFulfillment

**What changes in the atom:** Domain Rules, emits/consumes, Tech Spec, Platform API, possibly children.

---

### 5. BoundaryChange

Change to a domain boundary — transfer of responsibility, merging or splitting contexts.

| Characteristic   | Value                           |
|------------------|---------------------------------|
| **Scope**        | Maximum                         |
| **Stakeholders** | CTO + Architect + Product       |
| **Blocking**     | Yes                             |
| **Requires RFC** | Yes                             |

**Examples:**
- Extract the refund process from the Orders domain into a separate Refunds domain
- Merge the Payments and Billing domains
- Transfer notification responsibility from Orders to Notifications

**What changes in the atom:** parent/children, possibly creation of new atoms, deprecation of old ones, changes to emits/consumes across the entire graph.

---

### 6. PlatformChange

Change to an API contract without changing business logic.

| Characteristic   | Value                           |
|------------------|---------------------------------|
| **Scope**        | Technical                       |
| **Stakeholders** | Tech Lead                       |
| **Blocking**     | No                              |
| **Requires RFC** | No                              |

**Examples:**
- Renaming an endpoint while maintaining backward compatibility
- Adding a new optional field to the API response
- Changing the date format from Unix timestamp to ISO-8601
- API versioning (v1 -> v2) while preserving behavior

**What changes in the atom:** Platform API, Tech Spec. Domain Rules and AC are not affected.

## Summary Table

| Type             | Scope         | Stakeholders          | Blocking    | RFC  |
|------------------|---------------|-----------------------|-------------|------|
| ParameterChange  | Minimal       | 1 stakeholder         | No          | No   |
| RuleChange       | Medium        | Product + Tech        | Possibly    | No   |
| FlowChange       | Medium        | Product               | Possibly    | No   |
| ModelChange      | Significant   | Product + Tech + Arch | Yes         | No   |
| BoundaryChange   | Maximum       | CTO + Arch + Product  | Yes         | Yes  |
| PlatformChange   | Technical     | Tech Lead             | No          | No   |

## Decision Matrix: New Atom or Edit Existing?

Not every change requires creating a new atom. Use this matrix to make the decision:

| Situation                                                   | Action                      |
|------------------------------------------------------------|-----------------------------|
| Changing a parameter value (ParameterChange)                | Edit existing atom          |
| Adding a new DR to an existing scenario (RuleChange)        | Edit existing atom          |
| New scenario in the same context (FlowChange)               | Edit existing atom          |
| Significant scope expansion of an existing atom             | New child atom              |
| New independent behavior (FlowChange)                       | New atom                    |
| New aggregate or event (ModelChange)                        | New atom                    |
| Replacing existing behavior (any type)                      | New atom with `supersedes`  |
| Changing domain boundary (BoundaryChange)                   | New atoms + deprecation     |
| Purely technical API changes (PlatformChange)               | Edit existing atom          |

### Rule: Scope Does Not Expand

If a change causes an atom to start describing two different scenarios or two different aggregates — this is a signal for decomposition. Create a child atom or a new standalone atom.

## Related Documents

- [Git Conventions](git-conventions.md)
- [Amendments](amendments.md)
- [Gate Validation](gate-validation.md)
