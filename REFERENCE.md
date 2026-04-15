# Reference

This document provides supporting guidance for architecture deepening work:
- deep vs. shallow modules
- dependency categories
- testing strategy
- migration strategy
- RFC guidance

Use this file when the main skill needs more specific decision support.

## Deep vs. shallow modules

A deep module exposes a small interface relative to the complexity it hides.

A shallow module exposes an interface that is not much simpler than the work
required to implement or understand it. Shallow modules often increase the
number of boundaries without reducing the total complexity visible to callers.

### What “deeper” usually means in practice

A deeper design often does one or more of the following:

- owns a full workflow rather than one step in a workflow
- enforces invariants in one place
- hides sequencing, retries, validation, mapping, and error handling
- reduces the number of concepts callers must coordinate
- provides a domain-oriented entry point instead of plumbing-oriented steps
- lets tests focus on behavior at the boundary

Deeper does not mean:
- larger class
- more generic abstraction
- more layers
- more indirection
- more patterns

The goal is to reduce visible complexity, not to maximize structure.

## Choosing a boundary

A good boundary usually has:
- one coherent responsibility
- stable semantics recognizable in domain language
- meaningful internal complexity to hide
- a public interface shaped around outcomes, not implementation steps
- clear ownership of invariants and error behavior

A bad boundary often:
- groups unrelated helpers
- mirrors technical layers instead of domain behavior
- forces callers to orchestrate several steps
- leaks intermediate data structures widely
- exists mainly for organizational aesthetics

## Dependency categories

Classify dependencies before proposing the interface.

### 1. In-process

Examples:
- pure business logic
- parsing
- transformation
- in-memory coordination

Guidance:
- easiest category for direct deepening
- prefer concrete modules over unnecessary indirection
- tests can often stay fast and realistic without mocks

### 2. Local-substitutable

Examples:
- filesystem with temporary directory
- local database with test instance
- queue with local harness
- deterministic clock wrapper

Guidance:
- use real or realistic stand-ins in tests when practical
- deepen around the workflow, not just around the dependency
- avoid interfaces that exist only to support mocking if realistic testing is
  available

### 3. Remote but owned

Examples:
- internal service over HTTP or gRPC
- another deployable you control
- internal eventing boundary

Guidance:
- define the behavioral contract clearly
- consider ports/adapters if the boundary has multiple implementations or if
  protocol details are noisy
- keep transport details from leaking into core orchestration
- test the local boundary thoroughly and use integration tests for protocol
  correctness

### 4. True external

Examples:
- third-party API
- payment provider
- SaaS webhook source

Guidance:
- isolate the external specifics at the edge
- map external errors and payloads into local concepts
- keep fakes or mocks at the outer boundary only
- do not let third-party concepts leak through the codebase unless necessary

## Interface design guidance

When proposing interface options, aim to express:
- desired outcome
- ownership of sequencing
- ownership of invariants
- error semantics
- dependency handling strategy

### Prefer outcome-oriented entry points

Prefer:
- `createSubscription(...)`
- `processRefund(...)`
- `publishInvoice(...)`

Over:
- `validate(...)`, then `map(...)`, then `persist(...)`, then `notify(...)`

Callers should ask for an outcome. The module should decide how to achieve it.

### Avoid over-generalization

A generic interface is worse than a narrow one when:
- it exposes dimensions of variation callers do not need
- it makes common cases harder
- it shifts complexity outward to be “flexible”
- it invents future use cases

Start from real workflows.

### Make common cases easy

If most callers do the same thing, optimize for that path.
Edge cases can be supported through:
- optional parameters with restraint
- secondary entry points
- internal branching
- specialized advanced APIs if truly needed

Do not burden every caller with rare concerns.

## Testing strategy

Deepening should generally improve testing by moving tests upward to a more
meaningful boundary.

### Characterization tests

Before refactoring a risky area, capture current behavior.

Use characterization tests when:
- behavior is not fully documented
- logic is spread across multiple modules
- regressions are likely during consolidation

Test:
- externally observable outputs
- side effects
- error behavior
- ordering if it matters semantically

Avoid testing incidental implementation details.

### Boundary tests

After introducing a deeper module, prefer tests at its public boundary.

A strong boundary test typically:
- calls the public entry point
- sets up only relevant dependencies
- asserts domain outcomes and side effects
- ignores internal call sequence unless sequence is part of the contract

Boundary tests should replace many lower-level coordination tests.

### Integration tests

Use integration tests for:
- protocol correctness
- persistence correctness
- interaction with remote or external systems
- operational assumptions that unit tests cannot validate

Do not push all confidence into integration tests. Most logic should still be
validated near the module boundary.

### Mocking guidance

Mocking is most appropriate:
- at true external boundaries
- at expensive or nondeterministic edges
- when protocol behavior must be simulated carefully

Avoid deep mock trees inside the system. If many mocks are required, the
boundary is often too shallow.

## Migration patterns

Prefer reversible migrations that preserve behavior.

### Strangler pattern for internal modules

Use when replacing fragmented orchestration with a deeper module.

Steps:
1. identify a representative call path
2. add characterization tests
3. implement the new boundary
4. migrate one caller
5. verify behavior
6. migrate remaining callers gradually
7. delete obsolete plumbing

### Facade-first migration

Use when many callers depend on noisy steps.

Steps:
1. create a new facade over the existing internals
2. keep internals mostly unchanged at first
3. migrate callers to the facade
4. move logic inward over time
5. shrink or delete old entry points

This often reduces risk because behavior changes and interface changes are
separated.

### Adapter or compatibility shim

Use when:
- old and new interfaces must coexist for a while
- public API stability matters
- migration spans multiple teams

Guidance:
- keep adapters thin and temporary
- document deletion criteria
- avoid letting compatibility shape the final design permanently

### Branch-by-abstraction

Use carefully when:
- a major dependency or workflow must be swapped gradually
- multiple implementations must coexist briefly

Guidance:
- only introduce the abstraction if it clearly serves the migration
- remove migration scaffolding once the transition is complete

## Deletion criteria

Old modules or APIs can usually be removed when:
- all callers use the new boundary
- characterization and regression tests pass
- observability shows no production issues if relevant
- compatibility windows have expired
- the old path no longer carries unique behavior

Deletion should be explicit in the plan, not left as implied cleanup.

## Operational considerations

For production-sensitive changes, consider:
- feature flags
- canary rollout
- metrics or logs to compare old vs. new path
- idempotency
- latency changes
- error-rate changes
- rollback trigger and rollback procedure

Architecture changes are not just code changes when they affect runtime paths.

## RFC guidance

When the user asks for an RFC, include:

- problem statement
- current pain with evidence
- goals
- non-goals
- candidate designs
- recommended design
- migration plan
- verification plan
- risks and mitigations
- open questions

Keep the RFC concrete and grounded in current code, not abstract principles.

## Smell-to-action mapping

Use this table as quick guidance.

### Smell: repeated orchestration across callers
Likely action:
- consolidate into one deeper workflow boundary

### Smell: many helper modules for one concept
Likely action:
- regroup behavior around one owner

### Smell: tests require many mocks
Likely action:
- move behavior behind a stronger boundary and test there

### Smell: interface has many flags and sequencing rules
Likely action:
- redesign around outcomes or split into clearer entry points

### Smell: third-party details leak through core code
Likely action:
- isolate the external boundary and map into domain concepts

### Smell: every change touches several neighboring files
Likely action:
- identify missing owner and consolidate invariants

## Final recommendation quality bar

A recommendation is high quality when it:
- names the real responsibility clearly
- reduces complexity visible to callers
- improves testability at a meaningful boundary
- fits repository constraints
- includes incremental migration
- avoids speculative abstraction
- explains why this boundary is better than the alternatives