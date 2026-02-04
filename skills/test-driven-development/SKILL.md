---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---

# Test-Driven Development (TDD)

## Overview

테스트를 먼저 작성합니다. 실패를 확인합니다. 최소 코드로 통과시킵니다.

**Core principle:** 테스트 실패를 확인하지 않으면, 올바른 것을 테스트하는지 알 수 없습니다.

**규칙의 문자를 어기는 것은 규칙의 정신을 어기는 것입니다.**

## The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

테스트 전에 코드를 작성했나요? 삭제하세요. 처음부터 시작하세요.

**예외 없음:**
- "참고용"으로 보관 금지
- 테스트 작성하면서 "수정" 금지
- 보지도 마세요
- 삭제는 삭제입니다

## Red-Green-Refactor

```dot
digraph tdd_cycle {
    rankdir=LR;
    red [label="RED\nFailing test", shape=box, style=filled, fillcolor="#ffcccc"];
    green [label="GREEN\nMinimal code", shape=box, style=filled, fillcolor="#ccffcc"];
    refactor [label="REFACTOR\nClean up", shape=box, style=filled, fillcolor="#ccccff"];

    red -> green [label="실패 확인"];
    green -> refactor [label="통과 확인"];
    refactor -> red [label="다음 테스트"];
}
```

### RED - Failing Test 작성

하나의 최소 테스트 작성.

```typescript
test('retries failed operations 3 times', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };
  const result = await retryOperation(operation);
  expect(result).toBe('success');
  expect(attempts).toBe(3);
});
```

### Verify RED - 실패 확인

**필수. 절대 건너뛰지 마세요.**

```bash
npm test path/to/test.test.ts
```

확인:
- 테스트 실패 (에러가 아님)
- 실패 메시지가 예상대로
- 기능 누락으로 실패 (오타 아님)

### GREEN - Minimal Code

테스트를 통과시키는 가장 간단한 코드.

```typescript
async function retryOperation<T>(fn: () => Promise<T>): Promise<T> {
  for (let i = 0; i < 3; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === 2) throw e;
    }
  }
  throw new Error('unreachable');
}
```

### REFACTOR - Clean Up

GREEN 이후에만:
- 중복 제거
- 이름 개선
- 헬퍼 추출

테스트는 GREEN 유지. 동작 추가 금지.

## Red Flags - STOP and Start Over

- 테스트 전에 코드
- 구현 후 테스트
- 테스트가 바로 통과
- "나중에" 테스트 추가
- "이번 한 번만" 합리화
- "이미 수동 테스트했어"

**이 모든 것은: 코드 삭제. TDD로 다시 시작.**

## 진행 관리

Task 관리는 Claude가 상황에 맞게 선택:
- **Task tool** (subagent) - 복잡한 멀티스텝 작업
- **TodoWrite** - 간단한 체크리스트

## GitHub 연동

**커밋 메시지에 Epic 번호 포함:**

```bash
# 작업 중
git commit -m "feat: add retry operation

Refs #[epic-number]"

# 마지막 (PR 또는 최종 커밋)
git commit -m "feat: complete retry feature

Closes #[epic-number]"
```

## 관련 스킬

- **writing-plans**: TDD 전 상세 계획
- **systematic-debugging**: 버그 수정 시 TDD와 함께
- **verification**: 완료 전 검증
