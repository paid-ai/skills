---
name: paid-cli
description: Use when working with the Paid CLI or Paid API from an agent. Helps coding agents authenticate, inspect environments, discover commands, and call Paid resources safely with npx @paid-ai/cli.
---

# Paid CLI

Use the Paid CLI for Paid API resource work from the terminal. Prefer `npx @paid-ai/cli`.

## Workflow

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

## Common Gotchas

1. **Signal bulk create needs nested customer** — Use `{ "signals": [{ "customer": { "externalCustomerId": "..." }, "event_name": "..." }] }`, not `{ "externalCustomerId": "..." }` at the top level. The API returns a cryptic "Invalid input" otherwise
2. **Signals don't auto-create pricing** — Sending signals registers event names and auto-creates customers, but does not generate product attributes or pricing. Create products and pricing via `paid product create` first
3. **Monetary amounts are in cents** — The API expects amounts in the smallest currency unit (e.g. cents for USD), so $10 must be passed as `1000`, not `10`

## Safety

- Prefer sandbox/local profiles unless the user clearly asks for production.
- Confirm before running destructive commands or commands that affect billing state.
- If the CLI returns an API error, inspect the JSON response and report the actionable validation details.
