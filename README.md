# superpowers-with-codex

Superpowers execution skills that delegate implementation to Codex while Claude handles orchestration and review.

## What It Does

Provides Codex-powered alternatives to three superpowers execution skills:

| Skill | Codex Role | Claude Role |
|-------|-----------|-------------|
| codex-subagent-driven-development | Implementation | Orchestration + Spec Review + Code Quality Review |
| codex-executing-plans | Implementation | Orchestration |
| codex-dispatching-parallel-agents | Parallel Implementation | Orchestration + Integration Review |

## Requirements

- [superpowers](https://github.com/obra/superpowers) plugin installed
- [codex-plugin-cc](https://github.com/openai/codex-plugin-cc) plugin installed
- Codex CLI authenticated (`codex login`)

## Installation

```bash
/plugin install <github-url>
```

## Usage

After `superpowers:writing-plans` completes, choose the Codex-powered execution option:

- **codex-subagent-driven-development** — Codex implements, Claude reviews (recommended)
- **codex-executing-plans** — Codex implements, no review loop
- **codex-dispatching-parallel-agents** — parallel independent tasks via Codex
