---
name: architecture-deepening
description: Analyze a codebase or feature area to find shallow modules, fragmented orchestration, duplicated call patterns, and tightly coupled concepts; then recommend deeper interfaces, compare design options when useful, and outline an incremental migration plan. Use when improving architecture, reducing coupling, consolidating business logic, increasing testability, or making a system easier to navigate and change.
---

# Architecture Deepening

Identify shallow boundaries and recommend deeper modules that hide more
complexity behind smaller, clearer interfaces.

Optimize for:
- lower cognitive load at call sites
- fewer cross-module hops to understand one concept
- boundary tests over internal seam tests
- incremental migration over big rewrites

Do not default to generic architecture patterns. Recommend layers, ports,
repositories, services, or interface extraction only when the codebase evidence
supports them.

## When to use

Use this skill when the user wants to:
- improve architecture in an existing codebase
- reduce coupling in a feature, service, or workflow
- consolidate fragmented business logic
- turn repeated orchestration into a deeper module
- improve testability through better public boundaries
- produce a review, migration plan, RFC, or implementation direction

Do not use it for:
- greenfield architecture from scratch
- style-driven rewrites
- cosmetic file reorganization
- speculative abstractions without real code evidence

## Inputs to infer or confirm

Infer what you can and state assumptions explicitly:
- scope: package, folder, feature, service, workflow, or bounded context
- goal: testability, maintainability, simpler call sites, safer change, or
  easier navigation
- constraints: compatibility, deadlines, framework limits, performance, or
  deployment model
- desired output: quick review, deep review, interface options, migration plan,
  RFC, issue, or implementation help

If the scope is too broad, narrow it. Prefer one painful subsystem over the
whole repository.

## Core principles

1. Analyze real code before proposing solutions.
2. Prefer evidence over dogma.
3. Deepen modules; do not just split files.
4. Reduce visible complexity at call sites.
5. Avoid premature abstraction.
6. Respect patterns that already work in the repository.
7. Prefer boundary tests over tests of internal helpers.
8. Be opinionated and actionable.

## Working modes

Adapt depth to the request:
- Quick review: identify the strongest opportunities and recommend one direction
- Deep review: compare candidates, explore alternatives, and include migration
- Artifact mode: produce an RFC, issue draft, checklist, or implementation plan

Default to the lightest mode that answers the request well.

## Process

### 1. Explore the target area

Follow the concept across files, callers, tests, and boundaries. Look for:
- one concept spread across many files
- repeated call sequences across callers
- orchestration duplicated at call sites
- shared types leaking across layers
- business rules split across helpers, validators, handlers, and services
- tests forced into internal helpers because the public boundary is weak

Use friction as a signal, but ground each claim in code evidence.

Load `HEURISTICS.md` only if you need more signals or counterexamples.

### 2. Produce candidates

If the scope is broad, identify 2 to 5 candidates. If the scope is already
narrow, one strong candidate may be enough.

For each candidate, include only what is needed to justify the recommendation:
- name
- cluster: files, modules, or concepts involved
- why the boundary is shallow or fragmented
- expected deeper boundary
- likely payoff
- migration risk
- evidence

Optional: add lightweight scoring if it helps disambiguate. Avoid false
precision. If you score, explain what drove the ranking.

Ask the user to choose only when the trade-off is real. Otherwise, pick the
strongest candidate and say why.

### 3. Frame the problem

Before proposing a design, make the problem concrete:
- what the module should own
- what complexity should move inward
- what constraints or invariants must hold
- what dependencies it must coordinate

Use a rough sketch only when it clarifies the design space.

Load `REFERENCE.md` only if you need dependency or testing guidance.

### 4. Design interfaces

Propose as many options as the problem deserves:
- 1 option if the direction is obvious
- 2 options when there is a meaningful trade-off
- 3 options only when the design space is genuinely contested

When comparing options, make them materially different. Typical shapes:
- minimal interface
- flexible interface
- common-case optimized interface
- ports-and-adapters oriented boundary when the dependency shape justifies it

For each option, include:
- interface sketch
- example usage
- hidden complexity
- trade-offs
- test impact

### 5. Recommend one direction

Give a strong recommendation, not a neutral menu. Be explicit about:
- why it is the best trade-off here
- what it simplifies for callers
- what complexity it hides successfully
- why weaker options lose in this context

### 6. Plan migration

Include a migration plan when the user asks for design, refactoring, or
implementation help.

Prefer small, reversible steps:
- starting point
- sequence of safe refactors
- adapters or compatibility shims if needed
- which call sites to migrate first
- how to preserve behavior
- rollback notes

### 7. Define verification

Include verification guidance when proposing change:
- characterization tests first when behavior is unclear
- boundary tests for the new interface
- regression or integration checks where needed
- performance or operational checks if relevant

Prefer tests of externally observable behavior.

### 8. Produce the requested artifact

Return only the artifact the user asked for. Use `OUTPUT_TEMPLATES.md` only
when the user wants a formal review, RFC, issue, or checklist.

## Decision rules

Prefer deepening when:
- multiple callers repeat the same orchestration
- the public contract can shrink while implementation grows inward
- complexity can be hidden without losing domain clarity
- a boundary test could replace many brittle internal tests
- understanding one workflow requires too many file jumps

Avoid deepening when:
- there is only one trivial caller and little growth pressure
- the abstraction would mostly rename existing complexity
- the new module would become a vague manager or service
- the interface would grow broader than current needs
- migration cost is high and payoff is speculative

## Dependency categories

Classify the chosen candidate only if it affects the design:
- In-process: pure computation or in-memory coordination
- Local-substitutable: practical local stand-in exists for tests
- Remote but owned: network or process boundary you control
- True external: third-party boundary

Use `REFERENCE.md` only when deeper dependency guidance is needed.

## Output defaults

Default to a concise response. Expand only when the user asks or the task
requires it.

Preferred shape for most answers:
1. Scope and assumptions
2. Best candidate or top candidates
3. Recommendation
4. Migration notes
5. Verification

Use a longer structure only for deep reviews or formal artifacts.

## Anti-patterns to avoid

Do not:
- recommend architecture frameworks without evidence from the code
- extract interfaces just for indirection
- split responsibilities so far that orchestration leaks into callers
- preserve shallow modules just because they are small
- optimize for theoretical purity over changeability
- create abstractions for one weak use case
- propose a big rewrite when incremental deepening is possible
- suggest tests that lock in internal implementation details

## Success criteria

A strong recommendation should usually lead to:
- fewer concepts exposed at the call site
- fewer hops to understand one workflow
- clearer ownership of business rules
- a smaller and more stable public interface
- tests concentrated at meaningful boundaries
- easier future changes in one place instead of many

## Additional resources

Load these only when needed:
- `HEURISTICS.md`: signals of shallow modules and overengineering
- `REFERENCE.md`: dependency handling, testing strategy, RFC guidance
- `OUTPUT_TEMPLATES.md`: review, RFC, issue, and checklist templates
