**English** | [Русский](ru/gate-validation.md)

# Gate Validation

Gates are checkpoints between roles in the Atomic Spec pipeline. Each gate contains a checklist that must be fully completed before passing an atom to the next role.

## Gate A: Analyst -> Developer

Gate A verifies that the business specification is complete, consistent, and ready for implementation.

| #  | Check                     | Criterion                                                                |
|----|---------------------------|--------------------------------------------------------------------------|
| A1 | Intent is filled in       | 1-3 sentences, no technical details                                      |
| A2 | Domain Rules exist        | At least 1 DR with a unique ID and description of violation consequences |
| A3 | AC are written            | At least 1 Gherkin scenario, technology-neutral                          |
| A4 | Actors are specified      | Filled in frontmatter (`actors`) and referenced in AC                    |
| A5 | Events are specified      | Fields `emits` and `consumes` are filled in frontmatter                  |
| A6 | No technical decisions    | Intent and Domain Rules contain no frameworks, DBs, APIs, endpoints      |
| A7 | Open Questions            | All blocking questions are closed (`status: closed` with `resolution`)   |
| A8 | Change type is specified  | One of: `ParameterChange`, `RuleChange`, `FlowChange`, `ModelChange`, `BoundaryChange` |
| A9 | Negative AC               | For each happy-path (positive) scenario there is at least one negative AC |

### Examples of Gate A Violations

**A1 — violation**: "Buyer cancels an order via REST API POST endpoint" — mention of REST API in Intent.

**A6 — violation**: "DR-003: Data is stored in PostgreSQL in the orders table" — mention of a specific DB in Domain Rules.

**A9 — violation**: There is an AC "Successful order cancellation", but no AC for the case when cancellation is impossible.

## Gate B: Developer -> Tester

Gate B verifies that the technical implementation matches the business specification and is ready for testing.

| #  | Check                          | Criterion                                                                |
|----|--------------------------------|--------------------------------------------------------------------------|
| B1 | Tech Spec is filled in         | API contracts, data schemas, migrations are described                    |
| B2 | Platform section is filled in  | Endpoint, request body, responses with data types                        |
| B3 | Code exists                    | Implementation files are created or paths to them are specified          |
| B4 | AC are not violated            | Tech Spec does not contradict any Acceptance Criteria                    |
| B5 | DR are covered                 | Implementation covers all Domain Rules; each DR is marked in code        |
| B6 | NFR are addressed              | Constraints (PERF, SEC, IDMP) are reflected in implementation            |
| B7 | No TODO/FIXME without OQ       | Every TODO or FIXME in code references a specific Open Question          |
| B8 | Implementation Notes filled in | Compromises, dependencies, limitations, file map — all described         |

### Examples of Gate B Violations

**B4 — violation**: AC says "funds are returned to the original payment method", but Tech Spec describes a refund to the account balance.

**B5 — violation**: DR-002 describes a 30-minute cancellation window, but there is no time check in the code.

**B7 — violation**: Code contains `// TODO: handle edge case` without a reference to an OQ-ID.

## Gate C: Tester -> Done

Gate C verifies that testing fully covers the specification and implementation.

| #  | Check                      | Criterion                                                                |
|----|----------------------------|--------------------------------------------------------------------------|
| C1 | Test Plan is filled in     | Specific test cases with test data (not abstract descriptions)           |
| C2 | AC are covered             | Each Gherkin scenario has a corresponding test case                      |
| C3 | DR are covered             | Each Domain Rule is tested                                               |
| C4 | Edge cases                 | At least 1 negative scenario per AC                                      |
| C5 | NFR are tested             | PERF — benchmark, SEC — security check, IDMP — repeated call            |
| C6 | Coverage Matrix filled in  | AC -> TC -> DR matrix is filled, no empty rows                           |
| C7 | Postconditions             | Not only the response is checked, but also DB state, emitted events      |

### Examples of Gate C Violations

**C2 — violation**: There is AC-003 "Cancellation of a shipped order", but there is no corresponding test case in the Test Plan.

**C5 — violation**: Constraints specify `IDMP: repeated request is idempotent`, but there is no test case with a repeated call.

**C7 — violation**: The test checks for HTTP 200, but does not verify that the order actually transitioned to `cancelled` status in the database and the `order.cancelled` event was published.

## Summary Table

```
Gate A (9 checks)            Gate B (8 checks)            Gate C (7 checks)
─────────────────────        ─────────────────────        ─────────────────────
A1  Intent                   B1  Tech Spec                C1  Test Plan
A2  Domain Rules             B2  Platform API             C2  AC coverage
A3  AC                       B3  Code                     C3  DR coverage
A4  Actors                   B4  AC compliance            C4  Edge cases
A5  Events                   B5  DR coverage              C5  NFR tests
A6  No tech decisions        B6  NFR addressed            C6  Coverage Matrix
A7  Open Questions           B7  TODO/FIXME               C7  Postconditions
A8  Change type              B8  Impl Notes
A9  Negative AC
```

## Validation Process

1. The author creates a PR with the corresponding branch (see [Git Conventions](git-conventions.md)).
2. The reviewer goes through the gate checklist, marking each item.
3. If all items pass — the PR is merged, and the atom moves to the next role.
4. If there are violations — the PR is sent back for revision with specific items indicated.

## Related Documents

- [Roles and Pipeline](roles-and-pipeline.md)
- [Atom Anatomy](atom-anatomy.md)
- [Git Conventions](git-conventions.md)
