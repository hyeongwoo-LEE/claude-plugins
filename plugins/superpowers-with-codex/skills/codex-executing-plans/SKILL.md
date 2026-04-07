---
name: codex-executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints — supports both Codex delegation and Claude-native execution
---

# Executing Plans (Codex-Enhanced)

Load plan, review critically, execute all tasks, report when complete. Supports both Codex-delegated and Claude-native execution paths.

**Announce at start:** "I'm using codex-executing-plans to implement this plan."

## Step 0: Execution Mode Selection

Before starting, determine execution mode:

1. Check if Codex is available: Run `/codex:setup --json`
2. **If Codex IS available:**
   - Ask the user: "Codex가 사용 가능합니다. 어떤 방식으로 실행할까요?\n  1. **Codex** — Codex에 구현을 위임합니다 (Claude 토큰 절약)\n  2. **Claude** — Claude 서브에이전트가 직접 구현합니다 (기존 방식)"
   - Wait for user's choice before proceeding
3. **If Codex is NOT available:**
   - Announce: "Codex를 사용할 수 없어 Claude 방식으로 실행합니다."
   - Proceed with Claude path

## Step 1: Load and Review Plan

1. Read plan file
2. Review critically — identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

## Step 2: Execute Tasks

### Path A: Codex Execution

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

### Path B: Claude Execution

For each task:

1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

**Note:** If subagents are available, consider using codex-subagent-driven-development instead for higher quality with two-stage review.

## Step 3: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers-with-codex:finishing-a-development-branch (if available)
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly
- (Codex path) Codex reports BLOCKED repeatedly or task fails after retry

**Ask for clarification rather than guessing.**

## Red Flags

**Never:**
- Start implementation on main/master branch without explicit user consent
- Skip the execution mode selection step
- Ignore Codex's BLOCKED or NEEDS_CONTEXT status (Codex path)
- Skip verifications (Claude path)
- Force through blockers — stop and ask

## Integration

**Required workflow skills (if superpowers plugin installed):**
- **superpowers-with-codex:using-git-worktrees** — Set up isolated workspace before starting
- **superpowers-with-codex:writing-plans** — Creates the plan this skill executes
- **superpowers-with-codex:finishing-a-development-branch** — Complete development after all tasks

**Required for Codex path:**
- **codex plugin** — Codex CLI integration
