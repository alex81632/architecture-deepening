# Frontend State Example

This example shows how to identify a shallow boundary in a frontend feature
where a screen coordinates fetching, normalization, filtering, selection,
optimistic updates, and error handling directly in the component tree.

The goal is to deepen the module boundary around a feature workflow, not to
blindly introduce stores, hooks, or patterns.

## Context

Imagine a React feature with these files:

```text
src/
  features/issues/IssuesPage.tsx
  features/issues/useIssuesQuery.ts
  features/issues/useIssueFilters.ts
  features/issues/useIssueSelection.ts
  features/issues/useBulkAssign.ts
  features/issues/issueMappers.ts
  features/issues/IssueTable.tsx
  features/issues/IssueFilters.tsx
  features/issues/types.ts
```

Current behavior:
- fetch issues
- normalize API data
- keep filter state
- derive filtered issues
- manage selected rows
- perform bulk assign
- optimistically update visible rows
- show success and error states

The UI works, but most workflow coordination is still visible in the page.

## Current code shape

### `IssuesPage.tsx`

```tsx
import { useMemo } from "react";
import { useIssuesQuery } from "./useIssuesQuery";
import { useIssueFilters } from "./useIssueFilters";
import { useIssueSelection } from "./useIssueSelection";
import { useBulkAssign } from "./useBulkAssign";
import { mapIssueDtoToRow } from "./issueMappers";
import { IssueFilters } from "./IssueFilters";
import { IssueTable } from "./IssueTable";

export function IssuesPage() {
  const { data, isLoading, error, refetch } = useIssuesQuery();
  const { filters, setFilter, clearFilters } = useIssueFilters();
  const { selectedIds, toggleSelection, clearSelection } = useIssueSelection();
  const { bulkAssign, isSubmitting } = useBulkAssign();

  const rows = useMemo(() => {
    return (data ?? []).map(mapIssueDtoToRow);
  }, [data]);

  const filteredRows = useMemo(() => {
    return rows.filter((row) => {
      if (filters.status !== "all" && row.status !== filters.status) {
        return false;
      }

      if (filters.assignee === "unassigned" && row.assigneeId) {
        return false;
      }

      if (
        filters.search &&
        !row.title.toLowerCase().includes(filters.search.toLowerCase())
      ) {
        return false;
      }

      return true;
    });
  }, [rows, filters]);

  async function handleBulkAssign(userId: string) {
    const ids = Array.from(selectedIds);

    if (ids.length === 0) {
      return;
    }

    await bulkAssign({
      issueIds: ids,
      assigneeId: userId,
    });

    clearSelection();
    await refetch();
  }

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Could not load issues.</div>;
  }

  return (
    <>
      <IssueFilters
        filters={filters}
        onChange={setFilter}
        onClear={clearFilters}
      />

      <IssueTable
        rows={filteredRows}
        selectedIds={selectedIds}
        onToggleSelection={toggleSelection}
        onBulkAssign={handleBulkAssign}
        isSubmitting={isSubmitting}
      />
    </>
  );
}
```

## Why this boundary is shallow

There are hooks and helper modules, but the page still owns the feature
workflow.

The page must understand:
- loading and error states
- mapping and filtering sequence
- selection lifecycle
- mutation behavior
- post-mutation refresh
- which state resets after which action

This is still shallow because:
- complexity is not hidden, only distributed
- the page is an orchestration layer
- the public feature boundary is weak
- tests likely need to mount the page and know too much about internals

## Example output from the skill

## Architecture Deepening Review

### 1. Scope
- Area analyzed: issues listing and bulk assignment feature
- Goal: reduce page-level orchestration and strengthen feature boundary
- Constraints:
  - preserve existing UI behavior
  - avoid unnecessary global state
  - keep compatibility with current query/mutation layer
- Assumptions:
  - this feature will evolve with more actions and filters

### 2. Candidate Opportunities

#### Candidate 1: Deepen issues screen controller boundary
- Cluster:
  - `IssuesPage.tsx`
  - `useIssuesQuery.ts`
  - `useIssueFilters.ts`
  - `useIssueSelection.ts`
  - `useBulkAssign.ts`
  - `issueMappers.ts`
- Current responsibility split:
  - fetching, derivation, UI state, mutation flow, and refresh logic are spread
    across hooks and component code
- Why coupled:
  - they represent one feature workflow
- Why shallow today:
  - the page coordinates everything explicitly
- Expected deeper boundary:
  - one feature-level module owns issue list state and actions
- Dependency category:
  - remote but owned
- Scores:
  - Coupling pain: 4
  - Change frequency: 4
  - Test pain: 4
  - Interface shallowness: 5
  - Migration risk: 2
  - Expected payoff: 5
- Priority: high
- Evidence:
  - the page wires together many hooks and must know action sequencing

#### Candidate 2: Deepen filtering and derivation boundary
- Cluster:
  - `useIssueFilters.ts`
  - `issueMappers.ts`
  - local `useMemo` filtering logic
- Current responsibility split:
  - filter state and derived rows are separated awkwardly
- Why coupled:
  - derivation depends on filter semantics
- Why shallow today:
  - filtering logic leaks into the page
- Expected deeper boundary:
  - one issues view-model derivation boundary
- Dependency category:
  - in-process
- Scores:
  - Coupling pain: 3
  - Change frequency: 4
  - Test pain: 3
  - Interface shallowness: 4
  - Migration risk: 1
  - Expected payoff: 3
- Priority: medium
- Evidence:
  - filtering logic is inline and page-specific

#### Candidate 3: Deepen bulk action mutation boundary
- Cluster:
  - `useBulkAssign.ts`
  - page-level `handleBulkAssign`
- Current responsibility split:
  - mutation and post-success state handling are separate
- Why coupled:
  - successful mutation requires follow-up local behavior
- Why shallow today:
  - the page must remember to clear selection and refetch
- Expected deeper boundary:
  - one bulk assign action with local completion semantics
- Dependency category:
  - remote but owned
- Scores:
  - Coupling pain: 3
  - Change frequency: 3
  - Test pain: 3
  - Interface shallowness: 4
  - Migration risk: 1
  - Expected payoff: 3
- Priority: medium
- Evidence:
  - mutation is not fully owned by the feature action boundary

### 3. Chosen Candidate
- Name: deepen issues screen controller boundary
- Why chosen:
  - captures the largest amount of leaked orchestration
  - most likely to simplify future feature growth
  - creates a stronger UI-facing feature boundary

### 4. Problem Framing
- Responsibilities to consolidate:
  - data fetching state
  - row mapping and filtering
  - selection state
  - bulk action flow
  - post-action refresh and cleanup
- Complexity to hide:
  - how derived state, selection, and server mutation interact
- Constraints:
  - component composition should remain simple
  - avoid introducing heavy global architecture
- Invariants:
  - filtered rows reflect current filters and latest data
  - selection clears after successful bulk assign
  - bulk assign should not require page to remember cleanup steps
- Problem sketch:
  - the page currently acts as a workflow coordinator for a whole feature that
    could instead expose one feature-specific boundary

### 5. Interface Options

#### Option A — Minimal Interface
- Interface sketch:

```tsx
type UseIssuesScreenResult = {
  view: {
    rows: IssueRow[];
    isLoading: boolean;
    error: string | null;
    filters: IssueFiltersState;
    selectedIds: Set<string>;
    isSubmitting: boolean;
  };
  actions: {
    setFilter: (name: FilterName, value: string) => void;
    clearFilters: () => void;
    toggleSelection: (id: string) => void;
    bulkAssign: (userId: string) => Promise<void>;
    retry: () => Promise<void>;
  };
};

function useIssuesScreen(): UseIssuesScreenResult;
```

- Example usage:

```tsx
export function IssuesPage() {
  const { view, actions } = useIssuesScreen();

  if (view.isLoading) {
    return <div>Loading...</div>;
  }

  if (view.error) {
    return <div>{view.error}</div>;
  }

  return (
    <>
      <IssueFilters
        filters={view.filters}
        onChange={actions.setFilter}
        onClear={actions.clearFilters}
      />

      <IssueTable
        rows={view.rows}
        selectedIds={view.selectedIds}
        onToggleSelection={actions.toggleSelection}
        onBulkAssign={actions.bulkAssign}
        isSubmitting={view.isSubmitting}
      />
    </>
  );
}
```

- Hidden complexity:
  - fetch + map + filter + selection + mutation + refresh coordination
- Dependency strategy:
  - reuse existing hooks internally at first
- Trade-offs:
  - simple page API
  - feature hook may grow if too many unrelated concerns are added
- Test impact:
  - easier to test feature behavior at one hook boundary
- Overengineering risk:
  - low

#### Option B — Flexible Interface
- Interface sketch:

```tsx
type UseIssuesControllerOptions = {
  initialFilters?: Partial<IssueFiltersState>;
  autoRefreshOnBulkAssign?: boolean;
  selectionMode?: "multi" | "single";
};

function useIssuesController(
  options?: UseIssuesControllerOptions,
): UseIssuesControllerResult;
```

- Example usage:

```tsx
const { view, actions } = useIssuesController({
  autoRefreshOnBulkAssign: true,
  selectionMode: "multi",
});
```

- Hidden complexity:
  - same general consolidation, but with configurable behavior
- Dependency strategy:
  - useful if multiple screens share the same controller with meaningful
    differences
- Trade-offs:
  - more reusable
  - may expose variation too early
- Test impact:
  - more cases to test
- Overengineering risk:
  - medium

#### Option C — Common-case Optimized
- Interface sketch:

```tsx
function useDefaultIssuesScreen(): DefaultIssuesScreenResult;
function useEmbeddedIssuesPicker(): EmbeddedIssuesPickerResult;
```

- Example usage:

```tsx
const screen = useDefaultIssuesScreen();
```

- Hidden complexity:
  - separate common page workflow from alternate embedded workflow
- Dependency strategy:
  - good if there are truly distinct presentation contexts
- Trade-offs:
  - strong clarity when two stable use cases already exist
  - premature if there is only one real screen today
- Test impact:
  - very clear contracts if both workflows are real
- Overengineering risk:
  - medium

### 6. Recommendation
- Recommended option: Option A — Minimal Interface
- Why:
  - strongest simplification with lowest risk
  - keeps component tree readable
  - hides feature orchestration without imposing global state
  - can be implemented by composing existing hooks internally
- Rejected alternatives:
  - Option B is only justified if multiple screens truly need configurability
  - Option C should wait until a second stable workflow exists
- What not to do:
  - do not introduce a global store just because the page is busy
  - do not split the page into more tiny hooks if the orchestration remains
    exposed

### 7. Migration Plan
1. add characterization tests for current page behavior
2. create `useIssuesScreen()` that internally delegates to existing hooks
3. move filtering and derived rows into the new feature hook
4. move bulk assign cleanup and refresh semantics into the feature hook
5. migrate `IssuesPage` to use only `view` and `actions`
6. remove redundant orchestration from page component
7. later, evaluate whether some old hooks are now implementation details

### 8. Verification
- Characterization tests:
  - loading state
  - error state
  - filtering behavior
  - selection behavior
  - bulk assign success path
- Boundary tests:
  - `useIssuesScreen()` derives rows correctly
  - selection clears after successful bulk assign
  - retry and refresh behavior stay consistent
- Regression checks:
  - visible UI behavior remains unchanged
- Integration checks:
  - query and mutation calls still happen correctly
- Operational checks:
  - none expected beyond normal frontend monitoring

### 9. Success Criteria
- the page stops coordinating the feature workflow manually
- callers render a feature state rather than build one
- feature tests can target one boundary
- future actions such as bulk close or bulk label can fit the same owner cleanly

## Why this is a good example

This example is useful because frontend code often looks “modular” while still
being shallow.

Many teams split code into:
- hooks
- selectors
- mappers
- mutations
- components

But if the screen still owns the real workflow, complexity has only been spread
out, not hidden.

A deeper feature boundary can simplify the page dramatically without requiring:
- Redux
- a state machine framework
- a global architecture rewrite

That is exactly the kind of deepening this skill should prefer.