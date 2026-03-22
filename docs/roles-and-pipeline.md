**English** | [Русский](ru/roles-and-pipeline.md)

# Roles and Pipeline

The Atomic Spec methodology defines three roles. Each role owns specific atom sections, reads specific sections as an input contract, and is responsible for passing its gate.

## Pipeline

```
┌─────────────┐     Gate A     ┌─────────────┐     Gate B     ┌─────────────┐     Gate C     ┌──────┐
│  Analyst    │───────────────>│  Developer  │───────────────>│   Tester    │───────────────>│ Done │
│             │                │             │                │             │                │      │
│ Intent      │                │ Tech Spec   │                │ Test Plan   │                │active│
│ Domain Rules│                │ Platform API│                │ Platform    │                │done  │
│ AC          │                │ Impl Notes  │                │  Tests      │                │passed│
│ DMT         │                │ Code        │                │ Coverage    │                │      │
│ Constraints │                │             │                │  Matrix     │                │      │
│ Open Q      │                │             │                │ AC Review   │                │      │
│ Decision Log│                │             │                │             │                │      │
└─────────────┘                └─────────────┘                └─────────────┘                └──────┘
```

## Role: Analyst

### Owns Sections

- **Intent** — the essence of the atom in 1-3 sentences
- **Domain Rules** — domain rules with IDs and violation consequences
- **Decision Matrix (DMT)** — decision table for condition combinations
- **Constraints** — non-functional requirements (PERF, SEC, IDMP, etc.)
- **Acceptance Criteria** (draft) — Gherkin scenarios, technology-neutral
- **Open Questions** — capturing and closing open questions
- **Decision Log** — log of decisions made, with rationale

### Reads (Input Contract)

- Business requirements, Event Storming results, user requests
- Existing atoms (parent, see-also) for understanding context

### Output Contract

An atom that has passed Gate A: all analyst sections are filled in, blocking questions are closed, and the change type is specified.

### Area of Responsibility

- Correctness of domain rules
- Completeness of scenarios (positive and negative)
- Absence of technical decisions in business sections
- Closing blocking Open Questions

## Role: Developer

### Owns Sections

- **Tech Spec** — API contracts, data schemas, migrations
- **Platform API** — specific endpoints, request/response body with types
- **Implementation Notes** — trade-offs, dependencies, limitations, file map
- **Code** — implementation files

### Reads (Input Contract)

| Section              | What Is Extracted                                          |
|----------------------|------------------------------------------------------------|
| Intent               | Understanding of business context                          |
| Domain Rules         | Rules that the code must implement                         |
| Acceptance Criteria  | Scenarios that the code must support                       |
| Constraints          | NFR: performance, security, idempotency                    |
| DMT                  | Combinatorial logic for branching                          |
| Open Questions       | Unresolved questions affecting implementation              |

### Output Contract

An atom that has passed Gate B: Tech Spec is filled in, code is written, AC do not contradict the implementation, and all DR are covered in code.

### Area of Responsibility

- Technical correctness of the implementation
- Code compliance with domain rules
- No TODO/FIXME without a link to an Open Question
- Filling in Implementation Notes for handing off context to the tester

## Role: Tester

### Owns Sections

- **Test Plan** — specific test cases with test data
- **Platform Tests** — automated tests
- **Coverage Matrix** — coverage matrix AC -> TC -> DR
- **AC Review** — review of acceptance criteria for completeness and testability

### Reads (Input Contract)

| Section              | What Is Extracted                                          |
|----------------------|------------------------------------------------------------|
| Acceptance Criteria  | Scenarios that need to be covered by tests                 |
| Domain Rules         | Rules, each of which must be tested                        |
| Constraints          | NFR for specialized testing (benchmarks, security)         |
| Tech Spec            | API contracts for writing integration tests                |
| Platform API         | Endpoints, request/response formats for tests              |
| Implementation Notes | Dependencies and limitations affecting the test plan       |

### Output Contract

An atom that has passed Gate C: Test Plan is filled in, every AC is covered by a test, every DR is tested, and the Coverage Matrix is filled in.

### Area of Responsibility

- Completeness of test coverage
- Presence of negative scenarios
- Testing of NFR (benchmarks, security, idempotency)
- Verifying not only the response but also side effects (DB state, events)

## Contract-Based Work

The key principle of the pipeline is contract-based work between roles:

```
Analyst                          Developer                        Tester
    │                                 │                                 │
    │  writes Intent, DR, AC          │                                 │
    │  writes DMT, Constraints        │                                 │
    │  closes Open Questions          │                                 │
    │                                 │                                 │
    ├── Gate A ──────────────────────>│                                 │
    │                                 │  reads Intent, DR, AC           │
    │                                 │  reads Constraints, DMT         │
    │                                 │  writes Tech Spec, Platform API │
    │                                 │  writes Implementation Notes    │
    │                                 │  writes Code                    │
    │                                 │                                 │
    │                                 ├── Gate B ─────────────────────>│
    │                                 │                                 │  reads AC, DR, Constraints
    │                                 │                                 │  reads Tech Spec, Platform API
    │                                 │                                 │  writes Test Plan
    │                                 │                                 │  writes Platform Tests
    │                                 │                                 │  builds Coverage Matrix
    │                                 │                                 │
    │                                 │                                 ├── Gate C ──> Done
```

Each role:

1. **Reads** the sections of previous roles as an **immutable input contract**.
2. **Writes** only its own sections.
3. **Does not modify** other roles' sections without the amendment process.

This ensures predictability, traceability, and the ability to work on different atoms in parallel.

## Related Documents

- [Atom Anatomy](atom-anatomy.md)
- [Gate Validation](ru/gate-validation.md)
- [Amendments](ru/amendments.md)
