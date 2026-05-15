# Paid Agent Skills

Agent skills for working with Paid from coding agents such as Claude Code, Codex, Cursor, and others.

## Install

Install a skill with:

```bash
npx skills add paid-ai/skills --skill paid-cli
npx skills add paid-ai/skills --skill setup-paid
```

To install files instead of symlinks:

```bash
npx skills add paid-ai/skills --skill paid-cli --copy
npx skills add paid-ai/skills --skill setup-paid --copy
```

## Skills

- `paid-cli`: Helps agents use `npx @paid-ai/cli` safely, inspect environments, and discover commands.
- `setup-paid`: Guides agents through integrating Paid — the all-in-one business engine for AI agents handling pricing, billing, and usage tracking.
