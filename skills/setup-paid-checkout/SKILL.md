---
name: setup-paid-checkout
description: Skill for setting up Paid checkout — create checkout sessions, handle payment returns, identify customers, and configure checkout options.
---

# Paid Checkout Guide

**Always consult [docs.paid.ai](https://docs.paid.ai) for code examples and latest API.**

Create checkout sessions programmatically to collect payments from your customers. Checkout isn't just for one-time payments — when a customer completes checkout, Paid creates an order (subscription) that tracks their plan, billing cycle, and usage over time.

---

## Prerequisites

- A **Product ID** from the [Products dashboard](https://app.paid.ai/products) or via the [API](https://docs.paid.ai/api-reference/api-reference/products/create-product)
- [Stripe connected](https://app.paid.ai/settings/billing) (use Stripe sandbox for test mode with test payment methods)

---

## Creating a Session

Call the API with the products you want the customer to purchase and a URL to redirect them to after payment. The API returns an object containing a `url` where you should redirect your customer.

```typescript
const client = new PaidClient({ token: "YOUR_PAID_API_KEY" });

const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  externalCustomerId: currentUser.id,
  successUrl: "https://example.com/success?checkout_id={CHECKOUT_ID}",
});

// Redirect your customer to this URL
redirect(checkout.body.url);
```

```python
from paid import Paid

client = Paid(token="YOUR_PAID_API_KEY")

checkout = client.checkouts.create_checkout(
    products=[{"id": "prod_abc123"}],
    external_customer_id=current_user.id,
    success_url="https://example.com/success?checkout_id={CHECKOUT_ID}",
)

# Redirect your customer to this URL
redirect(checkout.url)
```

The `{CHECKOUT_ID}` placeholder in `successUrl` is automatically replaced with the checkout's display ID (e.g. `chk_abc123`), so you can retrieve the result when the customer returns.

---

## Handling the Return

After payment, verify the checkout before provisioning access. Check that status is `completed` **and** that `externalCustomerId` matches the authenticated user — this prevents one user from using another's checkout ID to gain unauthorised access.

```typescript
const checkoutId = req.query.checkout_id;
const checkout = await client.checkouts.getCheckout({ id: checkoutId });

if (
  checkout.body.status === "completed" &&
  checkout.body.externalCustomerId === currentUser.id
) {
  await provisionUserAccess(currentUser.id);
} else {
  showErrorMessage();
}
```

```python
checkout_id = request.args.get("checkout_id")
checkout = client.checkouts.get_checkout(id=checkout_id)

if (
    checkout.status == "completed"
    and checkout.external_customer_id == current_user.id
):
    provision_user_access(current_user.id)
else:
    show_error_message()
```

---

## Identifying the Customer

There are three ways that customers become associated with a checkout.

### `externalCustomerId` (recommended)

Pass your own user identifier — a database ID, UUID, or email. On the first checkout for a given ID, Paid creates the customer automatically. On subsequent checkouts with the same ID, Paid resolves to the existing customer, so upgrades and plan changes work seamlessly.

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  externalCustomerId: currentUser.id,
  successUrl: "https://example.com/success",
});
```

### `customerId`

Reference an existing Paid customer by their `cus_...` ID (created via the API or dashboard).

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  customerId: "cus_xyz789",
  successUrl: "https://example.com/success",
});
```

Only one of `customerId` or `externalCustomerId` may be provided.

### Anonymous

Omit both for public pricing pages — the checkout page collects the customer's name and email, and Paid creates a customer record on completion.

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  successUrl: "https://example.com/welcome",
});
```

```python
checkout = client.checkouts.create_checkout(
    products=[{"id": "prod_abc123"}],
    success_url="https://example.com/welcome",
)
```

---

## Multiple Products

Pass several products in a single session. If a product has plan tiers, the customer picks one during checkout. If it doesn't, it's shown directly with its base pricing.

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [
    { id: "prod_platform" },
    { id: "prod_addon_storage" },
  ],
  successUrl: "https://example.com/success",
});
```

```python
checkout = client.checkouts.create_checkout(
    products=[
        {"id": "prod_platform"},
        {"id": "prod_addon_storage"},
    ],
    success_url="https://example.com/success",
)
```

---

## Automatic Upgrades

If the `externalCustomerId` already has an active order for the product, Paid automatically handles the upgrade with proration. No need for separate upgrade logic — the same API call works for both new purchases and plan changes.

```typescript
// Works for both new customers and upgrades
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  externalCustomerId: currentUser.id,
  successUrl: "https://example.com/success",
});
```

```python
# Works for both new customers and upgrades
checkout = client.checkouts.create_checkout(
    products=[{"id": "prod_abc123"}],
    external_customer_id=current_user.id,
    success_url="https://example.com/success",
)
```

---

## Checkout Options

### Expiration

Sessions expire after 24 hours by default. Override with an ISO 8601 timestamp:

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  successUrl: "https://example.com/welcome",
  expiresAt: "2026-04-01T00:00:00.000Z",
});
```

### Single Use

Sessions are single-use by default — the link is archived once started. Set `singleUse: false` for reusable links (e.g. public pricing pages):

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  successUrl: "https://example.com/success",
  singleUse: false,
});
```

### Currency

Lock to a specific currency. If omitted, the customer can pay in any currency supported by the plan:

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  successUrl: "https://example.com/success",
  currency: "GBP",
});
```

### Collecting Customer Information

Paid collects billing address by default (required for tax). You can also request a phone number:

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  successUrl: "https://example.com/success",
  collectAddress: true,  // default — required for tax
  collectPhone: true,    // optional — off by default
});
```

### Metadata

Attach arbitrary key-value metadata for your own tracking. This data is returned on the checkout object but never shown to the customer:

```typescript
const checkout = await client.checkouts.createCheckout({
  products: [{ id: "prod_abc123" }],
  successUrl: "https://example.com/welcome",
  metadata: { campaign: "spring_launch", referrer: "pricing_page" },
});
```

```python
checkout = client.checkouts.create_checkout(
    products=[{"id": "prod_abc123"}],
    success_url="https://example.com/welcome",
    metadata={"campaign": "spring_launch", "referrer": "pricing_page"},
)
```

---

## Options Reference

| Option | Default | Notes |
|--------|---------|-------|
| `products` | — | Array of `{ id }` objects (required) |
| `successUrl` | — | Redirect URL after payment (required). `{CHECKOUT_ID}` placeholder supported |
| `externalCustomerId` | — | Your user identifier — auto-creates/resolves customers |
| `customerId` | — | Paid customer ID (`cus_...`). Mutually exclusive with `externalCustomerId` |
| `expiresAt` | 24 hours | ISO 8601 timestamp |
| `singleUse` | `true` | Set `false` for reusable links |
| `currency` | Any supported | Lock to a specific currency (e.g. `"GBP"`) |
| `collectAddress` | `true` | Required for tax calculation |
| `collectPhone` | `false` | Optionally collect phone number |
| `metadata` | — | Arbitrary key-value pairs (not shown to customer) |

---

## Common Gotchas

1. **Stripe must be connected** — Checkout will fail without [Stripe connected](https://app.paid.ai/settings/billing)
2. **Verify on return** — Always check `status === "completed"` AND `externalCustomerId` matches the authenticated user before provisioning access
3. **One customer identifier** — Only one of `customerId` or `externalCustomerId` may be provided
4. **Automatic upgrades** — Same API call handles new purchases and plan changes when using `externalCustomerId`
5. **Never hardcode API keys** — Use environment variables or `paid init`
6. **`getCheckout` takes an object** — Pass `{ id: "chk_..." }`, not a bare string. In Python use `get_checkout(id=checkout_id)`
7. **Use the latest SDK version** — Always install the latest version of `@paid-ai/paid-node` or `paid-python` to avoid missing features or bug fixes

---

## Resources

- [Dashboard](https://app.paid.ai)
- [Docs](https://docs.paid.ai)
- CLI Help: `paid help [command]`
