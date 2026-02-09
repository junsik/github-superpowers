---
name: verification
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - evidence before assertions always
---

# Verification Before Completion

## Overview

**Why verify?** "통과할 것이다"라는 가정은 코드 변경의 부작용을 놓칩니다. 실제 명령 실행 결과만이 현재 상태의 증거입니다. 캐시된 기억이나 추론은 증거가 아닙니다.

**Core principle:** 증거가 주장보다 먼저.

## 핵심 원칙

완료를 주장하기 전에 검증 명령을 새로 실행하세요. 이전 실행 결과는 코드 변경 후 유효하지 않습니다.

## The Gate Function

```
상태 주장이나 만족 표현 전에:

1. IDENTIFY: 이 주장을 증명하는 명령은?
2. RUN: 전체 명령 실행 (새로, 완전하게)
3. READ: 전체 출력, exit code, 실패 수 확인
4. VERIFY: 출력이 주장을 확인하나?
   - NO: 증거와 함께 실제 상태 보고
   - YES: 증거와 함께 주장
5. ONLY THEN: 주장하기

어떤 단계든 건너뛰기 = 거짓
```

## Common Failures

| 주장 | 필요 | 불충분 |
|------|------|--------|
| 테스트 통과 | 테스트 명령 출력: 0 failures | 이전 실행, "통과할 것" |
| 빌드 성공 | 빌드 명령: exit 0 | 린터 통과, 로그 괜찮아 보임 |
| 버그 수정됨 | 원래 증상 테스트: 통과 | 코드 변경, 수정됐다고 가정 |

## 주의 신호

검증이 불충분할 수 있는 징후:

- **"should", "probably" 같은 추측성 언어** → 실행 결과가 아닌 추론에 의존 중
- **부분 검증만 수행** → 한 테스트 통과가 전체 통과를 보장하지 않음
- **검증 전 만족 표현** → 결과 확인 전 결론을 내린 것

## GitHub 연동

**PR 생성 전 필수 검증 (백그라운드 병렬 실행):**

검증 명령들은 서로 독립적이므로 **백그라운드로 동시 실행**하여 시간을 절약합니다:

```bash
# 3개 검증을 백그라운드로 동시 실행
Bash(command: "npm test", run_in_background: true)        # → task_id_1
Bash(command: "npm run lint", run_in_background: true)     # → task_id_2
Bash(command: "npm run build", run_in_background: true)    # → task_id_3

# 각 결과 수집 (완료될 때까지 대기)
TaskOutput(task_id: task_id_1)  # 테스트 결과
TaskOutput(task_id: task_id_2)  # 린터 결과
TaskOutput(task_id: task_id_3)  # 빌드 결과
```

**모든 검증 통과 후 PR 생성:**

```bash
gh pr create --title "feat: feature name" --body "
## Verification

- [x] Tests: 34/34 pass
- [x] Lint: 0 errors
- [x] Build: exit 0

Closes #[issue-number]
"
```

**하나라도 실패하면:**
- 실패한 검증의 전체 출력 확인
- 수정 후 **모든 검증을 다시 실행** (부분 검증 금지)
- 전부 통과할 때까지 반복

## The Bottom Line

명령 실행 → 출력 확인 → 결과 주장. 이 순서가 신뢰할 수 있는 유일한 경로입니다.

## 관련 스킬

- **test-driven-development**: 테스트 작성
- **creating-prs**: PR 생성
