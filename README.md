# GitHub Superpowers

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

GitHub Superpowers는 [Superpowers](https://github.com/obra/superpowers) 워크플로우에 GitHub 프로젝트 관리를 통합한 Claude Code 플러그인입니다.

설계 → 구현 계획 → GitHub Issue → TDD 구현 → PR까지 전 과정을 자동으로 추적합니다.

## 주요 특징

- **GitHub 통합**: Design Issue, Epic, Milestone, Project 자동 연동
- **체크리스트 기반 Task 관리**: Epic 이슈에 Task를 체크리스트로 관리
- **자동 이슈 링크**: design.md, impl.md에 GitHub Issue 링크 자동 삽입
- **스택별 패턴**: Next.js (FSD), NestJS (Hexagonal), FastAPI 패턴 제공
- **gh CLI 기반**: 모든 GitHub 작업을 gh CLI로 수행

## 설치

### Claude Code

marketplace 등록 후 설치:

```bash
/plugin marketplace add junsik/github-superpowers
```

```bash
/plugin install github-superpowers@github-superpowers
```

### 설치 확인

```bash
/help
```

```
# 다음이 보여야 합니다:
# /github-superpowers:setup - 초기 설정
# /github-superpowers:milestone - 마일스톤 관리
```

### 초기 설정

처음 사용 시 `/init-github-superpowers` 명령어로 초기화:

```bash
/init-github-superpowers
```

이 명령어는:
- gh CLI 인증 확인
- GitHub Labels 생성 (design, epic)
- GitHub Project 생성 (커스텀 필드 포함)
- `.github/github-superpowers.json` 설정 파일 생성

## 워크플로우

```
요청 분석
    ↓
brainstorming → design.md → Design Issue (#N)
    ↓
writing-plans → impl.md
    ↓
creating-issues → Epic (#M) with Task checklist
    ↓
executing-plans → TDD per Task → Epic checklist update
    ↓
verification → creating-prs → Issue Close
```

### 1. Brainstorming (설계)

아이디어를 구체적인 설계로 발전시킵니다.

- 질문을 통한 요구사항 정제
- `docs/plans/YYYY-MM-DD-<topic>-design.md` 저장
- **자동으로 Design Issue 생성** (label: design)
- design.md에 Issue 링크 자동 추가

### 2. Writing Plans (구현 계획)

설계를 상세 구현 계획으로 분해합니다.

- Task별 2-5분 단위 bite-sized 스텝
- 정확한 파일 경로, 완전한 코드
- `docs/plans/YYYY-MM-DD-<feature>-impl.md` 저장

### 3. Creating Issues (GitHub 연동)

impl.md 기반으로 Epic 이슈를 자동 생성합니다.

- Task를 체크리스트로 포함
- Design Issue 참조 (`Closes #N`)
- Milestone, Project 연결
- impl.md에 Epic 링크 자동 추가

### 4. Executing Plans (구현)

각 Task를 TDD로 실행하고 Epic 체크리스트를 업데이트합니다.

- **using-git-worktrees**: 격리된 작업 공간
- **test-driven-development**: RED-GREEN-REFACTOR
- 커밋 메시지에 `Refs #<epic>` 포함
- Task 완료 시 Epic 체크리스트 자동 체크

### 5. Finishing (완료)

PR 생성 및 이슈 종료:

- **verification**: 완료 전 최종 검증
- **creating-prs**: PR에 `Closes #<epic>` 포함
- **finishing-a-development-branch**: 머지/PR/폐기 선택

## 스킬 목록

### 초기 설정
| 스킬 | 설명 |
|------|------|
| **setup** | Project, Labels 초기화 (처음 1회) |
| **milestone** | 마일스톤 생성/전환/종료 |

### 설계 → 구현 계획
| 스킬 | 설명 |
|------|------|
| **brainstorming** | 아이디어 → design.md → Design Issue |
| **writing-plans** | design.md → impl.md (Task 분해) |

### GitHub 추적
| 스킬 | 설명 |
|------|------|
| **creating-issues** | impl.md → Epic (체크리스트로 Task 관리) |
| **creating-prs** | PR 생성 |
| **closing-issues** | 이슈 종료 |

### 구현
| 스킬 | 설명 |
|------|------|
| **using-git-worktrees** | 격리된 작업 공간 생성 |
| **executing-plans** | impl.md Task별 TDD 실행 |
| **subagent-driven-development** | 서브에이전트 기반 개발 |
| **dispatching-parallel-agents** | 독립적 Task 병렬 실행 |
| **test-driven-development** | TDD 사이클 |
| **systematic-debugging** | 체계적 디버깅 |
| **verification** | 완료 전 검증 |

### 코드 리뷰
| 스킬 | 설명 |
|------|------|
| **requesting-code-review** | 코드 리뷰 요청 |
| **receiving-code-review** | 코드 리뷰 피드백 처리 |

### 완료
| 스킬 | 설명 |
|------|------|
| **finishing-a-development-branch** | 개발 브랜치 완료 (머지/PR/폐기) |

### 스택별 패턴 (참조용)
| 스킬 | 설명 |
|------|------|
| **nextjs-frontend** | FSD 아키텍처 + Zustand + React Query + shadcn/ui |
| **nestjs-backend** | Hexagonal Architecture + UseCase 패턴 + BullMQ |
| **fastapi-backend** | FastAPI + Celery + Async 패턴 |

## Templates

`templates/` 폴더에 GitHub Issue 및 PR 템플릿이 있습니다:

| 템플릿 | 용도 | 사용 스킬 |
|--------|------|----------|
| `design-issue.md` | Design Issue 생성 | brainstorming |
| `epic-issue.md` | Epic Issue 생성 (Task 체크리스트) | creating-issues |
| `pull-request.md` | PR 생성 | creating-prs |

## 설정 파일

`.github/github-superpowers.json`:

```json
{
  "project": {
    "owner": "<owner>",
    "number": 1,
    "fields": {
      "startDate": "Start Date",
      "endDate": "End Date",
      "priority": "Priority"
    }
  },
  "milestones": {
    "current": "v1.0.0",
    "strategy": "version"
  },
  "labels": {
    "design": "design",
    "epic": "epic"
  }
}
```

## Commands

| 명령어 | 설명 |
|--------|------|
| `/init-github-superpowers` | 초기 설정 (Project, Labels) |
| `/milestone` | 마일스톤 관리 |

## 전제 조건

- [Claude Code](https://claude.ai/code) 설치
- `gh CLI` 설치 및 인증 (`gh auth login`)
- GitHub 저장소 연결

## 철학

- **Test-Driven Development** - 테스트 먼저, 항상
- **GitHub as Source of Truth** - GitHub Issue로 진행 상태 추적
- **자동화된 연결** - 설계 → 구현 → GitHub 자동 연동
- **Evidence over Claims** - 성공 선언 전 검증

## 기반

이 플러그인은 [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent를 기반으로 합니다.

## 기여

이슈나 PR은 언제든 환영합니다.

## 라이선스

MIT License - [LICENSE](LICENSE) 참조
