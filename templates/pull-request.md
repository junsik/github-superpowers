# Pull Request Template

creating-prs 스킬에서 PR 생성 시 사용합니다.

## 사용법

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary

- <변경 사항 1>
- <변경 사항 2>
- <변경 사항 3>

## Related Issues

Closes #<epic-number>

## Changes

### Added
- <추가된 기능/파일>

### Changed
- <변경된 기능/파일>

### Fixed
- <수정된 버그>

## Test Plan

- [ ] 단위 테스트 통과
- [ ] 통합 테스트 통과
- [ ] 수동 테스트 완료

## Checklist

- [ ] 코드가 스타일 가이드를 따름
- [ ] 셀프 리뷰 완료
- [ ] 테스트 추가/업데이트
- [ ] 문서 업데이트 (필요시)

## Screenshots (해당시)

<스크린샷>
EOF
)"
```

## 구조

```markdown
## Summary

- [변경 사항 요약 bullet points]

## Related Issues

Closes #<epic-number>

## Changes

### Added
- [추가된 기능/파일]

### Changed
- [변경된 기능/파일]

### Fixed
- [수정된 버그]

## Test Plan

- [ ] 단위 테스트 통과
- [ ] 통합 테스트 통과
- [ ] 수동 테스트 완료

## Checklist

- [ ] 코드가 스타일 가이드를 따름
- [ ] 셀프 리뷰 완료
- [ ] 테스트 추가/업데이트
- [ ] 문서 업데이트 (필요시)

## Screenshots (해당시)

[UI 변경이 있으면 스크린샷]
```

## PR 제목 컨벤션

```
<type>: <short description>

Types:
- feat: 새로운 기능
- fix: 버그 수정
- refactor: 리팩토링
- docs: 문서
- test: 테스트
- chore: 기타
```

## 예시

```bash
gh pr create \
  --title "feat: add user authentication" \
  --body "$(cat <<'EOF'
## Summary

- JWT 기반 인증 시스템 추가
- 로그인/로그아웃 API 구현
- 인증 미들웨어 추가

## Related Issues

Closes #42

## Changes

### Added
- src/auth/jwt.ts - JWT 토큰 생성/검증
- src/middleware/auth.ts - 인증 미들웨어
- src/routes/auth.ts - 인증 라우트

### Changed
- src/app.ts - 인증 미들웨어 등록

## Test Plan

- [x] 단위 테스트 통과 (15 tests)
- [x] 통합 테스트 통과 (8 tests)
- [x] 수동 테스트 완료

## Checklist

- [x] 코드가 스타일 가이드를 따름
- [x] 셀프 리뷰 완료
- [x] 테스트 추가/업데이트
- [x] 문서 업데이트
EOF
)"
```

## 다음 단계

PR 머지 후:
- Epic 자동 종료 (Closes # 키워드)
- worktree 정리 (finishing-a-development-branch)
