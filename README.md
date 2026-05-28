# Paid Agent Skill

A consolidated agent skill for working with Paid from coding agents such as Claude Code, Codex, Cursor, and others.

## Install

Install the Paid skill with:

```bash
npx skills add paid-ai/skills --skill paid-cli
```

To install files instead of symlinks:

```bash
npx skills add paid-ai/skills --skill paid-cli --copy
```

## Skill

- `paid-cli`: Helps agents use `npx @paid-ai/cli` safely, inspect environments, discover commands, set up Paid signals/tracing, and implement Paid checkout.

## Migration

Previous standalone Paid skills (`setup-paid-signals` and `setup-paid-checkout`) are now references inside `paid-cli`. Fresh installs and updates should install only `paid-cli`.
