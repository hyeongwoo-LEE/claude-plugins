# superpowers-with-codex Design Spec

## Overview

superpowers-with-codex는 superpowers의 실행 스킬(executing-plans, subagent-driven-development, dispatching-parallel-agents)에 대응하는 Codex 위임 버전을 제공하는 Claude Code 플러그인이다.

Claude가 기획, 조율, 리뷰를 담당하고 Codex가 실제 구현을 수행함으로써 Claude 토큰 사용량을 절약한다.

## Goals

- Claude 토큰 절약: 가장 토큰을 많이 소비하는 구현 작업을 Codex에 위임
- 기존 superpowers 워크플로우와 자연스러운 통합
- 리뷰 품질 유지: 스펙 리뷰와 코드 품질 리뷰는 Claude가 수행

## Non-Goals

- superpowers의 기존 스킬을 대체하지 않음 (함께 사용, 사용자가 선택)
- brainstorming, writing-plans 등 비실행 스킬은 다루지 않음

## Dependencies

- **superpowers 플러그인** (필수): brainstorming, writing-plans, finishing-a-development-branch, using-git-worktrees, spec-reviewer-prompt, code-reviewer 등 사용
- **codex-plugin-cc 플러그인** (필수): `codex-companion.mjs task` 명령으로 Codex에 작업 위임

## Plugin Structure

```
superpowers-with-codex/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── codex-subagent-driven-development/
│   │   ├── SKILL.md
│   │   ├── implementer-prompt.md
│   │   └── spec-reviewer-prompt.md
│   ├── codex-executing-plans/
│   │   └── SKILL.md
│   └── codex-dispatching-parallel-agents/
│       └── SKILL.md
├── agents/
│   └── code-reviewer.md
├── docs/
│   └── specs/
│       └── 2026-04-03-superpowers-with-codex-design.md
└── README.md
```

## Prerequisite Check

모든 스킬 실행 시 첫 단계에서 codex-plugin-cc 설치 여부를 확인한다:

1. `/codex:setup --json` 실행
2. 성공 → 다음 단계 진행
3. 실패 → `/plugin install codex@ai-agent-marketplace` 안내 후 중단

## Skill 1: codex-subagent-driven-development

superpowers의 subagent-driven-development와 동일한 흐름이되, 구현을 Codex에 위임한다.

### Role Split

| 역할 | 담당 |
|------|------|
| 구현 | Codex (`task --wait --write`) |
| 스펙 리뷰 | Claude 서브에이전트 |
| 코드 품질 리뷰 | Claude 서브에이전트 |
| 조율/프롬프트 구성 | Claude (메인 세션) |

### Flow

```
태스크마다:
  1. implementer-prompt.md에 태스크 텍스트 + 컨텍스트 채워넣기
  2. Codex에 위임 (codex-companion.mjs task --wait --write "프롬프트")
  3. Codex 결과 확인
     - DONE → 4단계
     - DONE_WITH_CONCERNS → 우려 사항 검토 후 4단계
     - NEEDS_CONTEXT → 컨텍스트 보강 후 재위임
     - BLOCKED → 사용자에게 에스컬레이션
  4. Claude 서브에이전트로 스펙 리뷰 (spec-reviewer-prompt.md)
     - 통과 → 5단계
     - 미통과 → Codex에 수정 지시 (task --wait --write --resume) → 다시 4단계
  5. Claude 서브에이전트로 코드 품질 리뷰 (code-reviewer.md)
     - 통과 → 태스크 완료
     - 미통과 → Codex에 수정 지시 (task --wait --write --resume) → 다시 5단계
```

### Key Design Decisions

- Codex 수정 시 `--resume`으로 같은 스레드를 이어서 사용 (컨텍스트 유지)
- 스펙 리뷰는 반드시 코드 품질 리뷰보다 먼저 수행
- Claude가 Codex 결과의 상태 코드(DONE/BLOCKED 등)를 파싱해서 다음 단계를 결정
- 리뷰 프롬프트(spec-reviewer-prompt.md, code-reviewer.md)는 이 플러그인에 자체 복사본을 포함한다. superpowers의 프롬프트를 직접 참조하면 경로가 설치 환경마다 다를 수 있고, superpowers 업데이트 시 예기치 않게 동작이 변경될 수 있기 때문이다.

## Skill 2: codex-executing-plans

superpowers의 executing-plans와 동일한 흐름이되, 구현을 Codex에 위임한다. 리뷰 루프는 없다.

### Flow

```
1. 계획 파일 읽기, 태스크 목록 생성
2. 태스크마다:
   a. 프롬프트 구성 (태스크 텍스트 + 컨텍스트)
   b. Codex에 위임 (task --wait --write)
   c. 결과 확인
      - 성공 → 다음 태스크
      - 실패 → 사용자에게 에스컬레이션
3. 전체 완료 후 → superpowers:finishing-a-development-branch
```

## Skill 3: codex-dispatching-parallel-agents

독립적인 작업 여러 개를 Codex에 동시에 보낸다.

### Flow

```
1. 독립 작업 식별 및 그룹핑
2. 작업별 프롬프트 구성
3. 전부 Codex에 동시 위임 (task --background --write) × N개
4. 전체 완료 대기 (status --wait 반복)
5. 결과 회수 (result × N개)
6. Claude가 결과 통합 검토:
   - 충돌 여부 확인 (같은 파일 수정했는지)
   - 전체 테스트 실행
   - 문제 있으면 사용자에게 보고
```

### Key Design Decisions

- `--background`로 비동기 실행 (병렬 처리를 위해)
- 통합 검토는 Claude가 직접 수행 (충돌 감지, 테스트 실행)

## Implementer Prompt Template

Codex에 넘기는 프롬프트는 gpt-5-4-prompting 레시피 블록 구조를 따른다.

```markdown
<task>
{TASK_DESCRIPTION}

Context: {SCENE_SETTING_CONTEXT}
Working directory: {WORKING_DIRECTORY}
</task>

<verification_loop>
1. 구현 완료 후 반드시 테스트 실행
2. 테스트 통과 확인 후 커밋
3. 커밋 메시지에 태스크 번호 포함
</verification_loop>

<structured_output_contract>
작업 완료 후 아래 형식으로 보고:

STATUS: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT

## Changes
- 수정한 파일 목록과 변경 내용

## Test Results
- 실행한 테스트와 결과

## Concerns (if any)
- 발견한 문제나 의문점
</structured_output_contract>
```

### Placeholder Substitution

| 플레이스홀더 | 설명 | 채우는 주체 |
|---|---|---|
| `{TASK_DESCRIPTION}` | 계획에서 추출한 태스크 전체 텍스트 | Claude (메인 세션) |
| `{SCENE_SETTING_CONTEXT}` | 태스크가 전체 계획에서 차지하는 위치, 의존성 등 | Claude (메인 세션) |
| `{WORKING_DIRECTORY}` | 현재 작업 디렉토리 경로 | Claude (메인 세션) |

## Workflow Integration

기존 superpowers 워크플로우의 실행 단계에서 선택지로 추가된다.

```
superpowers:brainstorming
  ↓
superpowers:using-git-worktrees
  ↓
superpowers:writing-plans
  ↓ (사용자 선택)
  ├── superpowers:subagent-driven-development       (기존, Claude 전부)
  ├── superpowers:executing-plans                    (기존, Claude 전부)
  ├── codex-subagent-driven-development              (신규, Codex 구현 + Claude 리뷰)
  └── codex-executing-plans                          (신규, Codex 구현)
  ↓
superpowers:finishing-a-development-branch
```

codex-dispatching-parallel-agents는 위 흐름과 별개로, 독립적인 문제 여러 개를 병렬 처리할 때 수동 사용한다.

## Error Handling

### Codex 실행 실패

Codex job 상태가 `failed`인 경우:

1. `errorMessage` 확인
2. 컨텍스트 부족 → 프롬프트 보강 후 재시도
3. 태스크 복잡 → `--effort` 올려서 재시도
4. 근본 문제 → 사용자에게 에스컬레이션

### 리뷰 미통과

스펙 리뷰 또는 코드 품질 리뷰에서 이슈가 발견된 경우:

1. 리뷰어가 발견한 이슈를 Codex 수정 프롬프트에 포함
2. `task --wait --write --resume`으로 같은 스레드에서 수정 지시
3. 수정 후 같은 리뷰 다시 실행
4. 3회 반복 후에도 미통과 → 사용자에게 에스컬레이션

## Priority

구현 우선순위:

1. **codex-subagent-driven-development** — 핵심 스킬, 가장 토큰 절약 효과 큼
2. **codex-executing-plans** — 간소화 버전
3. **codex-dispatching-parallel-agents** — 병렬 처리
