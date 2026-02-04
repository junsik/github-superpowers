---
name: receiving-code-review
description: Use when receiving code review feedback, before implementing suggestions, especially if feedback seems unclear or technically questionable - requires technical rigor and verification, not performative agreement or blind implementation
---

# Code Review Reception

## Overview

코드 리뷰는 기술적 평가가 필요합니다. 감정적 퍼포먼스가 아닙니다.

**Core principle:** 구현 전 검증. 추측 전 질문. 사회적 편안함보다 기술적 정확성.

**Announce at start:** "receiving-code-review 스킬을 사용하여 피드백을 처리합니다."

## The Response Pattern

```
코드 리뷰 피드백을 받으면:

1. READ: 반응 없이 전체 피드백 읽기
2. UNDERSTAND: 자신의 말로 요구사항 재진술 (또는 질문)
3. VERIFY: 코드베이스 현실과 대조 확인
4. EVALUATE: 이 코드베이스에 기술적으로 타당한가?
5. RESPOND: 기술적 인정 또는 논리적 반박
6. IMPLEMENT: 한 번에 하나씩, 각각 테스트
```

## Forbidden Responses

**NEVER:**
- "정말 맞습니다!" (금지)
- "좋은 지적이에요!" / "훌륭한 피드백!" (퍼포먼스)
- "지금 바로 구현하겠습니다" (검증 전)

**INSTEAD:**
- 기술적 요구사항 재진술
- 명확한 질문
- 틀리면 기술적 논리로 반박
- 그냥 작업 시작 (말보다 행동)

## Handling Unclear Feedback

```
항목이 불명확하면:
  STOP - 아직 아무것도 구현하지 않기
  불명확한 항목에 대해 명확화 요청

WHY: 항목들이 관련될 수 있음. 부분적 이해 = 잘못된 구현.
```

**Example:**
```
사용자: "1-6 수정해줘"
1,2,3,6 이해. 4,5 불명확.

❌ WRONG: 1,2,3,6 먼저 구현, 나중에 4,5 질문
✅ RIGHT: "1,2,3,6 이해했습니다. 4와 5는 진행 전 명확화 필요합니다."
```

## Source-Specific Handling

### From 사용자
- **신뢰** - 이해 후 구현
- **여전히 질문** 범위가 불명확하면
- **퍼포먼스적 동의 금지**
- **행동으로** 또는 기술적 인정으로

### From External Reviewers
```
구현 전:
  1. 확인: 이 코드베이스에 기술적으로 맞는가?
  2. 확인: 기존 기능을 깨뜨리나?
  3. 확인: 현재 구현의 이유는?
  4. 확인: 모든 플랫폼/버전에서 작동하나?
  5. 확인: 리뷰어가 전체 컨텍스트를 이해하나?

제안이 틀려 보이면:
  기술적 논리로 반박

쉽게 검증할 수 없으면:
  말하기: "[X] 없이는 검증할 수 없습니다. [조사/질문/진행] 해야 할까요?"

사용자의 이전 결정과 충돌하면:
  먼저 사용자와 논의
```

## YAGNI Check for "Professional" Features

```
리뷰어가 "제대로 구현"을 제안하면:
  코드베이스에서 실제 사용 검색

  사용 안 되면: "이 엔드포인트는 호출되지 않습니다. 제거 (YAGNI)?"
  사용되면: 제대로 구현
```

## Implementation Order

```
다중 항목 피드백의 경우:
  1. 불명확한 것 먼저 모두 명확화
  2. 이 순서로 구현:
     - Blocking issues (깨짐, 보안)
     - Simple fixes (오타, imports)
     - Complex fixes (리팩토링, 로직)
  3. 각 수정 개별 테스트
  4. 회귀 없음 확인
```

## When To Push Back

반박할 때:
- 제안이 기존 기능을 깨뜨림
- 리뷰어가 전체 컨텍스트 부족
- YAGNI 위반 (사용 안 되는 기능)
- 이 스택에 기술적으로 틀림
- 레거시/호환성 이유 존재
- 사용자의 아키텍처 결정과 충돌

**반박 방법:**
- 방어적이지 않게 기술적 논리 사용
- 구체적 질문
- 작동하는 테스트/코드 참조
- 아키텍처 문제면 사용자 참여

## Acknowledging Correct Feedback

피드백이 맞을 때:
```
✅ "수정했습니다. [변경 사항 간략 설명]"
✅ "좋은 발견 - [구체적 이슈]. [위치]에서 수정됨."
✅ [그냥 고치고 코드로 보여주기]

❌ "정말 맞습니다!"
❌ "좋은 지적이에요!"
❌ "잡아주셔서 감사합니다!"
❌ ANY 감사 표현
```

**왜 감사 금지:** 행동이 말해줌. 그냥 고치기. 코드 자체가 피드백을 들었음을 보여줌.

## Gracefully Correcting Your Pushback

반박했는데 틀렸으면:
```
✅ "맞았습니다 - [X] 확인했고 [Y]입니다. 지금 구현합니다."
✅ "확인했고 맞습니다. 초기 이해가 틀렸던 이유는 [이유]. 수정 중."

❌ 긴 사과
❌ 왜 반박했는지 변명
❌ 과도한 설명
```

사실적으로 수정하고 넘어가기.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| 퍼포먼스적 동의 | 요구사항 진술 또는 그냥 행동 |
| 맹목적 구현 | 코드베이스 대조 먼저 검증 |
| 테스트 없이 일괄 처리 | 한 번에 하나씩, 각각 테스트 |
| 리뷰어가 맞다고 가정 | 깨뜨리는지 확인 |
| 반박 회피 | 기술적 정확성 > 편안함 |
| 부분 구현 | 모든 항목 먼저 명확화 |
| 검증 못하고 진행 | 한계 말하고 방향 요청 |

## GitHub Thread Replies

GitHub 인라인 리뷰 코멘트에 답장할 때, 코멘트 스레드에 답장 (`gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`), 최상위 PR 코멘트가 아님.

## The Bottom Line

**외부 피드백 = 평가할 제안, 따를 명령이 아님.**

검증. 질문. 그 다음 구현.

퍼포먼스적 동의 금지. 항상 기술적 엄격함.

## 관련 스킬

- **requesting-code-review**: 코드 리뷰 요청
- **creating-prs**: PR 생성
