# The Realm Stack

InterRealm's Realm Stack is a layered architecture for building decentralized, type-safe agent networks. Each layer builds on the previous one, from foundational connectivity to advanced multi-agent coordination.

---

## Layer 1: Foundation — Discovery & Connection

The base layer handles automatic capability discovery and authentication.

**How it works:**
1. Connect to a gateway with your auth credentials
2. Capabilities automatically resolve based on your realm's permissions
3. Static TypeScript types are generated for all available capabilities
4. Zero dependency hell. No credential management. Clean abstractions.

---

## Layer 2: Services — Basic Agent Building Blocks

Build individual agents using decorators and dependency injection.

### Simple Service Agent

```typescript
import { Service, Capability } from '@realmtrix/sdk';

@Capability({
  name: 'pricing',
  version: '1.0.0'
})
@Service({ name: 'PriceCheck' })
export class PriceCheckAgent {
  async call(request: PriceRequest): Promise<PriceResponse> {
    const price = await this.checkPrice(request.productId);
    return {
      productId: request.productId,
      price,
      currency: 'USD',
      timestamp: new Date()
    };
  }

  private async checkPrice(productId: string): Promise<number> {
    // Your pricing logic here
    return 99.99;
  }
}
```

**What happens:**
- The `@Service` decorator registers this as a callable service
- Request/response types are automatically inferred
- The gateway makes this available to other realms based on routing policies

---

## Layer 3: Events — Reactive Agent Communication

Subscribe to events across the network with type-safe handlers.

### Event Subscriber Agent

```typescript
import { Service, EventSubscriber, OnEvent } from '@realmtrix/sdk';

@Capability({ name: 'notifications', version: '1.0.0' })
@Service({ name: 'AlertService' })
export class AlertService {
  @OnEvent('inventory.low')
  async handleLowInventory(event: InventoryLowEvent): Promise<void> {
    console.log(`🚨 Low inventory alert: ${event.productId}`);

    // Send notification
    await this.sendAlert({
      type: 'inventory',
      severity: event.severity,
      message: `Product ${event.productId} is low: ${event.quantity} remaining`
    });
  }

  private async sendAlert(alert: Alert): Promise<void> {
    // Your notification logic here
  }
}
```

**What happens:**
- The `@OnEvent` decorator automatically subscribes to the event topic
- When any agent publishes `inventory.low`, this handler is called
- Type-safe event payloads (TypeScript knows the shape of `InventoryLowEvent`)

---

## Layer 4: Loops — Multi-Agent Coordination

Loops enable collaborative decision-making through a three-phase pattern:

1. **Recruitment** — Broadcast a task, agents self-select based on capability
2. **Execution** — Participating agents work in parallel
3. **Aggregation** — Results are combined using a strategy (vote, merge, consensus)

### Loop Participant Agent

```typescript
import { Service, Loop, LoopParticipant } from '@realmtrix/sdk';

@Capability({ name: 'pricing', version: '1.0.0' })
@Service({ name: 'CompetitivePricing' })
@LoopParticipant('price-aggregation')
export class CompetitivePricingAgent {
  // This agent can participate in price-aggregation loops
  async participate(request: PriceRequest): Promise<PriceOffer> {
    const myPrice = await this.calculatePrice(request);

    return {
      agentId: this.id,
      price: myPrice,
      confidence: 0.95,
      source: 'competitive-analysis'
    };
  }
}
```

**What happens:**
- Agent declares it can participate in `price-aggregation` loops
- When another agent initiates a `price-aggregation` loop, this agent can join
- Provides its contribution (price offer) during execution phase

### Loop Initiator Agent

```typescript
import { Service, LoopInitiator, Inject } from '@realmtrix/sdk';

@Capability({ name: 'pricing', version: '1.0.0' })
@Service({ name: 'PriceOptimizer' })
export class PriceOptimizerAgent {
  @Inject()
  private loops!: LoopInitiator;

  async call(request: OptimizeRequest): Promise<OptimizedPrice> {
    // Initiate a loop to gather price recommendations
    const result = await this.loops.initiate({
      loopName: 'price-aggregation',
      capability: 'pricing',
      input: {
        productId: request.productId,
        targetMargin: request.targetMargin
      },
      options: {
        minParticipants: 3,
        recruitmentTimeout: 5000,
        executionTimeout: 30000,
        aggregation: 'average'
      }
    });

    return {
      recommendedPrice: result.aggregatedPrice,
      participantCount: result.participants.length,
      confidence: result.confidence
    };
  }
}
```

**What happens:**
- Loop is broadcast to all connected agents with `pricing` capability
- 3+ agents join and provide their price recommendations
- Results are averaged (aggregation strategy)
- Final price returned with audit trail of all participants

---

## Layer 5: LoopStacks — Complex Workflows

LoopStacks chain multiple loops together for recursive, multi-phase coordination.

### LoopStack Example

```typescript
import { Service, LoopStack, Inject } from '@realmtrix/sdk';

@Capability({ name: 'workflows', version: '1.0.0' })
@Service({ name: 'ContentGenerationWorkflow' })
export class ContentGenerationWorkflow {
  @Inject('content', 'ContentGenerationStack')
  private contentStack!: LoopStack<ContentRequest, ContentResult>;

  async call(request: ContentRequest): Promise<ContentResult> {
    // Initiate the entire stack
    // This triggers multiple loops in sequence:
    // 1. Outline generation (multiple agents collaborate)
    // 2. Content writing (distributed across sections)
    // 3. Review and editing (peer review loop)
    // 4. Final aggregation

    return await this.contentStack.initiate({
      topic: request.topic,
      targetLength: request.targetLength,
      style: request.style
    });
  }
}
```

**What happens:**
- The LoopStack is defined in a capability YAML
- Each loop in the stack recruits agents, executes in parallel, aggregates
- Results flow from one loop to the next
- Agents can spawn sub-loops recursively (nested coordination)
- Full audit trail of every decision at every level

---

## Workflow Patterns

### Simple Workflow (No Loops)

```typescript
import { Service } from '@realmtrix/sdk';

@Capability({ name: 'workflows', version: '1.0.0' })
@Service({ name: 'SimpleWorkflow' })
export class SimpleWorkflow {
  async call(request: WorkflowRequest): Promise<WorkflowResult> {
    // Traditional sequential workflow
    const step1 = await this.stepOne(request);
    const step2 = await this.stepTwo(step1);
    const step3 = await this.stepThree(step2);

    return step3;
  }

  private async stepOne(req: WorkflowRequest) { /* ... */ }
  private async stepTwo(data: any) { /* ... */ }
  private async stepThree(data: any) { /* ... */ }
}
```

**When to use:**
- Simple, linear workflows
- Single-agent processing
- No need for multi-agent coordination

### Agentic Workflow (With LoopStacks)

```typescript
import { Service, LoopStack, Inject } from '@realmtrix/sdk';

@Capability({ name: 'workflows', version: '1.0.0' })
@Service({ name: 'AgenticWorkflow' })
export class AgenticWorkflow {
  @Inject('content', 'ContentGenerationStack')
  private contentStack!: LoopStack<ContentRequest, ContentResult>;

  @Inject('review', 'PeerReviewStack')
  private reviewStack!: LoopStack<ReviewRequest, ReviewResult>;

  async call(request: WorkflowRequest): Promise<WorkflowResult> {
    // Step 1: Generate content using multi-agent loop
    const content = await this.contentStack.initiate({
      topic: request.topic,
      requirements: request.requirements
    });

    // Step 2: Review content using peer review loop
    const review = await this.reviewStack.initiate({
      content: content.text,
      criteria: request.qualityCriteria
    });

    // Step 3: Finalize based on review
    return {
      finalContent: review.approved ? content.text : content.revisedText,
      reviewScore: review.score,
      participantCount: content.agentCount + review.reviewerCount
    };
  }
}
```

**When to use:**
- Complex, multi-phase workflows
- Need consensus or aggregation at each step
- Recursive coordination (loops spawning sub-loops)
- Distributed decision-making

---

## Layer 6: Integration — Dependency Injection

The SDK provides automatic dependency injection for InterRealm services:

```typescript
import { Service, Inject } from '@realmtrix/sdk';

@Service({ name: 'OrderProcessor' })
export class OrderProcessor {
  // Inject a service from another capability
  @Inject('pricing', 'PriceCheck')
  private priceService!: PriceCheckService;

  // Inject an event publisher
  @Inject()
  private events!: EventPublisher;

  // Inject a loop initiator
  @Inject()
  private loops!: LoopInitiator;

  async call(request: OrderRequest): Promise<OrderResult> {
    // Use injected service
    const price = await this.priceService.call({
      productId: request.productId
    });

    // Use injected event publisher
    await this.events.publish('order.created', {
      orderId: request.orderId,
      price: price.price
    });

    return { orderId: request.orderId, total: price.price };
  }
}
```

---

## The Complete Stack

```
┌─────────────────────────────────────────┐
│   Layer 6: Dependency Injection         │  Cross-cutting integration
├─────────────────────────────────────────┤
│   Layer 5: LoopStacks                   │  Multi-phase workflows
├─────────────────────────────────────────┤
│   Layer 4: Loops                        │  Multi-agent coordination
├─────────────────────────────────────────┤
│   Layer 3: Events                       │  Reactive communication
├─────────────────────────────────────────┤
│   Layer 2: Services                     │  Basic agent building blocks
├─────────────────────────────────────────┤
│   Layer 1: Discovery & Connection       │  Foundation
└─────────────────────────────────────────┘
```

**Build type-safe agents. Connect to the InterRealm network. Zero dependency hell.**
