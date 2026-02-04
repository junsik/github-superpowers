---
name: verification
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - evidence before assertions always
---

# Verification Before Completion

## Overview

검증 없이 완료를 주장하는 것은 효율이 아니라 거짓입니다.

**Core principle:** 증거가 주장보다 먼저, 항상.

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

이 메시지에서 검증 명령을 실행하지 않았다면, 통과를 주장할 수 없습니다.

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

## Red Flags - STOP

- "should", "probably", "seems to" 사용
- 검증 전 만족 표현 ("Great!", "Perfect!", "Done!")
- 검증 없이 커밋/푸시/PR
- 부분 검증에 의존
- "이번 한 번만" 생각
- 피곤해서 끝내고 싶음

## GitHub 연동

**PR 생성 전 필수 검증:**

```bash
# 1. 테스트 실행
npm test

# 2. 린터 실행
npm run lint

# 3. 빌드 실행
npm run build

# 4. 모두 통과 확인 후 PR 생성
gh pr create --title "feat: feature name" --body "
## Verification

- [x] Tests: 34/34 pass
- [x] Lint: 0 errors
- [x] Build: exit 0

Closes #[issue-number]
"
```

## The Bottom Line

**검증에 지름길 없음.**

명령 실행. 출력 읽기. 그 다음 결과 주장.

협상 불가.

## 관련 스킬

- **test-driven-development**: 테스트 작성
- **creating-prs**: PR 생성
