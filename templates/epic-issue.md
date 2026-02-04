# Epic Issue Template

creating-issues 스킬에서 Epic Issue 생성 시 사용합니다.

## 사용법

```bash
EPIC_BODY=$(cat <<'EOF'
## 설계 문서

Closes #<design-issue-number>

## 목표

<impl.md에서 Goal 요약>

## 아키텍처

<impl.md에서 Architecture 요약>

## Tasks

- [ ] Task 1: <설명>
- [ ] Task 2: <설명>
- [ ] Task 3: <설명>
- [ ] Task 4: <설명>

## 구현 계획

상세 계획: `docs/plans/YYYY-MM-DD-<feature>-impl.md`

## 검증 기준

- [ ] 모든 테스트 통과
- [ ] 코드 리뷰 완료
- [ ] 문서화 완료
EOF
)

gh issue create \
  --title "epic: <feature-name>" \
  --body "$EPIC_BODY" \
  --label "epic" \
  --milestone "$MILESTONE_TITLE"
```

## 구조

```markdown
## 설계 문서

Closes #<design-issue-number>

## 목표

- [목표 1]
- [목표 2]

## 아키텍처

[간략한 아키텍처 설명]

## Tasks

- [ ] Task 1: [설명]
- [ ] Task 2: [설명]
- [ ] Task 3: [설명]

## 구현 계획

상세 계획: `docs/plans/YYYY-MM-DD-<feature>-impl.md`

## 검증 기준

- [ ] 모든 테스트 통과
- [ ] 코드 리뷰 완료
- [ ] 문서화 완료
```

## Labels

- `epic` - 구현 Epic

## Task 체크리스트 업데이트

Task 완료 시:

```bash
# Epic body에서 해당 Task 체크
CURRENT_BODY=$(gh issue view $EPIC_NUMBER --json body -q .body)
UPDATED_BODY=$(echo "$CURRENT_BODY" | sed 's/- \[ \] Task N/- [x] Task N/')
gh issue edit $EPIC_NUMBER --body "$UPDATED_BODY"
```

## 커밋 메시지

```bash
# 작업 중
git commit -m "feat: <description>

Refs #<epic-number>"

# 마지막 (PR에서)
git commit -m "feat: complete <feature>

Closes #<epic-number>"
```

## 다음 단계

Epic 생성 후:
1. impl.md 헤더에 Epic 링크 추가
2. Project Roadmap에 추가
3. executing-plans 스킬로 구현 시작
