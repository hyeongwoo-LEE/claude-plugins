---
name: codex-dispatching-parallel-agents
description: Use when facing 2+ independent tasks that can be delegated to Codex in parallel without shared state or sequential dependencies — requires codex plugin installed
---

# Codex Dispatching Parallel Agents

Dispatch independent tasks to Codex in parallel. Each task runs in its own Codex thread concurrently. Claude coordinates dispatch, monitors progress, and integrates results.

**Core principle:** One Codex task per independent problem domain. Let them work concurrently.

**Announce at start:** "I'm using codex-dispatching-parallel-agents to run N independent tasks via Codex in parallel."

## Prerequisite Check

Before starting, verify codex-plugin-cc is available:

1. Run `/codex:setup --json`
2. If Codex is not available:
   - Tell user: "codex plugin is required. Install with: `/plugin install codex@ai-agent-marketplace`"
   - Stop execution

## When to Use

**Use when:**
- 2+ independent tasks with no shared state
- Each task can be understood without context from others
- Tasks won't edit the same files

**Don't use when:**
- Tasks are related (fix one might fix others)
- Tasks would edit the same files (merge conflicts)
- Need to understand full system state first

## The Process

### 1. Identify Independent Domains

Group work by what's independent:
- Different test files failing for different reasons
- Different subsystems needing fixes
- Different features that don't overlap

### 2. Create Focused Prompts

For each task, prepare a Codex prompt using the implementer-prompt.md template from codex-subagent-driven-development:
- **Specific scope:** one problem domain
- **Clear goal:** what "done" looks like
- **Constraints:** don't change code outside scope
- **Expected output:** structured status report

### 3. Dispatch All in Parallel

For each task:
```bash
node "<codex-companion-path>" task --background --write "<prompt>"
```

All dispatched in rapid succession (not waiting for completion).

### 4. Monitor Progress

Poll for completion:
```bash
node "<codex-companion-path>" status --wait
```

Or check periodically:
```bash
node "<codex-companion-path>" status
```

### 5. Collect Results

For each completed job:
```bash
node "<codex-companion-path>" result <job-id>
```

Parse STATUS from each result.

### 6. Integrate and Verify

Claude reviews all results:
1. **Check for conflicts:** Did any tasks edit the same files?
   - If yes: review the overlapping changes, resolve manually or escalate
2. **Run full test suite:** Verify all changes work together
3. **Spot check:** Review each task's changes for correctness
4. **Report:** Summarize what was done, any issues found

## Error Handling

- If a task fails: note it, continue with others, address failed tasks after
- If multiple tasks edited same file: flag conflict to user
- If test suite fails after integration: investigate which task's changes caused it

## Red Flags

**Never:**
- Dispatch tasks that would edit the same files
- Skip the integration verification step
- Ignore failed tasks
- Start on main/master branch without user consent

## Integration

**Required plugins:**
- **codex-plugin-cc** — Codex CLI integration

**Optional:**
- **superpowers** — for finishing-a-development-branch after all tasks complete
