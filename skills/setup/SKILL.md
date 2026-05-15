---
name: setup-paid
description: Skill for integrating Paid - the all-in-one business engine for AI agents handling pricing, billing, and usage tracking.
---

# Paid Integration Guide

**Always consult [docs.paid.ai](https://docs.paid.ai) for code examples and latest API.**

Paid is the all-in-one business engine for AI agents — handling usage-based pricing, outcome-based pricing, credits, subscriptions, margins, billing, and renewals via Stripe.

---

## Quick Reference

### Environment Variables
- `PAID_API_KEY` - API key (required). Run `paid init` to authenticate and store locally

### Installation
```bash
npm install @paid-ai/paid-node    # Node.js
pip install paid-python            # Python
```

### Core Methods

| Method | Purpose |
|--------|---------|
| `initializeTracing` | Initialize tracing with API key (call once at startup) |
| `paidAutoInstrument` | Auto-instrument supported AI SDKs |
| `trace` | Wrap AI calls with customer/product context |
| `signal` | Record usage events |

---

## Core Config Options

| Option | Notes |
|--------|-------|
| `PAID_API_KEY` | Required. Run `paid init` to authenticate |

---

## Integration Patterns

### Autoinstrument → Trace → Signal

Always follow this order for tracked actions:

```typescript
import { initializeTracing, paidAutoInstrument, trace, signal } from "@paid-ai/paid-node";

initializeTracing(process.env.PAID_API_KEY!);
paidAutoInstrument();

await trace({
  externalCustomerId: "<CUSTOMER_ID>",
  externalProductId: "<PRODUCT_ID>",
}, async () => {
  const response = await openai.chat.completions.create({ ... });
  signal("chat_completion", true);
});
```

```python
from paid.tracing import initialize_tracing, paid_autoinstrument, paid_tracing, signal

initialize_tracing()
paid_autoinstrument()

@paid_tracing("<CUSTOMER_ID>", external_product_id="<PRODUCT_ID>")
def run_agent(user_input: str):
    response = openai.chat.completions.create(...)
    signal(event_name="chat_completion", enable_cost_tracing=True)
```

### Manual Tracing

Use when autoinstrumentation is not compatible:

```typescript
import { initializeTracing, trace, signal } from "@paid-ai/paid-node";

initializeTracing(process.env.PAID_API_KEY!);

await trace({
  externalCustomerId: "<CUSTOMER_ID>",
  externalProductId: "<PRODUCT_ID>",
}, async () => {
  const result = await customAiCall(input);
  signal("query_processed", true, { /* optional metadata */ });
});
```

```python
from paid.tracing import initialize_tracing, paid_tracing, signal

initialize_tracing()

@paid_tracing("<CUSTOMER_ID>", external_product_id="<PRODUCT_ID>")
def run_agent(user_input: str):
    result = custom_ai_call(user_input)
    signal(event_name="query_processed", enable_cost_tracing=True)
```

### Python Async Context Manager

```python
async with paid_tracing("<CUSTOMER_ID>", external_product_id="<PRODUCT_ID>"):
    result = await your_async_agent_function(user_input)
```

---

## Autoinstrumentation Compatibility

| AI SDK | Supported | Notes |
|--------|-----------|-------|
| `openai` | Yes | Node.js and Python |
| `anthropic` | Yes | Node.js and Python |
| Vercel AI SDK (`ai`) | Yes | Underlying provider instrumented |
| AWS Bedrock (`boto3`) | Yes | Supported |
| LangChain | Yes | Python |
| Gemini | Yes | Python |
| Custom HTTP | No | Use manual tracing |
| Unknown SDK | No | Use manual tracing |

---

## Environment Detection

| Signal | Environment |
|--------|-------------|
| `pyproject.toml`, `requirements.txt`, `.py` files | Python |
| `package.json` (no `"type": "module"`) | TypeScript/Node (CJS) |
| `package.json` with `"type": "module"` | TypeScript/Node (ESM) |
| `next.config.*` | Next.js (always use ESM path) |
| `bun.lockb` | Bun runtime |

### Package Manager Detection (TypeScript)

| Signal | Package Manager |
|--------|----------------|
| `pnpm-lock.yaml` | pnpm |
| `bun.lockb` | bun |
| `yarn.lock` | yarn |
| `package-lock.json` | npm |

### Package Manager Detection (Python)

| Signal | Package Manager |
|--------|----------------|
| `pyproject.toml` with `[tool.poetry]` | poetry |
| `pyproject.toml` with `[tool.uv]` or `.python-version` | uv |
| `requirements.txt` only | pip |
| `Pipfile` | pipenv |

### AI Framework Detection

| Dependency | SDK |
|------------|-----|
| `openai` | OpenAI SDK |
| `anthropic` | Anthropic SDK |
| `ai` (npm) | Vercel AI SDK |
| `langchain` | LangChain |
| `google-generativeai` / `@google/generative-ai` | Gemini |
| `boto3` | AWS Bedrock |
| `mistralai` | Mistral |
| `openai-agents` | OpenAI Agents SDK |

---

## Framework Setup

| Framework | Approach |
|-----------|----------|
| Node.js (CJS) | Direct import from `@paid-ai/paid-node` |
| Node.js (ESM) | CJS wrapper via `createRequire` |
| Next.js | `serverExternalPackages` + `instrumentation.ts` + CJS wrapper |
| Python | Direct import from `paid.tracing` |

### CJS Setup

```typescript
// paid.ts (create at project root)
import { initializeTracing, paidAutoInstrument } from "@paid-ai/paid-node";

initializeTracing(process.env.PAID_API_KEY!);
paidAutoInstrument();
```

Import at the **very top** of entry file, before any AI SDK imports:

```typescript
import "./paid"; // must be first import
import OpenAI from "openai";
```

### ESM Setup

Create CJS initialization wrapper (`src/paid-tracing.cjs`):

```javascript
const { initializeTracing, paidAutoInstrument } = require("@paid-ai/paid-node");

initializeTracing(process.env.PAID_API_KEY);
paidAutoInstrument();
```

Create re-export wrapper (`src/lib/paid.ts`):

```typescript
import { createRequire } from "module";
const require = createRequire(import.meta.url);
const paid = require("@paid-ai/paid-node");

export const trace: typeof import("@paid-ai/paid-node").trace = paid.trace;
export const signal: typeof import("@paid-ai/paid-node").signal = paid.signal;
```

Load in entry file:

```typescript
import { createRequire } from "module";
const require = createRequire(import.meta.url);
require("./paid-tracing.cjs");

import { generateText } from "ai";
```

Import `trace` and `signal` from the local wrapper, not the package:

```typescript
import { trace, signal } from "@/lib/paid";
```

### Next.js Setup

Add to `next.config.ts`:

```typescript
const nextConfig: NextConfig = {
  serverExternalPackages: ["@paid-ai/paid-node"],
};
```

Create `src/paid-tracing.cjs` (same as ESM above).

Create `src/instrumentation.ts`:

```typescript
export async function register() {
  if (process.env.NEXT_RUNTIME === "nodejs") {
    const { createRequire } = await import("module");
    const require = createRequire(import.meta.url);
    require("./paid-tracing.cjs");
  }
}
```

Create `src/lib/paid.ts` (same ESM re-export wrapper above).

Use in API routes:

```typescript
import { trace, signal } from "@/lib/paid"; // NOT "@paid-ai/paid-node"

export async function POST(request: Request) {
  const result = await trace({
    externalCustomerId: "<CUSTOMER_ID>",
    externalProductId: "<PRODUCT_ID>",
  }, async () => {
    return streamText({ ... });
  });
}
```

### Python Setup

```python
from paid.tracing import initialize_tracing, paid_autoinstrument, paid_tracing, signal

initialize_tracing()
paid_autoinstrument()

@paid_tracing("<CUSTOMER_ID>", external_product_id="<PRODUCT_ID>")
def run_agent(user_input: str):
    response = openai.chat.completions.create(...)
    signal(event_name="chat_completion", enable_cost_tracing=True)
```

---

## CLI Commands

The Paid CLI handles authentication, resource creation, and validation. This skill is installed via the CLI (`paid init --install-agent-skill`).

### Authentication

```bash
paid init                         # Authenticate and store API key
paid init --env-type Sandbox      # Authenticate against sandbox
paid env                          # Show active environment profile
paid env switch production        # Switch to production profile
```

### Products

```bash
paid product create name="My AI Agent"                    # Create product
paid product list                                         # List all products
paid product get-external --external-id <PRODUCT_ID>      # Get by external ID
paid product update-external --external-id <PRODUCT_ID> name="New Name"
```

### Customers

```bash
paid customer create name="Acme Corp" external_id="acme-123"   # Create customer
paid customer list                                              # List all customers
paid customer get-external --external-id <CUSTOMER_ID>          # Get by external ID
```

### Signals

```bash
paid signal create-bulk '{"signals": [{"event_name": "chat_completion"}, {"event_name": "web_search"}]}'
```

### Pricing

```bash
paid pricing list --product-id <PRODUCT_ID>               # List pricing for product
paid pricing update --product-id <PRODUCT_ID> '{"attribute": "usage", ...}'
```

### Orders & Validation

```bash
paid order list                                           # List orders (verify billing)
paid order get --id <ORDER_ID>                            # Get order details
paid checkout list                                        # List checkouts
```

All commands accept `--file <path>` to read JSON body from a file or `--stdin` to pipe JSON.

---

## Install Commands

### TypeScript / Node.js

| Package Manager | Command |
|----------------|---------|
| npm | `npm install @paid-ai/paid-node` |
| pnpm | `pnpm add @paid-ai/paid-node` |
| yarn | `yarn add @paid-ai/paid-node` |
| bun | `bun add @paid-ai/paid-node` |

### Python

| Package Manager | Command |
|----------------|---------|
| pip | `pip install paid-python` |
| uv | `uv add paid-python` |
| poetry | `poetry add paid-python` |
| pipenv | `pipenv install paid-python` |

---

## Common Gotchas

1. **Signal event names** — Must match exactly between code `signal("event_name")` and the Paid dashboard Event Name
2. **ESM projects** — Use CJS wrapper via `createRequire`, never import directly from `@paid-ai/paid-node`
3. **Next.js requires three pieces** — `serverExternalPackages` in next.config, `instrumentation.ts` for init, and `src/lib/paid.ts` wrapper
4. **Initialize first** — `initializeTracing()` must run before `paidAutoInstrument()` and before any AI client is created
5. **Signal after success** — Only send signals after the work completes successfully
6. **Signals always required** — Autoinstrumentation handles cost tracing; signals handle billable events. Both are needed
7. **Never ask for API key** — Direct users to run `paid init` or add to `.env`
8. **Idempotent customers** — Customers are idempotent by External ID
9. **Vercel AI SDK** — Pass `experimental_telemetry: { isEnabled: true }` for cost data
10. **Python threading** — Call `initialize_tracing()` from the main thread first

---

## Resources

- [Dashboard](https://app.paid.ai)
- [Docs](https://docs.paid.ai)
- CLI Help: `paid help [command]`
