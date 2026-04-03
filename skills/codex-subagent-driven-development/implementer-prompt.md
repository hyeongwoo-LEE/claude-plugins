# Codex Implementer Prompt Template

Claude가 아래 템플릿의 플레이스홀더를 채운 후, 결과 텍스트를 `codex-companion.mjs task --wait --write`의 프롬프트 인자로 전달한다.

## Template

~~~
<task>
You are implementing: {TASK_NAME}

{TASK_DESCRIPTION}

Context: {SCENE_SETTING_CONTEXT}
Working directory: {WORKING_DIRECTORY}
</task>

<default_follow_through_policy>
- Implement exactly what the task specifies. Do not add features, refactoring, or improvements beyond scope.
- Follow existing codebase patterns and conventions.
- If the task specifies TDD, write the failing test first, then implement.
- If something is unclear, report NEEDS_CONTEXT instead of guessing.
- If the task is too complex or you are stuck, report BLOCKED instead of producing bad work.
</default_follow_through_policy>

<verification_loop>
1. Implement the code changes specified in the task.
2. Run all relevant tests. If no test command is specified, look for the project's standard test runner.
3. If tests fail, fix the implementation until tests pass.
4. Commit your work with a descriptive message.
</verification_loop>

<action_safety>
- Only modify files mentioned in the task or directly required by the implementation.
- Do not restructure, rename, or refactor code outside the task scope.
- Do not add dependencies unless the task explicitly requires them.
</action_safety>

<structured_output_contract>
When done, output EXACTLY this format:

STATUS: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT

## Changes
- List each file changed with a one-line description of what changed

## Test Results
- Test command run and pass/fail count

## Concerns
- Any issues, doubts, or observations (omit section if none)

## Blocked Reason
- What you are stuck on and what you need (omit section if not BLOCKED/NEEDS_CONTEXT)
</structured_output_contract>
~~~

## Placeholders

| Placeholder | Description | Filled by |
|---|---|---|
| `{TASK_NAME}` | Task number and name from plan | Claude (main session) |
| `{TASK_DESCRIPTION}` | Full task text from plan, including all steps and code blocks | Claude (main session) |
| `{SCENE_SETTING_CONTEXT}` | Where this task fits in the overall plan, dependencies on prior tasks, architectural context | Claude (main session) |
| `{WORKING_DIRECTORY}` | Absolute path to current working directory | Claude (main session) |
