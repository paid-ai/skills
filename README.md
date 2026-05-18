# Paid Agent Skills

Agent skills for working with Paid from coding agents such as Claude Code, Codex, Cursor, and others.

## Install

Install a skill with:

```bash
npx skills add paid-ai/skills --skill paid-cli
npx skills add paid-ai/skills --skill setup-paid-signals
npx skills add paid-ai/skills --skill setup-paid-checkout
```

To install files instead of symlinks:

```bash
npx skills add paid-ai/skills --skill paid-cli --copy
npx skills add paid-ai/skills --skill setup-paid-signals --copy
npx skills add paid-ai/skills --skill setup-paid-checkout --copy
```

## Skills

- `paid-cli`: Helps agents use `npx @paid-ai/cli` safely, inspect environments, and discover commands.
- `setup-paid-signals`: Sets up Paid signals and tracing — autoinstrumentation, manual tracing, and framework-specific setup for Node.js, Next.js, and Python.
- `setup-paid-checkout`: Sets up Paid checkout — create sessions, handle payment returns, identify customers, and configure checkout options.
