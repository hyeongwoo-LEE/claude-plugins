# superpowers-with-codex

A standalone plugin that includes all superpowers skills plus Codex-enhanced execution. Works independently — no need to install the superpowers plugin separately.

## What It Does

Provides **all 14 superpowers skills** with 3 execution skills enhanced to support both Codex delegation and Claude-native execution:

### Codex-Enhanced Execution Skills

| Skill | With Codex | Without Codex |
|-------|-----------|---------------|
| codex-subagent-driven-development | Codex implements, Claude reviews | Claude subagents implement and review |
| codex-executing-plans | Codex implements tasks sequentially | Claude executes tasks directly |
| codex-dispatching-parallel-agents | Codex runs tasks in parallel | Claude subagents run in parallel |

### Included Workflow Skills (from superpowers)

- **brainstorming** — Explore ideas and designs before implementation
- **writing-plans** — Write comprehensive implementation plans
- **finishing-a-development-branch** — Complete development and integrate work
- **using-git-worktrees** — Create isolated workspaces for feature work
- **test-driven-development** — Red-green-refactor cycle
- **systematic-debugging** — 4-phase root cause investigation
- **requesting-code-review** — Dispatch code review subagents
- **receiving-code-review** — Technical evaluation of review feedback
- **verification-before-completion** — Evidence before completion claims
- **writing-skills** — TDD for skill documentation
- **using-superpowers** — Skill discovery and usage

## How Codex-Enhanced Skills Work

1. **Detect Codex availability** — runs `/codex:setup --json`
2. **If Codex available** — asks user: "Codex or Claude?"
3. **If Codex not available** — automatically falls back to Claude-native execution

## Requirements

**Required:**
- None — works standalone

**Optional (enables Codex delegation for execution skills):**
- [codex plugin](https://github.com/openai/codex-plugin-cc) installed and authenticated

## Installation

```bash
/plugin marketplace add hyeongwoo-LEE/claude-plugins
/plugin install superpowers-with-codex
```

## Usage

Start with brainstorming or writing-plans, then execute:

```
brainstorming → writing-plans → codex-subagent-driven-development → finishing-a-development-branch
```

Or invoke any skill directly as needed.
