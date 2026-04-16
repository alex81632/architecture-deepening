# architecture-deepening

A Claude Code / Cursor skill for improving software architecture by identifying
shallow modules, proposing deeper interfaces, and generating incremental
migration plans.

This skill is inspired by ideas from _A Philosophy of Software Design_,
especially deep vs. shallow modules, but it is designed to be practical for
day-to-day work in real codebases.

Instead of defaulting to generic architecture advice, it focuses on:
- finding fragmented workflows
- detecting shallow module boundaries
- proposing deeper interfaces
- comparing design options when useful
- planning safe, incremental refactors
- improving testability and navigability

## What this skill does

Given a repository, subsystem, or feature area, the skill will:

1. explore the code and identify architectural friction
2. find one or more candidates for deepening shallow modules
3. prioritize or recommend the strongest candidate
4. compare interface options when there is a real trade-off
5. recommend a direction
6. create an incremental migration plan when change is proposed
7. define verification and testing guidance

The skill is intended to help with:
- backend workflows
- frontend feature boundaries
- services and monoliths
- codebases with too much orchestration at call sites
- systems that are hard to navigate or change safely

## Philosophy

A deep module has a small interface relative to the complexity it hides.

A shallow module has an interface that exposes almost as much complexity as the
module itself.

This skill tries to improve architecture by reducing visible complexity.

That usually means:
- callers ask for outcomes instead of orchestrating steps
- business rules move behind clearer boundaries
- repeated sequencing is consolidated
- tests move toward meaningful public interfaces
- migration happens incrementally instead of through rewrites

This skill does _not_ assume that better architecture means:
- more layers
- more interfaces
- more indirection
- more patterns
- “clean architecture” by default

## Repository structure

```text
architecture-deepening/
├── README.md
├── SKILL.md
├── HEURISTICS.md
├── REFERENCE.md
├── OUTPUT_TEMPLATES.md
└── examples/
    ├── backend-service-example.md
    └── frontend-state-example.md
```

For installation, copy the files into a skill directory such as:

```text
.cursor/
└── skills/
    └── architecture-deepening/
```

Many Cursor setups are also compatible with `.claude/skills/`.

## Installation

### Claude Code

Create this directory in your repository:

```text
.claude/skills/architecture-deepening/
```

Then copy these files into it:
- `SKILL.md`
- `HEURISTICS.md`
- `REFERENCE.md`
- `OUTPUT_TEMPLATES.md`
- `examples/backend-service-example.md`
- `examples/frontend-state-example.md`

### Cursor

Create this directory:

```text
.cursor/skills/architecture-deepening/
```

Then copy the same files there.

If your Cursor setup already reads `.claude/skills/`, you can keep a single
copy in `.claude/skills/`.

## When to use this skill

Use this skill when you want to:
- improve the architecture of an existing codebase
- refactor a feature or service boundary
- reduce coupling
- consolidate business logic
- make code easier to test
- make a system easier to understand
- turn repeated orchestration into a deeper module
- write an RFC or migration plan for an architectural refactor

It works especially well when:
- one workflow is spread across many small files
- handlers or components know too much
- the same sequence is repeated in multiple places
- tests rely on too many mocks
- changing one behavior requires edits across multiple modules

## When not to use it

This skill is not intended for:
- greenfield architecture from scratch
- enforcing a specific architecture style across all projects
- cosmetic file reorganization
- speculative abstractions without real code evidence
- “make this clean architecture” type requests

If the current pain is small and the payoff is unclear, the skill should prefer
doing less.

## How it works

The skill follows an adaptive workflow:

### 1. Explore
It explores the codebase and looks for:
- fragmented orchestration
- shallow boundaries
- repeated call patterns
- leaked complexity
- weak test boundaries

### 2. Generate candidates
If the scope is broad, it identifies 2 to 5 candidates. If the scope is
already narrow, one strong candidate may be enough.

Scoring is optional. When used, it is meant to clarify trade-offs rather than
create false precision.

### 3. Frame the chosen problem
It identifies:
- the responsibility that should be consolidated
- the complexity that should move inward
- the invariants that must be preserved
- the constraints that shape the design

### 4. Design multiple interfaces
It proposes only as many options as the problem deserves:
- 1 option if the direction is obvious
- 2 options when there is a meaningful trade-off
- 3 options only when the design space is genuinely contested

Typical option shapes include:
- minimal interface
- flexible interface
- common-case optimized interface

### 5. Recommend one direction
It compares the options and makes an opinionated recommendation.

### 6. Plan migration
It generates a safe, incremental migration path.

### 7. Define verification
It suggests:
- characterization tests
- boundary tests
- regression checks
- integration checks when needed

### 8. Produce the requested artifact
It returns the smallest useful artifact by default and expands into a formal
review, RFC, issue, checklist, or implementation plan only when requested.

## Working modes

The skill adapts its depth to the request:
- `Quick review`: identify the strongest opportunities and recommend one direction
- `Deep review`: compare candidates, explore alternatives, and include migration
- `Artifact mode`: produce an RFC, issue draft, checklist, or implementation plan

## Output shape

By default, the skill stays concise and expands only when needed.

Most answers use a compact shape:

- scope and assumptions
- best candidate or top candidates
- recommendation
- migration notes
- verification strategy

Longer, sectioned output is reserved for deep reviews and formal artifacts.

## Example prompts

### General architecture review

```text
Use the architecture-deepening skill to review this codebase and find the best
opportunities to deepen shallow modules.
```

### Scoped review

```text
Use architecture-deepening on the billing workflow in
src/services/billing/.
Focus on reducing orchestration in handlers and improving testability.
```

### Quick review

```text
Use architecture-deepening on src/payments/refunds.
Give me a quick review: identify the strongest deepening opportunity, recommend
one direction, and keep the answer concise.
```

### Frontend feature

```text
Use architecture-deepening on the issues screen feature.
I want to simplify page-level orchestration and make the feature easier to
extend.
```

### Backend service boundary

```text
Use architecture-deepening on the order creation flow.
Please identify candidates, compare interface options, and recommend one with
an incremental migration plan.
```

### RFC generation

```text
Use architecture-deepening on the notification pipeline and produce an RFC for
the strongest deepening opportunity.
```

### GitHub issue generation

```text
Use architecture-deepening on src/features/inbox and write a GitHub issue for
the highest-priority architectural refactor.
```

## What makes a good candidate

This skill tends to work best when it finds areas where:
- one business concept is spread across multiple modules
- callers must coordinate several steps manually
- helpers expose internal sequencing
- shared types leak through the system
- tests are forced into internal seams
- one behavior requires too many file jumps to understand

## What makes a poor candidate

It should avoid recommending deepening when:
- there is only one trivial caller
- the new abstraction would mostly rename existing complexity
- the payoff is speculative
- the interface would become broader than the actual need
- migration cost is high and current pain is low

## Included files

### `SKILL.md`
The main workflow and decision rules.

### `HEURISTICS.md`
Concrete signals of:
- shallow modules
- fragmented orchestration
- overengineering risk
- good deepening opportunities

### `REFERENCE.md`
Supporting guidance for:
- deep vs. shallow modules
- dependency categories
- testing strategy
- migration patterns
- RFC guidance

### `OUTPUT_TEMPLATES.md`
Templates for:
- architecture review
- short recommendation
- RFC
- GitHub issue
- migration checklist
- implementation plan

### `examples/backend-service-example.md`
A backend deep review example where order creation is fragmented across handlers and
services.

### `examples/frontend-state-example.md`
A frontend deep review example where a page coordinates too much feature state and workflow
logic directly.

## Design goals

This skill is optimized for:

- practical recommendations over abstract principles
- evidence from the actual codebase
- deeper modules, not just smaller files
- opinionated output
- incremental migration
- better test boundaries
- better human and AI navigability

## Non-goals

This skill is not trying to:
- replace architectural judgment
- enforce one universal architecture style
- maximize abstraction
- produce idealized rewrites
- recommend layers for their own sake

## Tips for best results

- give the skill a focused scope if the repository is large
- specify your main goal:
  - testability
  - maintainability
  - simpler call sites
  - safer change
  - clearer ownership
- mention constraints such as:
  - backward compatibility
  - deadlines
  - no breaking API changes
  - no framework migration
- ask for a specific output if needed:
  - quick review
  - deep review
  - review
  - RFC
  - issue
  - migration plan

Good prompt example:

```text
Use architecture-deepening on src/payments/refunds.
Focus on reducing cross-module orchestration and improving boundary tests.
Do not suggest a broad rewrite. I want the top 3 candidates and a recommended
migration plan for one of them.
```

## Why this exists

Many refactoring tools are good at:
- spotting duplication
- suggesting smaller functions
- proposing common architecture patterns

But that is not always enough.

Real architecture pain often comes from:
- fragmented ownership
- thin wrappers
- noisy call paths
- too many boundaries with too little hidden complexity

This skill exists to help identify and improve those situations.