# superpowers-with-codex

Codex-enhanced execution skills that support both Codex delegation and Claude-native execution. Automatically detects Codex availability and lets the user choose their preferred execution mode.

## What It Does

Provides enhanced versions of three execution skills that work **with or without Codex**:

| Skill | With Codex | Without Codex |
|-------|-----------|---------------|
| codex-subagent-driven-development | Codex implements, Claude reviews | Claude subagents implement and review |
| codex-executing-plans | Codex implements tasks sequentially | Claude executes tasks directly |
| codex-dispatching-parallel-agents | Codex runs tasks in parallel | Claude subagents run in parallel |

## How It Works

Each skill follows the same flow:

1. **Detect Codex availability** — runs `/codex:setup --json`
2. **If Codex available** — asks user: "Codex로 위임할까요, Claude로 실행할까요?"
3. **If Codex not available** — automatically falls back to Claude-native execution
4. **Execute** — follows the chosen path (Codex or Claude)

## Requirements

**Required:**
- None — works standalone with Claude-native execution

**Optional (enables Codex delegation):**
- [codex plugin](https://github.com/openai/codex-plugin-cc) installed and authenticated
- [superpowers](https://github.com/obra/superpowers) plugin (for additional workflow skills like writing-plans, finishing-a-development-branch)

## Installation

```bash
/plugin marketplace add hyeongwoo-LEE/claude-plugins
/plugin install superpowers-with-codex
```

## Usage

These skills can be invoked directly or used after `superpowers:writing-plans` completes:

- **codex-subagent-driven-development** — best for plans with independent tasks needing review
- **codex-executing-plans** — best for sequential task execution without review loops
- **codex-dispatching-parallel-agents** — best for 2+ independent tasks that can run concurrently
