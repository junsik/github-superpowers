---
name: closing-issues
description: Use when PR is merged or work is complete - closes GitHub issues with proper documentation
---

# Issue 종료

PR 머지 또는 작업 완료 후 이슈를 종료합니다.

## 자동 종료 (권장)

PR 머지 시 자동으로 이슈 종료:

```markdown
# PR 본문 또는 커밋 메시지에
Closes #[epic-number]
Fixes #[epic-number]
Resolves #[epic-number]
```

## 수동 종료

```bash
gh issue close [epic-number] --comment "
## Completion Summary

- PR: #[pr-number]
- Changes: [변경 사항 요약]
- Testing: All tests pass
"
```

## Epic 종료 전 확인

Epic의 체크리스트가 모두 완료되었는지 확인:

```bash
# Epic 상태 확인
gh issue view [epic-number]
```

**체크리스트 완료 후:**
- PR에서 `Closes #[epic-number]` 또는
- 수동으로 `gh issue close`

## 종료 체크리스트

- [ ] Epic 체크리스트 모두 완료
- [ ] PR 머지 확인
- [ ] 테스트 통과 확인
- [ ] 완료 코멘트 작성

## 관련 스킬

- **creating-issues**: Epic 생성
- **creating-prs**: PR 생성
- **verification**: 완료 전 검증
