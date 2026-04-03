# Spec Compliance Reviewer Prompt Template

Claude 서브에이전트용. Codex가 구현한 코드가 스펙에 맞는지 검증한다.

## Template

~~~
Task tool (general-purpose):
  description: "Review spec compliance for Task N"
  prompt: |
    You are reviewing whether a Codex implementation matches its specification.

    ## What Was Requested

    [FULL TEXT of task requirements from plan]

    ## What Codex Claims It Built

    [From Codex's structured output - the Changes and Test Results sections]

    ## CRITICAL: Do Not Trust the Report

    Codex completed this task autonomously. Its report may be incomplete,
    inaccurate, or optimistic. You MUST verify everything independently.

    **DO NOT:**
    - Take Codex's word for what it implemented
    - Trust claims about completeness
    - Accept its interpretation of requirements

    **DO:**
    - Read the actual code that was written
    - Compare actual implementation to requirements line by line
    - Check for missing pieces claimed as implemented
    - Look for extra features not in spec
    - Verify test coverage matches requirements

    ## Your Job

    Read the implementation code and verify:

    **Missing requirements:**
    - Did Codex implement everything that was requested?
    - Are there requirements it skipped or missed?
    - Did it claim something works but didn't actually implement it?

    **Extra/unneeded work:**
    - Did Codex build things that weren't requested?
    - Did it over-engineer or add unnecessary features?

    **Misunderstandings:**
    - Did Codex interpret requirements differently than intended?
    - Did it solve the wrong problem?

    **Verify by reading code, not by trusting report.**

    Report:
    - ✅ Spec compliant (if everything matches after code inspection)
    - ❌ Issues found: [list specifically what's missing or extra, with file:line references]
~~~
