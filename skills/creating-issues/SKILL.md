---
name: creating-issues
description: Use after writing-plans completes impl.md - creates GitHub Epic with task checklist based on the implementation plan
---

# GitHub Epic 생성

`writing-plans` 스킬에서 생성된 impl.md 파일을 기반으로 Epic 이슈를 생성합니다.

**Announce at start:** "creating-issues 스킬을 사용하여 Epic 이슈를 생성합니다."

## Plan → Epic 변환

- Design Issue → Epic에서 `Closes #N`으로 참조
- Goal/Architecture → Epic 설명
- 각 Task → 체크리스트 항목 (`- [ ] Task`)

## 워크플로우

```dot
digraph issue_creation {
    "impl.md" [shape=box];
    "Epic 생성" [shape=box, style=filled, fillcolor=lightgreen];
    "프로젝트 연결" [shape=box];
    "impl.md 업데이트" [shape=box];
    "TDD 구현" [shape=doublecircle];

    "impl.md" -> "Epic 생성";
    "Epic 생성" -> "프로젝트 연결";
    "프로젝트 연결" -> "impl.md 업데이트";
    "impl.md 업데이트" -> "TDD 구현";
}
```

## Epic 생성

```bash
# 설정 파일에서 값 읽기
PROJECT_OWNER=$(jq -r '.project.owner' .github/github-superpowers.json)
PROJECT_NUMBER=$(jq -r '.project.number' .github/github-superpowers.json)
MILESTONE_TITLE=$(jq -r '.milestones.current' .github/github-superpowers.json)

# Epic body (Task를 체크리스트로)
EPIC_BODY="## 설계 문서
Closes #[design-issue-number]

## 구현 계획
[impl.md 요약]

## Tasks
- [ ] Task 1: [설명]
- [ ] Task 2: [설명]
- [ ] Task 3: [설명]
"

EPIC_URL=$(gh issue create \
  --title "epic: <feature-name>" \
  --body "$EPIC_BODY" \
  --label "epic,feat" \
  --milestone "$MILESTONE_TITLE")

# Project에 추가
gh project item-add $PROJECT_NUMBER \
  --owner $PROJECT_OWNER \
  --url "$EPIC_URL"
```

## impl.md 업데이트

Epic 생성 후 impl.md 헤더에 Epic 번호/URL 자동 추가:

```bash
# Epic 번호 추출
EPIC_NUMBER=$(echo "$EPIC_URL" | grep -oE '[0-9]+$')

# impl.md 헤더에 Epic 링크 추가 (헤더 다음 줄에)
sed -i '1a\
**GitHub Epic:** #'"$EPIC_NUMBER"' ('"$EPIC_URL"')' docs/plans/YYYY-MM-DD-<feature-name>-impl.md

# 변경사항 커밋
git add docs/plans/YYYY-MM-DD-<feature-name>-impl.md
git commit -m "docs: link epic #$EPIC_NUMBER to impl.md"
```

**결과 예시:**
```markdown
# [Feature Name] Implementation Plan
**GitHub Epic:** #42 (https://github.com/owner/repo/issues/42)

> **For Claude:** REQUIRED SUB-SKILL: Use executing-plans...
```

## Closing 정책

| 상황 | 방법 |
|------|------|
| Task 완료 | Epic 체크리스트에서 수동 체크 |
| Epic 완료 | PR에서 `Closes #epic` |

## 커밋 메시지

```bash
git commit -m "feat: add feature component

Refs #[epic-number]"
```

마지막 커밋 또는 PR에서:
```bash
git commit -m "feat: complete feature implementation

Closes #[epic-number]"
```

## 다음 단계

Epic 생성 완료 후:
- **REQUIRED:** Use test-driven-development 스킬
- impl.md의 Task 순서대로 TDD 진행
- 각 Task 완료 시 Epic 체크리스트 업데이트

## 관련 스킬

- **writing-plans**: impl.md 생성 (이전 단계)
- **test-driven-development**: TDD 구현 (다음 단계)
- **creating-prs**: PR 생성
