# Heuristics

Use these heuristics to identify shallow modules, promising deepening
opportunities, and cases where abstraction would likely be overengineering.

These heuristics are not laws. Use them as evidence-gathering tools, not as
dogma.

## Signals of shallow modules

A module is likely shallow when its interface exposes almost as much complexity
as its implementation hides.

Common signals:

- the public API has many parameters, modes, or flags
- callers need to understand internal sequencing to use it correctly
- the module mostly forwards work to other modules without owning meaningful
  behavior
- the interface is large, but the implementation is thin
- the module exists mainly as a naming wrapper around a few calls
- callers must coordinate retries, validation, error mapping, or state
  transitions themselves
- the module leaks intermediate types that spread across the codebase
- usage requires reading several helper functions to understand one workflow
- tests focus on internal helpers because the public boundary is too weak
- changes to one business rule require edits in multiple neighboring modules

## Signals of fragmented orchestration

These often indicate an opportunity for a deeper module that owns a workflow.

Look for:

- repeated multi-step call sequences across several handlers or components
- validation in one file, transformation in another, persistence elsewhere, and
  error handling duplicated at the edges
- similar orchestration with minor variations repeated across use cases
- the same domain concept represented by several helper layers
- every new feature duplicates “the usual flow” with small modifications
- call sites assembling the same objects in the same order
- logic spread across controller, service, repository, mapper, and validator
  with no obvious owner

## Signals of low-cohesion boundaries

A boundary may be badly chosen when:

- methods in the same module do not seem related by one central purpose
- a module has several entry points that never benefit from shared hidden logic
- the module name is vague, such as `Manager`, `Utils`, `Helpers`, or
  `Service`, and the implementation confirms the vagueness
- the boundary groups code by technical layer rather than by a coherent domain
  capability
- moving one rule requires touching multiple modules because ownership is
  unclear

## Signals of excessive interface complexity

Large interfaces are not always bad, but they are a warning sign.

Pay attention when:

- many methods are rarely used
- multiple methods must be called in the right order
- valid call sequences are implied rather than enforced
- boolean flags or option bags control major behavioral branches
- different callers use disjoint subsets of the API
- the API mirrors internal implementation concepts instead of domain concepts
- onboarding requires memorizing “how to use it safely”

## Signals that a deeper module could help

A deeper module is promising when it can hide meaningful complexity while
presenting a smaller, more stable, more natural interface.

Good signs:

- many callers want the same outcome but currently perform the steps manually
- the workflow has stable business semantics even if internal details vary
- error handling, retries, idempotency, caching, or transactional behavior can
  be owned centrally
- domain invariants can be enforced in one place
- multiple implementation details can move behind one boundary
- boundary tests could replace many brittle lower-level tests
- understanding one use case currently requires too many file jumps
- the proposed public API can be described in domain language rather than
  plumbing language

## Signals that a repeated pattern should become a module

Promote repeated logic into a deeper module when most of the following are true:

- the pattern appears in at least 3 places, or in fewer places but with clear
  growth pressure
- the sequence is semantically important, not just syntactically similar
- variations are narrow enough to fit one coherent abstraction
- callers benefit from a simpler public contract
- centralizing the behavior improves testability or reliability
- the new module would own a complete responsibility, not just a step

Do not promote patterns that are only superficially similar.

## Signals of test pain caused by shallow boundaries

Testing pain often reveals poor module boundaries.

Look for:

- many mocks are needed to test one behavior
- tests assert internal calls instead of externally observable outcomes
- small implementation changes break large numbers of tests
- tests are forced into low-level helpers because higher-level boundaries are
  awkward
- fixtures duplicate orchestration that production callers also duplicate
- integration tests are bloated because ownership is unclear

## Signals that deepening may improve AI or human navigation

This skill should also optimize for navigability.

Promising signs:

- one feature requires opening many files before the main behavior becomes clear
- names describe implementation steps instead of domain outcomes
- the “main path” of a workflow is hidden among plumbing
- call stacks bounce between thin wrappers
- the same concept is represented differently across modules
- a new contributor would struggle to answer “where does this behavior live?”

## Overengineering red flags

Do not deepen or abstract just because you can.

Be cautious when:

- there is only one simple caller and no realistic growth path
- the abstraction would mainly rename existing concepts
- the new module would need many configuration knobs immediately
- the interface becomes more generic than any current use case requires
- the proposal introduces several layers at once
- migration cost is high, but current pain is minor
- the abstraction depends on hypothetical future flexibility
- the result would hide important domain semantics rather than complexity
- the proposal creates a “god service” that coordinates everything vaguely
- the new boundary would be harder to explain than the current code

## Questions to ask while analyzing a candidate

Use these questions to sharpen diagnosis:

- What is the real responsibility trying to emerge here?
- What do callers repeatedly have to know that they should not need to know?
- Which details could move inward without making the design opaque?
- Is the interface smaller than the hidden complexity?
- Could one boundary test replace several low-level tests?
- Does this module own behavior or merely route calls?
- Are we consolidating a concept or just moving files around?
- Would a new contributor find the workflow faster after this change?
- What invariants should this boundary enforce?
- What would make this recommendation obviously too abstract?

## Candidate scoring guidance

Use a 1 to 5 scale for each dimension.

### Coupling pain

- 1: mostly isolated
- 2: occasional coupling annoyance
- 3: coupling visible in normal changes
- 4: frequent cross-module edits required
- 5: heavy coupling is a recurring source of bugs or delay

### Change frequency

- 1: rarely changes
- 2: changes infrequently
- 3: changes regularly
- 4: changes often
- 5: changes constantly or blocks feature work

### Test pain

- 1: boundary already easy to test
- 2: manageable test friction
- 3: tests are awkward or overly internal
- 4: tests require many mocks or brittle setup
- 5: test pain is severe and slows development

### Interface shallowness

- 1: already fairly deep
- 2: somewhat shallow
- 3: visibly shallow
- 4: very shallow
- 5: mostly a thin wrapper with leaked complexity

### Migration risk

- 1: low-risk and local
- 2: mostly local with minor coordination
- 3: moderate risk
- 4: broad coordination required
- 5: high-risk due to public API, rollout, or operational constraints

### Expected payoff

- 1: little practical benefit
- 2: minor cleanup
- 3: noticeable improvement
- 4: strong improvement to daily work
- 5: major simplification or risk reduction

## Good recommendation checklist

Before finalizing a recommendation, verify that most of the following are true:

- the chosen boundary owns a coherent responsibility
- the public interface gets smaller or clearer
- repeated orchestration moves inward
- important invariants become enforceable at one point
- tests can move toward the module boundary
- migration can happen incrementally
- the recommendation fits the codebase style and constraints
- the payoff is concrete, not aesthetic