---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# Requesting Code Review

code-reviewer 서브에이전트를 디스패치하여 문제가 연쇄되기 전에 잡습니다.

**Core principle:** 일찍 리뷰, 자주 리뷰.

**Announce at start:** "requesting-code-review 스킬을 사용하여 코드 리뷰를 요청합니다."

## When to Request Review

**Mandatory:**
- subagent-driven-development에서 각 Task 후
- 주요 기능 완료 후
- main 머지 전

**Optional but valuable:**
- 막혔을 때 (새로운 관점)
- 리팩토링 전 (기준 확인)
- 복잡한 버그 수정 후

## How to Request

**1. Get git SHAs:**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. Dispatch code-reviewer subagent:**

Task tool 사용, `code-reviewer.md` 템플릿 채우기

**Placeholders:**
- `{WHAT_WAS_IMPLEMENTED}` - 방금 빌드한 것
- `{PLAN_OR_REQUIREMENTS}` - 무엇을 해야 하는지
- `{BASE_SHA}` - 시작 커밋
- `{HEAD_SHA}` - 끝 커밋
- `{DESCRIPTION}` - 간략 요약

**3. Act on feedback:**
- Critical 이슈 즉시 수정
- Important 이슈 진행 전 수정
- Minor 이슈 나중에 메모
- 리뷰어가 틀리면 반박 (논리와 함께)

## Example

```
[Task 2 완료: Add verification function]

You: 진행 전 코드 리뷰 요청하겠습니다.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[Dispatch code-reviewer subagent]
  WHAT_WAS_IMPLEMENTED: conversation index 검증 및 복구 함수
  PLAN_OR_REQUIREMENTS: Task 2 from .claude/github-superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: verifyIndex() 및 repairIndex() 추가, 4가지 이슈 타입

[Subagent returns]:
  Strengths: Clean architecture, real tests
  Issues:
    Important: Missing progress indicators
    Minor: Magic number (100) for reporting interval
  Assessment: Ready to proceed

You: [progress indicators 수정]
[Task 3로 진행]
```

## Integration with Workflows

**Subagent-Driven Development:**
- 각 Task 후 리뷰
- 문제가 복합되기 전에 잡기
- 다음 Task 전에 수정

**Executing Plans:**
- 각 배치 (3 tasks) 후 리뷰
- 피드백 받고 적용하고 계속

**Ad-Hoc Development:**
- 머지 전 리뷰
- 막혔을 때 리뷰

## Red Flags

**Never:**
- "간단해서" 리뷰 건너뛰기
- Critical 이슈 무시
- 수정 안 된 Important 이슈로 진행
- 유효한 기술적 피드백에 반론

**리뷰어가 틀렸으면:**
- 기술적 논리로 반박
- 작동하는 코드/테스트 보여주기
- 명확화 요청

See template at: requesting-code-review/code-reviewer.md

## 관련 스킬

- **receiving-code-review**: 코드 리뷰 받기
- **subagent-driven-development**: 서브에이전트 기반 개발
- **executing-plans**: 계획 실행
