---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

랜덤한 수정은 시간 낭비이고 새 버그를 만듭니다.

**Core principle:** 수정 시도 전에 반드시 근본 원인을 찾으세요. 증상 수정은 실패입니다.

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

Phase 1을 완료하지 않으면 수정을 제안할 수 없습니다.

## The Four Phases

### Phase 1: Root Cause Investigation

**수정 시도 전에:**

1. **에러 메시지 주의 깊게 읽기**
   - 스택 트레이스 전체 읽기
   - 라인 번호, 파일 경로, 에러 코드 확인

2. **일관되게 재현**
   - 정확한 단계는?
   - 매번 발생하나?

3. **최근 변경 확인**
   - Git diff, 최근 커밋
   - 새 의존성, 설정 변경

4. **데이터 흐름 추적**
   - 잘못된 값이 어디서 시작되나?
   - 소스를 찾을 때까지 추적

### Phase 2: Pattern Analysis

1. **작동하는 예제 찾기**
2. **참조와 비교**
3. **차이점 식별**

### Phase 3: Hypothesis and Testing

1. **단일 가설 형성** - "X가 근본 원인이라고 생각합니다, 왜냐하면 Y"
2. **최소한으로 테스트** - 한 번에 하나의 변수만
3. **계속하기 전 검증**

### Phase 4: Implementation

1. **Failing Test Case 생성**
2. **단일 수정 구현**
3. **수정 검증**
4. **3회 이상 실패 시: 아키텍처 의심**

## Red Flags - STOP

- "일단 빠른 수정, 조사는 나중에"
- "X 바꿔보고 되나 보자"
- "여러 변경 추가하고 테스트"
- 조사 없이 해결책 제안
- 2회 이상 시도 후 "한 번 더 수정"

**모두: 멈추세요. Phase 1로 돌아가세요.**

## GitHub 연동

**디버깅 과정 문서화:**

```bash
# 이슈 코멘트로 디버깅 진행상황 기록
gh issue comment [issue-number] --body "
## Debugging Progress

### Root Cause Investigation
- Error: [에러 내용]
- Reproduction: [재현 단계]

### Hypothesis
- [가설]

### Fix
- [수정 내용]
"
```

## 관련 스킬

- **test-driven-development**: 수정용 failing test 생성
- **verification**: 수정 후 검증
