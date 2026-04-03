---
name: codex-executing-plans
description: Use when you have a written implementation plan and want Codex to handle implementation without review loops — requires codex plugin installed
---

# Codex Executing Plans

Load plan, delegate each task to Codex sequentially, report when complete. No review loops — use codex-subagent-driven-development if you want spec and code quality reviews.

**Announce at start:** "I'm using codex-executing-plans to implement this plan. Codex will handle implementation."

## Prerequisite Check

Before starting, verify codex-plugin-cc is available:

1. Run `/codex:setup --json`
2. If Codex is not available:
   - Tell user: "codex plugin is required. Install with: `/plugin install codex@ai-agent-marketplace`"
   - Stop execution

## When to Use

- You have a written implementation plan
- You want Codex to handle implementation (save Claude tokens)
- You don't need review loops between tasks
- For review loops, use codex-subagent-driven-development instead

## The Process

### Step 1: Load and Review Plan

1. Read plan file
2. Review critically — identify any questions or concerns
3. If concerns: raise them with user before starting
4. If no concerns: create TodoWrite and proceed

### Step 2: Execute Tasks

For each task:

1. Mark as in_progress
2. Prepare Codex prompt:
   - Read `../codex-subagent-driven-development/implementer-prompt.md` template
   - Fill placeholders with task text, context, working directory
3. Dispatch to Codex:
   - Run: `Bash(node "<codex-companion-path>" task --wait --write "<filled prompt>")`
4. Handle result:
   - **DONE / DONE_WITH_CONCERNS** → mark task complete, proceed to next
   - **NEEDS_CONTEXT** → provide context, re-dispatch
   - **BLOCKED** → escalate to user
   - Job `failed` → retry with `--effort medium` or escalate
5. Mark as completed

### Step 3: Complete Development

After all tasks complete:
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Codex reports BLOCKED repeatedly
- Task fails after retry with higher effort
- Plan has critical gaps
- You don't understand an instruction

**Ask for clarification rather than guessing.**

## Red Flags

**Never:**
- Start implementation on main/master branch without explicit user consent
- Skip the prerequisite check
- Ignore Codex's BLOCKED or NEEDS_CONTEXT status
- Force through blockers — stop and ask

## Integration

**Required plugins:**
- **superpowers** — writing-plans, finishing-a-development-branch, using-git-worktrees
- **codex-plugin-cc** — Codex CLI integration
