---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# Finishing a Development Branch

## Overview

개발 작업 완료를 안내하고 명확한 옵션을 제시하여 선택된 워크플로우를 처리합니다.

**Core principle:** 테스트 확인 → 옵션 제시 → 선택 실행 → 정리.

**Announce at start:** "finishing-a-development-branch 스킬을 사용하여 작업을 완료합니다."

## The Process

### Step 1: Verify Tests

**옵션 제시 전에 테스트 통과 확인:**

```bash
# 프로젝트 테스트 스위트 실행
npm test / cargo test / pytest / go test ./...
```

**테스트 실패 시:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

멈추기. Step 2로 진행 금지.

**테스트 통과 시:** Step 2로 진행.

### Step 2: Determine Base Branch

```bash
# 일반적인 베이스 브랜치 시도
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

또는 질문: "이 브랜치는 main에서 분기했습니다 - 맞나요?"

### Step 3: Present Options

정확히 이 4가지 옵션 제시:

```
구현 완료. 어떻게 하시겠습니까?

1. <base-branch>로 로컬 머지
2. 푸시하고 Pull Request 생성
3. 브랜치 유지 (나중에 처리)
4. 작업 폐기

어떤 옵션을 선택하시겠습니까?
```

**설명 추가 금지** - 옵션을 간결하게 유지.

### Step 4: Execute Choice

#### Option 1: Merge Locally

```bash
# 베이스 브랜치로 전환
git checkout <base-branch>

# 최신 풀
git pull

# feature 브랜치 머지
git merge <feature-branch>

# 머지 결과 테스트 확인
<test command>

# 테스트 통과 시
git branch -d <feature-branch>
```

Then: Worktree 정리 (Step 5)

#### Option 2: Push and Create PR

```bash
# 브랜치 푸시
git push -u origin <feature-branch>

# PR 생성
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<변경 사항 2-3줄 요약>

## Test Plan
- [ ] <검증 단계>

Closes #<epic-number>
EOF
)"
```

Then: Worktree 정리 (Step 5)

#### Option 3: Keep As-Is

보고: "브랜치 <name> 유지. Worktree <path>에 보존됨."

**Worktree 정리 금지.**

#### Option 4: Discard

**먼저 확인:**
```
영구 삭제됩니다:
- 브랜치 <name>
- 모든 커밋: <commit-list>
- Worktree at <path>

'discard' 입력하여 확인.
```

정확한 확인 대기.

확인되면:
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

Then: Worktree 정리 (Step 5)

### Step 5: Cleanup Worktree

**Options 1, 2, 4의 경우:**

worktree 확인:
```bash
git worktree list | grep $(git branch --show-current)
```

있으면:
```bash
git worktree remove <worktree-path>
```

**Option 3의 경우:** Worktree 유지.

## Quick Reference

| Option | Merge | Push | Keep Worktree | Cleanup Branch |
|--------|-------|------|---------------|----------------|
| 1. Merge locally | ✓ | - | - | ✓ |
| 2. Create PR | - | ✓ | ✓ | - |
| 3. Keep as-is | - | - | ✓ | - |
| 4. Discard | - | - | - | ✓ (force) |

## Common Mistakes

**테스트 검증 건너뛰기**
- **Problem:** 깨진 코드 머지, 실패하는 PR 생성
- **Fix:** 옵션 제시 전 항상 테스트 확인

**열린 질문**
- **Problem:** "다음에 뭘 할까요?" → 모호함
- **Fix:** 정확히 4가지 구조화된 옵션 제시

**자동 worktree 정리**
- **Problem:** 필요할 수 있는 worktree 제거 (Option 2, 3)
- **Fix:** Option 1과 4에서만 정리

**폐기 확인 없음**
- **Problem:** 실수로 작업 삭제
- **Fix:** 'discard' 타이핑 확인 필요

## Red Flags

**Never:**
- 실패하는 테스트로 진행
- 결과 테스트 확인 없이 머지
- 확인 없이 작업 삭제
- 명시적 요청 없이 force-push

**Always:**
- 옵션 제시 전 테스트 확인
- 정확히 4가지 옵션 제시
- Option 4에서 타이핑 확인 받기
- Option 1 & 4에서만 worktree 정리

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - 모든 Task 완료 후
- **executing-plans** (Step 5) - 모든 배치 완료 후

**Pairs with:**
- **using-git-worktrees** - 해당 스킬이 생성한 worktree 정리

## 관련 스킬

- **using-git-worktrees**: worktree 생성
- **creating-prs**: PR 생성 (Option 2)
- **closing-issues**: Epic 종료
