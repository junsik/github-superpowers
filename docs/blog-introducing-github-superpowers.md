# GitHub Superpowers: Claude Code로 설계부터 PR까지 자동 추적하기

AI 코딩 어시스턴트를 사용하다 보면 한 가지 아쉬운 점이 있습니다. 코드는 잘 작성해주는데, **프로젝트 관리는 여전히 수동**이라는 점입니다.

"이 기능 구현해줘" → 코드 작성 → 커밋 → 이슈 업데이트 → PR 생성...

이 과정에서 GitHub Issue 업데이트는 잊어버리고, 나중에 "이게 어떤 이슈랑 연결된 거였지?" 하며 히스토리를 뒤지게 됩니다.

**GitHub Superpowers**는 이 문제를 해결합니다.

## 무엇인가?

GitHub Superpowers는 [Superpowers](https://github.com/obra/superpowers) 워크플로우에 GitHub 프로젝트 관리를 통합한 **Claude Code 플러그인**입니다.

설계 문서 작성부터 구현 계획, GitHub Issue 생성, TDD 구현, PR 생성까지 **전 과정을 자동으로 추적**합니다.

```
요청 → brainstorming → design.md → Design Issue (#1)
                            ↓
                    writing-plans → impl.md
                            ↓
                    creating-issues → Epic (#2) with Task checklist
                            ↓
                    executing-plans → TDD per Task → Epic checklist 자동 체크
                            ↓
                    verification → PR → Issue Close
```

## 왜 만들었나?

Superpowers는 훌륭한 워크플로우입니다. brainstorming으로 설계를 다듬고, TDD로 구현하고, 검증 후 완료하는 체계적인 프로세스를 제공합니다.

하지만 한 가지 빠진 게 있었습니다: **GitHub 연동**.

- 설계 문서는 로컬에만 존재
- 구현 진행 상황을 GitHub에서 추적할 수 없음
- 커밋과 이슈의 연결이 수동

GitHub Superpowers는 이 간극을 메웁니다:

1. **design.md 저장 시 자동으로 Design Issue 생성**
2. **impl.md의 Task들을 Epic Issue의 체크리스트로 변환**
3. **Task 완료 시 Epic 체크리스트 자동 체크**
4. **커밋 메시지에 자동으로 Issue 참조 추가**

## 핵심 기능

### 1. Design Issue 자동 생성

brainstorming 스킬로 설계를 완료하면:

```markdown
<!-- .claude/github-superpowers/plans/2024-01-15-user-auth-design.md -->
# User Authentication Design

> GitHub Issue: #42

## Overview
...
```

design.md에 Issue 링크가 자동 삽입되고, GitHub에 Design Issue가 생성됩니다.

### 2. Epic with Task Checklist

impl.md 작성 후 creating-issues 스킬이 자동 실행:

```markdown
<!-- GitHub Issue #43 -->
# [Epic] User Authentication

## Tasks
- [ ] Task 1: Setup auth module structure
- [ ] Task 2: Implement JWT service
- [ ] Task 3: Create login endpoint
- [ ] Task 4: Add refresh token logic

Closes #42  <!-- Design Issue 참조 -->
```

### 3. 진행 상황 자동 추적

executing-plans 스킬로 Task를 완료할 때마다:

```markdown
- [x] Task 1: Setup auth module structure  <!-- 자동 체크! -->
- [x] Task 2: Implement JWT service
- [ ] Task 3: Create login endpoint
- [ ] Task 4: Add refresh token logic
```

GitHub Issue가 실시간으로 업데이트됩니다.

### 4. 스택별 패턴 제공

프로젝트 스택에 맞는 best practices를 스킬로 제공:

- **nextjs-frontend**: FSD 아키텍처 + Zustand + React Query + shadcn/ui
- **nestjs-backend**: Hexagonal Architecture + UseCase 패턴 + BullMQ
- **fastapi-backend**: FastAPI + Celery + Async 패턴

## 설치

```bash
# Marketplace 등록
/plugin marketplace add junsik/github-superpowers

# 플러그인 설치
/plugin install github-superpowers@github-superpowers-marketplace
```

### 초기 설정

```bash
/init-github-superpowers
```

이 명령어로:
- GitHub Labels 생성 (design, epic)
- GitHub Project 생성 (커스텀 필드 포함)
- `.github/github-superpowers.json` 설정 파일 생성

## 실제 사용 예시

### 1. 새 기능 요청

```
"사용자 인증 기능을 추가해줘"
```

Claude가 자동으로 brainstorming 스킬 발동:

```
brainstorming 스킬을 사용하여 설계를 시작합니다.

질문 1: 인증 방식은 JWT vs Session 중 어떤 것을 선호하시나요?
질문 2: OAuth 소셜 로그인이 필요한가요?
...
```

### 2. 설계 완료 → Design Issue 자동 생성

```
설계가 완료되었습니다.
- 저장: .claude/github-superpowers/plans/2024-01-15-user-auth-design.md
- GitHub Issue: #42 (design label)
```

### 3. 구현 계획 → Epic 자동 생성

```
impl.md 작성이 완료되었습니다.
- 저장: .claude/github-superpowers/plans/2024-01-15-user-auth-impl.md
- GitHub Epic: #43 (4개 Task 체크리스트)
```

### 4. TDD 구현 → 체크리스트 자동 업데이트

```
Task 1 완료: Setup auth module structure
- 테스트 통과: 3/3
- Epic #43 체크리스트 업데이트 완료
```

### 5. PR 생성 → Issue 자동 Close

```
PR #44 생성 완료
- Title: feat: implement user authentication
- Closes #43, Closes #42
```

## 워크플로우 철학

GitHub Superpowers는 다음 원칙을 따릅니다:

1. **Test-Driven Development** - 테스트 먼저, 항상
2. **GitHub as Source of Truth** - GitHub Issue로 진행 상태 추적
3. **자동화된 연결** - 설계 → 구현 → GitHub 자동 연동
4. **Evidence over Claims** - 성공 선언 전 검증

## 마치며

GitHub Superpowers는 AI 코딩 어시스턴트의 빈틈을 메웁니다.

코드 작성뿐 아니라 **프로젝트 관리까지 자동화**하여, 개발자가 정말 중요한 일에 집중할 수 있게 합니다.

---

**Links:**
- GitHub: https://github.com/junsik/github-superpowers
- 기반: [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent

**설치:**
```bash
/plugin marketplace add junsik/github-superpowers
/plugin install github-superpowers@github-superpowers-marketplace
```
