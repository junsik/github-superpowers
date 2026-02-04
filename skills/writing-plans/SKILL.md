---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

구현 **가이드**를 작성합니다. 무엇을 어떤 순서로 할지 명확히 정의합니다.

**impl.md ≠ 코드 생성기**
- 전체 구현 코드 작성 X
- 핵심 인터페이스/타입 시그니처만
- 접근법과 완료 기준에 집중

**Announce at start:** "writing-plans 스킬을 사용하여 구현 계획을 작성합니다."

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>-impl.md`

## Bite-Sized Task Granularity

**각 스텝은 하나의 액션 (2-5분):**
- "failing test 작성" - 스텝
- "실행하여 실패 확인" - 스텝
- "최소 코드로 테스트 통과" - 스텝
- "테스트 실행 및 통과 확인" - 스텝
- "커밋" - 스텝

## Plan Document Header

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use executing-plans to implement this plan task-by-task.

**Goal:** [한 문장으로 무엇을 만드는지]

**Architecture:** [2-3 문장으로 접근법]

**Tech Stack:** [핵심 기술/라이브러리]

**GitHub Issue:** #[issue-number] (Epic)

---
```

## Task Structure

**CRITICAL: impl.md는 구현 가이드이지, 구현 코드가 아닙니다.**

```markdown
### Task N: [Component Name]

**목표:** [이 Task가 달성하는 것 - 1문장]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py` (함수명 또는 라인 범위)
- Test: `tests/exact/path/to/test.py`

**접근법:**
1. [무엇을 먼저 하고]
2. [그 다음 무엇을 하고]
3. [최종적으로 무엇을 확인]

**핵심 인터페이스:** (선택 - 복잡한 경우만)
```python
class SomeClass:
    def method(self, arg: Type) -> ReturnType:
        """한 줄 설명."""
        ...
```

**테스트 케이스:**
- `test_happy_path`: 정상 동작 확인
- `test_error_case`: 에러 처리 확인

**완료 기준:**
- [ ] 테스트 통과
- [ ] 린터 통과
```

**코드 스니펫 규칙:**
- 시그니처 + docstring 수준만 (구현부는 `...` 또는 `pass`)
- 전체 구현 코드 작성 금지 - `executing-plans`에서 작성
- 복잡한 알고리즘이면 의사코드로 설명

## After the Plan

**impl.md 완성 후 자동으로 GitHub 연동:**

1. **REQUIRED:** Use creating-issues 스킬
2. Epic 이슈 생성 (Task를 체크리스트로 포함)
3. 프로젝트/마일스톤 연결
4. impl.md에 Epic 번호 업데이트

```dot
digraph after_plan {
    "impl.md 완성" [shape=box];
    "creating-issues" [shape=box, style=filled, fillcolor=lightgreen];
    "Epic 생성" [shape=box];
    "구현 시작" [shape=doublecircle];

    "impl.md 완성" -> "creating-issues" [label="자동"];
    "creating-issues" -> "Epic 생성";
    "Epic 생성" -> "구현 시작";
}
```

**구현 시작 시:**
- **REQUIRED:** Use test-driven-development 스킬
- 각 Task별로 TDD 사이클 진행
- 커밋 메시지에 `Refs #[epic-number]` 포함
- 마지막 커밋/PR에서 `Closes #[epic-number]`

## 컨텍스트 복구 (/clear 후 시작 시)

```
/clear 후 사용자 입력 예시:
- "#42 구현 계획 세워줘"
- "design.md 기반으로 impl.md 만들어줘"

복구 순서:
1. GitHub Issue 번호가 있으면 → `gh issue view #N` 으로 내용 로드
2. 파일 경로가 있으면 → 해당 design.md 읽기
3. 둘 다 없으면 → `docs/plans/` 에서 최신 *-design.md 찾기
```

## 완료 후 (AskUserQuestion)

impl.md 저장 + Epic 생성 후:

```
AskUserQuestion:
"구현 계획이 완료되었습니다.
- 저장: docs/plans/YYYY-MM-DD-<feature>-impl.md
- GitHub Epic: #M (N개 Task)

다음 단계는?"

옵션:
1. `/clear` 후 구현 시작 → 새 세션에서 "#M 구현해줘"
2. 이어서 구현 (컨텍스트 유지, 서브에이전트 사용)
3. 오늘은 여기까지
```

## Remember

- 정확한 파일 경로
- **구현 가이드** (전체 코드 X) - 코드는 `executing-plans`에서
- 핵심 인터페이스/타입만 스니펫으로
- Task당 목표 1개, 명확한 완료 기준
- impl.md 목표 길이: 200-500줄 (2000줄 넘으면 분할 고려)

## 관련 스킬

- **brainstorming**: 계획 전 설계
- **creating-issues**: Epic 생성
- **test-driven-development**: TDD 구현
