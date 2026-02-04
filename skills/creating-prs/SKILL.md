---
name: creating-prs
description: Use when implementation is complete and verified, ready to create pull request for code review
---

# Pull Request 생성

구현 완료 후 코드 리뷰를 위한 PR을 생성합니다.

## 전제 조건

**REQUIRED:** verification 스킬로 먼저 검증 완료

- 모든 테스트 통과
- 린터 통과
- 빌드 성공

## PR 생성

```bash
gh pr create \
  --title "feat: [feature description]" \
  --body "$(cat <<'EOF'
## Summary

[변경 사항 요약]

## Changes

- [변경 1]
- [변경 2]

## Testing

- [x] Unit tests
- [x] Integration tests
- [x] Manual testing

## Checklist

- [x] Tests pass
- [x] Lint clean
- [x] Build succeeds

Closes #[issue-number]
EOF
)"
```

## Issue 연결

**커밋 메시지에 이슈 번호:**
```bash
git commit -m "feat: add feature

Closes #123"
```

**PR 본문에 이슈 번호:**
```markdown
Closes #123
```

## PR 체크리스트

- [ ] verification 스킬로 검증 완료
- [ ] 테스트 추가/수정
- [ ] 문서 업데이트 (필요시)
- [ ] 이슈 번호 연결
- [ ] 리뷰어 지정

## 관련 스킬

- **verification**: PR 전 검증
- **test-driven-development**: 테스트 작성
- **closing-issues**: PR 머지 후 이슈 종료
