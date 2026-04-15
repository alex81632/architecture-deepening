# Backend Service Example

This example shows how to identify a shallow boundary in a backend workflow
where order creation is spread across handlers, validators, repositories,
pricing helpers, and notification code.

The goal is not to impose a framework. The goal is to consolidate a fragmented
workflow into a deeper module with a smaller, clearer interface.

## Context

Imagine a backend service with these files:

```text
src/
  http/create-order-handler.ts
  services/order-validator.ts
  services/price-calculator.ts
  services/inventory-checker.ts
  repositories/order-repository.ts
  repositories/inventory-repository.ts
  integrations/payment-gateway.ts
  integrations/email-service.ts
  domain/order.ts
  tests/create-order-handler.test.ts
```

Current flow:
- the HTTP handler validates input
- calls the price calculator
- checks inventory
- calls payment gateway
- saves the order
- sends confirmation email

The system works, but each caller must know too much about the workflow.

## Current code shape

### `create-order-handler.ts`

```ts
import { validateOrderInput } from "../services/order-validator";
import { calculatePrice } from "../services/price-calculator";
import { checkInventory, reserveInventory } from "../services/inventory-checker";
import { orderRepository } from "../repositories/order-repository";
import { paymentGateway } from "../integrations/payment-gateway";
import { emailService } from "../integrations/email-service";

export async function createOrderHandler(req, res) {
  const input = req.body;

  const validation = validateOrderInput(input);

  if (!validation.ok) {
    return res.status(400).json({
      error: validation.error,
    });
  }

  const price = await calculatePrice(input.items, input.customerId);
  const available = await checkInventory(input.items);

  if (!available) {
    return res.status(409).json({
      error: "inventory_unavailable",
    });
  }

  const paymentResult = await paymentGateway.charge({
    customerId: input.customerId,
    amount: price.total,
    currency: "USD",
  });

  if (!paymentResult.ok) {
    return res.status(402).json({
      error: "payment_failed",
    });
  }

  await reserveInventory(input.items);

  const order = await orderRepository.insert({
    customerId: input.customerId,
    items: input.items,
    total: price.total,
    paymentId: paymentResult.paymentId,
    status: "confirmed",
  });

  await emailService.sendOrderConfirmation({
    to: input.email,
    orderId: order.id,
  });

  return res.status(201).json(order);
}
```

## Why this boundary is shallow

Even though there are many modules, the caller still owns the real workflow.

The handler must know:
- validation sequencing
- pricing flow
- inventory policy
- payment timing
- persistence timing
- email timing
- error mapping
- response mapping

The modules are small, but the workflow is still visible from the outside.

This is a classic shallow design:
- many small pieces
- low hidden complexity
- orchestration spread outward
- weak ownership of business invariants

## Example output from the skill

## Architecture Deepening Review

### 1. Scope
- Area analyzed: order creation workflow
- Goal: reduce call-site orchestration and improve testability
- Constraints:
  - preserve HTTP contract
  - preserve payment and email integrations
  - avoid big-bang rewrite
- Assumptions:
  - order creation is a common workflow
  - more order-related features will likely be added later

### 2. Candidate Opportunities

#### Candidate 1: Deepen order creation workflow
- Cluster:
  - `create-order-handler.ts`
  - `order-validator.ts`
  - `price-calculator.ts`
  - `inventory-checker.ts`
  - `order-repository.ts`
  - `payment-gateway.ts`
  - `email-service.ts`
- Current responsibility split:
  - validation, pricing, inventory, payment, persistence, and notification are
    all separate
- Why coupled:
  - they participate in one business transaction
- Why shallow today:
  - the public caller still coordinates the full business flow
- Expected deeper boundary:
  - one module owns order creation from request to confirmed order result
- Dependency category:
  - remote but owned plus true external
- Scores:
  - Coupling pain: 5
  - Change frequency: 4
  - Test pain: 4
  - Interface shallowness: 5
  - Migration risk: 3
  - Expected payoff: 5
- Priority: high
- Evidence:
  - the handler manually coordinates all major steps
  - tests likely need many mocks or setup points
  - any policy change touches several modules

#### Candidate 2: Deepen inventory reservation boundary
- Cluster:
  - `inventory-checker.ts`
  - `inventory-repository.ts`
- Current responsibility split:
  - availability and reservation are separate
- Why coupled:
  - they represent one inventory decision flow
- Why shallow today:
  - callers must manage the sequence
- Expected deeper boundary:
  - one inventory allocation boundary
- Dependency category:
  - remote but owned
- Scores:
  - Coupling pain: 3
  - Change frequency: 3
  - Test pain: 3
  - Interface shallowness: 4
  - Migration risk: 2
  - Expected payoff: 3
- Priority: medium
- Evidence:
  - availability and reservation likely belong together

#### Candidate 3: Deepen payment result handling
- Cluster:
  - `payment-gateway.ts`
  - handler-specific error mapping
- Current responsibility split:
  - payment charge and payment error mapping are separate
- Why coupled:
  - payment semantics affect business outcome
- Why shallow today:
  - raw payment outcomes leak upward
- Expected deeper boundary:
  - local payment attempt result mapped into domain language
- Dependency category:
  - true external
- Scores:
  - Coupling pain: 2
  - Change frequency: 3
  - Test pain: 3
  - Interface shallowness: 3
  - Migration risk: 2
  - Expected payoff: 2
- Priority: low
- Evidence:
  - useful cleanup, but narrower than the full workflow issue

### 3. Chosen Candidate
- Name: deepen order creation workflow
- Why chosen:
  - highest payoff
  - strongest evidence of shallow orchestration
  - improves both changeability and testing

### 4. Problem Framing
- Responsibilities to consolidate:
  - input-to-order-confirmation workflow
  - inventory and payment sequencing
  - persistence timing
  - domain error mapping
  - side-effect coordination
- Complexity to hide:
  - pricing, inventory, payment, and confirmation sequencing
- Constraints:
  - HTTP layer should remain thin
  - existing dependencies should be reused initially
  - payment failure semantics must remain stable
- Invariants:
  - do not create confirmed order without successful payment
  - do not confirm order when inventory is unavailable
  - reserve inventory only during successful creation path
- Problem sketch:
  - one business request currently requires a caller to coordinate many internal
    steps that belong to the same business capability

### 5. Interface Options

#### Option A — Minimal Interface
- Interface sketch:

```ts
type CreateOrderInput = {
  customerId: string;
  email: string;
  items: Array<{
    sku: string;
    quantity: number;
  }>;
};

type CreateOrderResult =
  | {
      ok: true;
      order: Order;
    }
  | {
      ok: false;
      reason:
        | "invalid_input"
        | "inventory_unavailable"
        | "payment_failed";
      message: string;
    };

interface OrderCreator {
  create(input: CreateOrderInput): Promise<CreateOrderResult>;
}
```

- Example usage:

```ts
const result = await orderCreator.create(req.body);

if (!result.ok) {
  return mapOrderCreationFailure(res, result);
}

return res.status(201).json(result.order);
```

- Hidden complexity:
  - validation
  - pricing
  - inventory check and reservation
  - payment orchestration
  - persistence
  - confirmation email trigger
- Dependency strategy:
  - keep existing concrete dependencies inside the module at first
- Trade-offs:
  - simplest for callers
  - may need internal branching as workflows grow
- Test impact:
  - boundary tests become straightforward
  - many coordination tests move inside one module
- Overengineering risk:
  - low

#### Option B — Flexible Interface
- Interface sketch:

```ts
type CreateOrderCommand = {
  customerId: string;
  email: string;
  items: Array<{
    sku: string;
    quantity: number;
  }>;
  paymentMode?: "charge_now" | "authorize_only";
  notificationMode?: "send_confirmation" | "silent";
};

interface OrderApplicationService {
  execute(command: CreateOrderCommand): Promise<OrderCreationOutcome>;
}
```

- Example usage:

```ts
const outcome = await orderApplicationService.execute({
  ...req.body,
  paymentMode: "charge_now",
  notificationMode: "send_confirmation",
});
```

- Hidden complexity:
  - same as Option A, but exposes more configuration dimensions
- Dependency strategy:
  - supports future variations more directly
- Trade-offs:
  - more extensible
  - risks leaking workflow choices outward too early
- Test impact:
  - still testable at boundary
  - more cases to cover
- Overengineering risk:
  - medium

#### Option C — Common-case Optimized
- Interface sketch:

```ts
interface Orders {
  placeStandardOrder(input: PlaceStandardOrderInput): Promise<PlaceOrderResult>;
  placeManualReviewOrder(
    input: PlaceManualReviewOrderInput,
  ): Promise<PlaceOrderResult>;
}
```

- Example usage:

```ts
const result = await orders.placeStandardOrder(req.body);
```

- Hidden complexity:
  - standard path becomes very easy
  - edge cases can have dedicated entry points
- Dependency strategy:
  - separate common path from exceptional flow
- Trade-offs:
  - excellent if workflow variants are already real
  - premature if there is currently only one meaningful path
- Test impact:
  - strong clarity if separate paths truly matter
- Overengineering risk:
  - medium

### 6. Recommendation
- Recommended option: Option A — Minimal Interface
- Why:
  - best matches current evidence
  - removes orchestration from handler
  - keeps public contract small
  - avoids exposing speculative workflow variation
- Rejected alternatives:
  - Option B exposes flexibility not yet proven necessary
  - Option C is good later if multiple stable order paths emerge
- What not to do:
  - do not create a generic `OrderService` with many knobs
  - do not split the current flow into even more “helper” modules

### 7. Migration Plan
1. add characterization tests around current order creation behavior
2. introduce `OrderCreator.create(input)` behind the existing handler
3. implement `OrderCreator` initially by delegating to current modules
4. migrate the handler to use only `OrderCreator`
5. move error mapping into domain-friendly result values
6. progressively pull validation, pricing, and sequencing inward
7. remove now-redundant direct calls from handlers
8. delete obsolete orchestration helpers only after all callers migrate

### 8. Verification
- Characterization tests:
  - successful order creation
  - validation failure
  - inventory unavailable
  - payment failure
- Boundary tests:
  - `OrderCreator.create` returns correct result for each scenario
- Regression checks:
  - API response shape remains unchanged
- Integration checks:
  - payment integration still invoked correctly
  - persistence still stores expected order state
- Operational checks:
  - monitor payment failure rate and order success rate during rollout

### 9. Success Criteria
- handlers no longer orchestrate business transaction steps
- callers request an outcome rather than a sequence
- tests target `OrderCreator` boundary rather than helper interactions
- future order policy changes happen mostly in one place

## Why this is a good example

This is a strong deepening opportunity because:
- the workflow has clear business meaning
- many technical steps form one coherent responsibility
- the caller currently knows too much
- a smaller interface can hide substantial internal complexity
- migration can happen incrementally