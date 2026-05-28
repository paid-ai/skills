---
name: paid-cli
description: "Use when working with Paid from an agent: Paid CLI, Paid API resources, products, pricing, customers, orders, checkouts, signals, tracing, Node.js, Next.js, Python, or SDK setup. Helps coding agents authenticate, inspect environments, discover command shape, implement checkout, instrument usage/cost signals, and avoid billing mistakes with npx @paid-ai/cli."
---

# Paid CLI

Use this single Paid skill for CLI resource work and Paid integration setup. Prefer `npx @paid-ai/cli` for terminal commands unless the project already provides the `paid` binary.

## Choose a Workflow

- CLI/API resource work: stay in this file.
- Signals and tracing setup: read `references/signals-and-tracing.md`.
- Checkout setup: read `references/checkout.md`.
- Paid skill packaging or installation updates: keep all Paid guidance inside this `paid-cli` directory. Do not recreate standalone `setup-paid-signals` or `setup-paid-checkout` skill packages; those workflows are consolidated as references under `paid-cli`.

## CLI Workflow

1. Check the active profile before making API calls:

```bash
npx @paid-ai/cli env
```

2. Discover command shape from the CLI instead of guessing:

```bash
npx @paid-ai/cli --help
npx @paid-ai/cli product --help
npx @paid-ai/cli product create --help
```

3. Use saved auth by default. Do not ask the user to paste secrets unless authentication is missing and the user explicitly wants to initialize it.

```bash
npx @paid-ai/cli init
```

4. Use `--env <name>` when the user names a specific Paid profile. Be careful with production profiles and confirm before destructive operations such as delete or archive.

5. For JSON request bodies, use the simplest reliable form:

```bash
npx @paid-ai/cli customer create name="Acme Corp" externalId=acme
npx @paid-ai/cli customer create --file ./customer.json
cat customer.json | npx @paid-ai/cli customer create --stdin
```

Examples in the reference files may use `paid` as shorthand for the installed binary. Translate those commands to `npx @paid-ai/cli` when there is no local/global `paid` binary.

## Common Gotchas

1. **Signal bulk create needs nested customer** — Use `{ "signals": [{ "customer": { "externalCustomerId": "..." }, "event_name": "..." }] }`, not `{ "externalCustomerId": "..." }` at the top level. The API returns a cryptic "Invalid input" otherwise
2. **Signals don't auto-create pricing** — Sending signals registers event names and auto-creates customers, but does not generate product attributes or pricing. Create products and pricing via `npx @paid-ai/cli product create` first
3. **Monetary amounts are in cents** — The API expects amounts in the smallest currency unit (e.g. cents for USD), so $10 must be passed as `1000`, not `10`
4. **Checkout returns must be verified** — Before provisioning access, retrieve the checkout and require both `status === "completed"` and a matching `externalCustomerId`.
5. **Signal names must match** — Event names in code must match the Paid dashboard and pricing configuration exactly.

## Safety

- Prefer sandbox/local profiles unless the user clearly asks for production.
- Confirm before running destructive commands or commands that affect billing state.
- If the CLI returns an API error, inspect the JSON response and report the actionable validation details.
- Do not ask the user to paste API keys. Use saved auth, `npx @paid-ai/cli init`, or environment variables.
