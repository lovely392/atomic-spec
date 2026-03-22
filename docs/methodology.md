**English** | [Русский](ru/methodology.md)

# Atomic Spec Methodology

## Philosophy

Atomic Spec unifies four approaches into a single methodology for working with specifications:

- **DDD (Domain-Driven Design)** — the specification is built around the domain, not around technical decisions. Domain rules come first; implementation comes second.
- **TDD (Test-Driven Development)** — acceptance criteria and the test plan are written before or in parallel with implementation, not after.
- **Use-Case-Driven** — each atom describes a specific use case with actors, actions, and expected outcomes.
- **Requirements-Driven** — requirements are captured explicitly and traced through the entire pipeline from Intent to test coverage.

## Core Idea: One File = One Unit of Knowledge

An atom is the minimal indivisible unit of specification. A single Markdown file contains everything you need to know about a particular feature, rule, or constraint:

- **Business context** (Intent, Domain Rules)
- **Acceptance criteria** (Acceptance Criteria)
- **Technical decisions** (Tech Spec, Platform API)
- **Test planning** (Test Plan, Coverage Matrix)
- **Decision history** (Decision Log, Open Questions)

There is no need to switch between Confluence, Jira, Google Docs, and Slack to understand what is going on. Everything is in one file, under version control.

### Benefits of This Approach

- **Traceability**: from a business requirement to a specific test — a single chain in a single file.
- **Autonomy**: each atom is self-contained. It can be read, understood, and reviewed in isolation.
- **Versionability**: Git stores the full change history of every atom.
- **Reviewability**: a PR on an atom is a PR on the specification, code, and tests all at once.

## Progressive Disclosure Principle

An atom is not filled in all at once but incrementally, as it moves through the pipeline:

```
Analyst                   Developer                Tester
─────────────────────     ─────────────────────    ─────────────────────
Intent                    Tech Spec                Test Plan
Domain Rules              Platform API             Platform Tests
Acceptance Criteria       Implementation Notes     Coverage Matrix
DMT, Constraints          Code                     AC Review
Open Questions
Decision Log
```

At each stage only the sections belonging to the current role are filled in. Previous sections are not modified without explicit agreement (amendment).

This principle ensures:

- **Focus**: each role works with its own slice of information.
- **Contract**: the boundary between roles is clearly defined by gates.
- **Quality**: each stage is validated before handoff to the next role.

## Three Status Axes

The state of an atom is described by three independent axes:

### 1. Semantic Status (`status`)

Reflects the maturity of the specification as a document:

| Value        | Description                                                        |
|-------------|--------------------------------------------------------------------|
| `draft`     | The atom is in progress, has not yet passed all gates              |
| `active`    | The atom is complete and serves as the current specification       |
| `deprecated`| The atom has been replaced by another (see `supersedes` on the new atom) |

### 2. Implementation Status (`implementation`)

Reflects the state of the code:

| Value         | Description                                    |
|--------------|------------------------------------------------|
| `none`       | Code has not been written yet                  |
| `in-progress`| Implementation has started                     |
| `done`       | Implementation is complete, passed Gate B      |

### 3. Verification Status (`verification`)

Reflects the state of testing:

| Value         | Description                                    |
|--------------|------------------------------------------------|
| `none`       | Testing has not started                        |
| `in-progress`| Test plan created, tests are being written     |
| `passed`     | All tests pass, Coverage Matrix is filled in   |
| `failed`     | Tests revealed issues, rework required         |

### Axis Combinations

The three axes are independent. A typical sequence of atom states:

```
draft / none / none           — Analyst is working
draft / in-progress / none    — Developer is implementing
draft / done / in-progress    — Tester is verifying
active / done / passed        — Atom is complete
```

Atypical but valid combinations:

```
active / done / failed        — Regression: tests failed on an active atom
deprecated / done / passed    — Atom replaced by a new one, but old code still works
```

## Methodology Test

The Atomic Spec methodology passes a simple test:

> **Can you reconstruct the full project history by reading only `git log`?**

If the answer is yes, the methodology is being applied correctly. This is ensured by:

- **Atom-first**: specification atom first, code second. Every commit is tied to a specific atom.
- **One atom = one PR**: changes are atomic and isolated.
- **Change type in the commit**: the commit message makes it clear what changed (parameter, rule, flow, model, boundary).
- **Gates as milestones**: passing a gate is recorded in the history.
- **Decision Log**: the rationale behind decisions is preserved in the atom itself.

Thus the repository becomes the single source of truth — not only for code but also for all requirements, decisions, and their justifications.

## Related Documents

- [Quick Start](getting-started.md)
- [Atom Anatomy](atom-anatomy.md)
- [Roles and Pipeline](roles-and-pipeline.md)
- [Gate Validation](ru/gate-validation.md)
- [Change Types](ru/change-types.md)
- [Git Conventions](ru/git-conventions.md)
- [Amendments](ru/amendments.md)
