---
title: InterRealm Capabilities
description: Dynamic, decentralized capabilities for building distributed AI systems with zero dependency hell and type-safe integration.
sidebar_position: 1
slug: /
---

# InterRealm Capabilities

## Overview

InterRealm works on a foundation of **dynamic, decentralized capabilities** that are automatically discovered and shared as clients connect to the network. Realms provide the governance layer that controls access to these capabilities, while our SDK and CLI make agent development effortless.

### How It Works

1. **Connect to a gateway** with your auth credentials
2. **Capabilities automatically resolve** based on your realm's permissions
3. **Static TypeScript types are generated** for all available capabilities
4. **Build type-safe agents** using decorators and dependency injection

**The result:** Zero dependency hell. No credential management. Clean abstractions. Type-safe integration with the entire InterRealm network.

---

## Key Benefits

### ✅ Zero Agent Dependencies
Your agents don't depend on specific implementations — they depend on **capability interfaces**. If a better implementation comes online, your agents automatically use it. No code changes needed.

### ✅ No Credential Hell
API keys, auth tokens, and secrets are managed by the gateway. Your application code **never sees credentials**. Deploy the same code to dev, staging, and production — the gateway handles environment-specific credentials automatically.

### ✅ Dynamic Discovery
As new realms connect and publish capabilities, they become available to your agents instantly. The CLI auto-generates updated types, and your IDE shows you what's newly available.

### ✅ Type-Safe Integration
Every service, event, and loop is strongly typed. Your IDE provides autocomplete, and TypeScript catches integration errors at compile time.

---

## The Capability Model

InterRealm capabilities are defined using a simple, open protocol. Each capability can expose:

- **Services** — Request/response RPC calls
- **Events** — Pub/sub event streams
- **Loops** — AI coordination patterns (recruitment → execution → aggregation)
- **LoopStacks** — Sequences of loops for complex workflows

### Publishing Capabilities

When you build an agent, the CLI automatically publishes its capabilities to the network:

```bash
# Your agent declares what it provides
realm-cli publish ./my-agent

# The gateway registers your capabilities
# Other realms can now discover and use them
```

---

## Services: Request/Response Pattern

```typescript
import { Service, Capability } from '@realmtrix/sdk';

@Capability({ name: 'pricing', version: '1.0.0' })
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
    return 99.99;
  }
}
```

---

## CLI Workflow

1. **Initialize your project**
```bash
realm-cli init my-agent --template typescript
```

2. **Login to gateway**
```bash
realm-cli login --gateway wss://my-company.interrealm.io --realm my-company.production
```

3. **Develop your agent**
```typescript
import { Service, Inject } from '@realmtrix/sdk';
import { PriceCheckService } from '../generated/capabilities/pricing';

@Service({ name: 'MyAgent' })
export class MyAgent {
  @Inject('pricing', 'PriceCheck')
  private pricing!: PriceCheckService;

  async call(req: MyRequest): Promise<MyResponse> {
    const result = await this.pricing.call({ productId: req.productId });
    return { price: result.price };
  }
}
```

4. **Publish capabilities**
```bash
realm-cli publish ./my-agent
```

5. **Sync updates**
```bash
realm-cli sync
```

---

## Summary

**InterRealm capabilities** enable:
- **Services** — Simple RPC between agents
- **Events** — Pub/sub for decoupled communication
- **Loops** — Multi-agent coordination with verifiable consensus
- **LoopStacks** — Recursive workflows for agentic intelligence

**Build distributed AI systems as easily as calling functions.**
