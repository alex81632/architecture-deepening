# Output Templates

Use these templates when the user asks for a specific artifact. Adapt them to
the repository, the scope analyzed, and the depth requested.

Prefer the smallest useful artifact. Expand only when the task genuinely needs
more structure.

## Template: Architecture Deepening Review

```text
## Architecture Deepening Review

### 1. Scope
- Area analyzed:
- Goal:
- Constraints:
- Assumptions:

### 2. Candidate Opportunities

Repeat this block only as many times as needed.

#### Candidate: <name>
- Cluster:
- Current responsibility split:
- Why coupled:
- Why shallow today:
- Expected deeper boundary:
- Dependency category:
- Scores: optional
- Priority:
- Evidence:

### 3. Chosen Candidate
- Name:
- Why chosen:

### 4. Problem Framing
- Responsibilities to consolidate:
- Complexity to hide:
- Constraints:
- Invariants:
- Problem sketch:

### 5. Interface Options

Include this section only when comparing alternatives.

Repeat the option block only as many times as needed.

#### Option: <name>
- Interface sketch:
- Example usage:
- Hidden complexity:
- Dependency strategy:
- Trade-offs:
- Test impact:
- Overengineering risk:

### 6. Recommendation
- Recommended option:
- Why:
- Rejected alternatives:
- What not to do:

### 7. Migration Plan
1. 
2. 
3. 
4. 

### 8. Verification
- Characterization tests:
- Boundary tests:
- Regression checks:
- Integration checks:
- Operational checks:

### 9. Success Criteria
- Simpler call sites:
- Complexity moved inward:
- Testability improvement:
- Evidence the module is deeper:
```

## Template: Short Recommendation

Use this when the user wants a concise answer instead of a full review.

```text
## Recommended Architecture Deepening

### Problem
- 

### Why the current boundary is shallow
- 

### Recommended deeper boundary
- 

### Proposed interface
- 

### Why this is better
- 

### Migration plan
1. 
2. 
3. 

### Verification
- 
```

## Template: RFC

```text
# RFC: Deepen <module or workflow name>

## Status
- Proposed

## Context
- Area:
- Authors:
- Date:
- Related issues:

## Problem Statement
Describe the current pain in concrete terms.
- Which workflow or concept is fragmented?
- Why is the current boundary shallow?
- What changes are currently difficult?
- What code evidence supports this?

## Goals
- 
- 
- 

## Non-goals
- 
- 

## Current Design
- Main modules involved:
- Current call flow:
- Known friction points:
- Testing pain:
- Operational constraints:

## Design Options

Include only the options that matter. One option is fine if the direction is
obvious. Use two or three only when there is a real trade-off.

### Option: <name>
- Interface:
- Benefits:
- Drawbacks:
- Migration shape:

## Recommended Design
- Chosen option:
- Why this option:
- Why not the others:
- Key invariants:
- Dependency strategy:

## Migration Plan
1. 
2. 
3. 
4. 
5. 

## Verification Plan
- Characterization tests:
- Boundary tests:
- Integration checks:
- Operational validation:

## Risks
- 
- 

## Mitigations
- 
- 

## Rollback Plan
- 

## Open Questions
- 
```

## Template: GitHub Issue

```text
# Deepen <module or workflow name>

## Summary
The current implementation of <area> is fragmented across <modules/files>. This
creates a shallow boundary that exposes too much orchestration to callers and
makes changes harder than necessary.

## Current Pain
- 
- 
- 

## Evidence
- Repeated call sequence:
- Coupled files/modules:
- Test friction:
- Change friction:

## Proposed Direction
Introduce a deeper boundary around <responsibility> that owns:
- 
- 
- 

Possible interface sketch:
- 

## Expected Benefits
- simpler call sites
- clearer ownership
- fewer cross-module edits
- stronger boundary tests

## Migration Plan
- [ ] add characterization tests
- [ ] introduce new boundary
- [ ] migrate first caller
- [ ] verify behavior
- [ ] migrate remaining callers
- [ ] remove obsolete code

## Verification
- [ ] characterization tests pass
- [ ] new boundary tests added
- [ ] regressions checked
- [ ] operational checks completed if needed

## Notes
- Avoid turning this into a generic service layer
- Keep the interface outcome-oriented
- Prefer incremental migration over rewrite
```

## Template: Migration Checklist

```text
## Migration Checklist for <module or workflow name>

### Preparation
- [ ] confirm scope
- [ ] identify current callers
- [ ] identify constraints and compatibility requirements
- [ ] add characterization tests for current behavior

### New Boundary
- [ ] define responsibility of the new module
- [ ] define invariants
- [ ] choose recommended interface
- [ ] create boundary tests

### Introduction
- [ ] implement the new boundary using existing internals where possible
- [ ] add temporary adapter or shim if needed
- [ ] migrate one low-risk caller first
- [ ] compare behavior with old path

### Rollout
- [ ] migrate remaining callers incrementally
- [ ] remove duplicated orchestration from call sites
- [ ] move hidden complexity inward
- [ ] update tests to target the new boundary

### Cleanup
- [ ] delete obsolete entry points
- [ ] delete temporary adapters
- [ ] remove redundant tests tied to internal coordination
- [ ] update documentation

### Validation
- [ ] regression suite passes
- [ ] integration checks pass
- [ ] performance impact acceptable
- [ ] rollback plan documented
```

## Template: Implementation Plan

```text
## Implementation Plan

### Goal
- 

### Chosen Boundary
- 

### Public Interface
- 

### Internals to Consolidate
- 
- 
- 

### Step-by-step
1. 
2. 
3. 
4. 
5. 

### Risks
- 
- 

### Verification
- 
- 

### Done When
- 
- 
```