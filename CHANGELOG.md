# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-02-09

### Added

- **Design-Aware Task 구조** (`writing-plans`): impl.md Task에 `Design Reference`, `구현 스펙`, `수용 기준` 필드 추가 — 에이전트가 design.md 없이도 구현 가능하도록 스펙 인라인
- **Plan Stress Test** (`writing-plans`): Step 3.5에 셀프체크 게이트 추가 — 구현 충분성, 스펙 완전성, AC 분배, 테스트 가능성 4개 항목 검증 후 ExitPlanMode
- **MVP Check-in Gate** (`executing-plans`): 전체 Task의 ~50% 시점에서 설계 정렬 확인하는 AskUserQuestion 체크인 추가
- **Design Document 로드** (`executing-plans`): Step 0에 design.md 사전 읽기 단계 추가, Agent Team 사용 시 lead가 관련 스펙을 Task 할당 메시지에 포함
- **Design Document 확인** (`implementer-prompt.md`): 매 Task 시작 전 Design Reference 필드 및 design.md 링크 확인 절차 추가
- **작업 규모별 경로 분기** (`using-github-superpowers`): Quick (1-2 파일) / Standard (3-5 파일) / Full (6+ 파일) 경로 라우팅

### Changed

- **Opus 4.6 톤 최적화** (`using-github-superpowers`): `<EXTREMELY-IMPORTANT>` 강제 호출 제거 → 자율 판단 기반 스킬 선택, Red Flags 테이블 → 일반 가이드라인
- **Opus 4.6 톤 최적화** (`session-start.sh`): `EXTREMELY_IMPORTANT` → `IMPORTANT` 태그
- **Why-first 규칙** (`test-driven-development`): Iron Law / Red Flags → "Why TDD?" 이유 설명 + 주의 신호
- **Why-first 규칙** (`verification`): Iron Law / Red Flags → "Why verify?" 이유 설명 + 주의 신호
- **Code Snippet Rules** (`writing-plans`): DTO/인터페이스는 전체 필드 정의 필수, 구현 로직은 시그니처만, 줄 수 제한 → 충실도 기준으로 전환

## [1.1.0] - 2025-02-07

### Added

- **Plan Mode 통합** (`writing-plans`): EnterPlanMode/ExitPlanMode를 활용하여 구현 계획 시 읽기 전용 탐색 강제 및 사용자 승인 게이트 적용
- **Agent Team 파이프라인** (`subagent-driven-development`): TeamCreate + SendMessage로 implementer → spec-reviewer → quality-reviewer 파이프라인 구성
- **Agent Team 병렬 실행** (`dispatching-parallel-agents`): 에이전트 간 SendMessage로 직접 소통 가능한 병렬 실행
- **Task 의존성 분석** (`executing-plans`): impl.md Task 간 의존성 분석 후 최적 실행 방식(파이프라인/병렬/수동) 자동 추천
- **백그라운드 병렬 검증** (`verification`): `run_in_background` + `TaskOutput`으로 test, lint, build 동시 실행
- **Explore 에이전트 타입**: 리뷰어(spec-reviewer, quality-reviewer)에 읽기 전용 에이전트 적용으로 코드 수정 물리적 차단

### Changed

- `writing-plans`: 자체 탐색 + AskUserQuestion 방식 → Plan Mode (Edit/Write 차단) + ExitPlanMode 승인 방식으로 전환
- `subagent-driven-development`: Task 순차 발사 + lead 중계 → Agent Team 파이프라인 + 에이전트 간 직접 피드백 루프
- `dispatching-parallel-agents`: fire-and-forget Task 병렬 → TeamCreate 기반 에이전트 팀 + 관련 이슈 발견 시 SendMessage 소통
- `executing-plans`: 단일 실행 방식 → 3가지 실행 모드 선택 (Agent Team 파이프라인, Agent Team 병렬, 수동 워크트리)
- `verification`: 순차 검증 → 백그라운드 병렬 검증 (3개 명령 동시 실행)
- `using-github-superpowers`: Plan Mode 및 Agent Team 관련 설명 추가
- 리뷰어 프롬프트 전면 개편 (implementer-prompt.md, spec-reviewer-prompt.md, code-quality-reviewer-prompt.md)

## [1.0.0] - 2025-02-01

### Added

- 초기 릴리스
- GitHub Superpowers 워크플로우 (brainstorming → writing-plans → creating-issues → executing-plans → verification → creating-prs)
- GitHub Issue, Epic, Milestone, Project 자동 연동
- 체크리스트 기반 Task 관리
- TDD 사이클 (test-driven-development)
- 체계적 디버깅 (systematic-debugging)
- Git worktree 격리 작업 (using-git-worktrees)
- 스택별 패턴: Next.js (FSD), NestJS (Hexagonal), FastAPI
- 코드 리뷰 워크플로우 (requesting-code-review, receiving-code-review)
- gh CLI 기반 모든 GitHub 작업
