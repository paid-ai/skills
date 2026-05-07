# Paid Agent Skills

Agent skills for working with Paid from coding agents such as Claude Code, Codex, Cursor, and others.

## Install

Install the Paid CLI skill with:

```bash
npx skills add paid-ai/skills --skill paid-cli
```

To install files instead of symlinks:

```bash
npx skills add paid-ai/skills --skill paid-cli --copy
```

## Skills

- `paid-cli`: Helps agents use `npx @paid-ai/cli` safely, inspect environments, discover commands, and avoid exposing API keys.
