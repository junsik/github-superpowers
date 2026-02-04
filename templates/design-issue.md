# Design Issue Template

brainstorming 스킬에서 Design Issue 생성 시 사용합니다.

## 사용법

```bash
gh issue create \
  --title "design: <feature-name>" \
  --body-file docs/plans/YYYY-MM-DD-<topic>-design.md \
  --label "design" \
  --milestone "$MILESTONE_TITLE"
```

## 구조

Design Issue의 본문은 design.md 파일 내용 전체입니다.

design.md 파일은 다음 구조를 따릅니다:

```markdown
# <Feature Name> Design

## Overview
[무엇을 만드는지 1-2 문장]

## Goals
- [목표 1]
- [목표 2]

## Non-Goals
- [하지 않을 것]

## Architecture
[아키텍처 설명]

## Components
### Component 1
[설명]

### Component 2
[설명]

## Data Flow
[데이터 흐름]

## Error Handling
[에러 처리 전략]

## Testing Strategy
[테스트 전략]

## Open Questions
- [ ] [결정 필요한 사항]

## Decision Log
| 날짜 | 결정 | 이유 |
|------|------|------|
| YYYY-MM-DD | [결정 내용] | [이유] |
```

## Labels

- `design` - 설계 문서

## 다음 단계

Design Issue 생성 후:
1. design.md 헤더에 Issue 링크 추가
2. Project Roadmap에 추가
3. writing-plans 스킬로 impl.md 작성
