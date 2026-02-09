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
- [ ] Serena 메모리 업데이트 (해당 시)

## Serena 메모리 업데이트 (Optional)

Serena MCP가 활성화된 프로젝트라면, 구현 완료 후 축적된 지식을 메모리에 반영합니다.
다음 세션에서 설계/구현 시 코드베이스를 처음부터 다시 탐색하지 않도록 해줍니다.

**업데이트가 필요한 경우:**
- 새 모듈/엔티티/서비스가 추가된 경우
- 아키텍처 패턴이나 컨벤션이 변경된 경우
- 중요한 설계 결정이 내려진 경우

```
# 1. 기존 메모리 확인
list_memories

# 2. 변경사항에 맞는 메모리 업데이트
# 새 모듈 추가 → codebase_structure 업데이트
edit_memory("codebase_structure", mode="literal",
  needle="[관련 섹션]",
  repl="[업데이트된 내용]")

# 아키텍처/패턴 변경 → architecture_and_conventions 업데이트
edit_memory("architecture_and_conventions", mode="literal",
  needle="[관련 섹션]",
  repl="[업데이트된 내용]")

# 중요한 설계 결정 → 새 메모리 생성 또는 기존 메모리 추가
write_memory("design_decisions_YYYY_MM_DD",
  content="# 설계 결정사항\n\n- [결정 내용과 근거]")
```

**업데이트 불필요:** 단순 버그 수정, 리팩토링 등 구조 변경이 없는 경우 건너뜁니다.

## 관련 스킬

- **creating-issues**: Epic 생성
- **creating-prs**: PR 생성
- **verification**: 완료 전 검증
