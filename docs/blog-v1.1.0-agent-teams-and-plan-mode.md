# GitHub Superpowers v1.1.0: Agent Team 오케스트레이션과 Plan Mode

v1.0.0에서 GitHub Superpowers는 설계부터 PR까지 전 과정을 자동 추적하는 워크플로우를 제공했습니다.

하지만 실제로 사용하면서 두 가지 한계가 드러났습니다:

1. **구현 계획 중에 실수로 코드를 수정할 수 있다** — 탐색만 해야 하는 단계에서 Edit/Write가 열려 있었음
2. **에이전트들이 서로 대화할 수 없다** — 리뷰어가 구현자에게 직접 피드백을 줄 수 없고, 항상 리드를 거쳐야 했음

v1.1.0은 Claude Code의 최신 기능을 활용하여 이 두 가지 문제를 해결합니다.

## 무엇이 바뀌었나?

### 1. Plan Mode로 구현 계획 안전하게

**Before (v1.0.0):**

```
writing-plans 스킬 시작
→ 코드베이스 탐색 (Edit/Write 가능 — 위험!)
→ AskUserQuestion으로 수동 승인
→ impl.md 저장
```

탐색 단계에서 Edit과 Write 도구가 열려 있어서, 실수로 코드를 수정할 위험이 있었습니다. 승인 게이트도 AskUserQuestion에 의존하는 수동 방식이었습니다.

**After (v1.1.0):**

```
EnterPlanMode
→ 코드베이스 탐색 (Edit/Write 물리적 차단!)
→ plan file에 구현 계획 작성
→ ExitPlanMode (사용자 승인 게이트)
→ 승인 후 impl.md 저장
```

Plan Mode에 들어가면 **Edit과 Write 도구가 물리적으로 차단**됩니다. 읽기 전용으로 코드베이스를 탐색하고, 계획을 작성한 뒤, ExitPlanMode로 사용자 승인을 받습니다. 승인 후에야 impl.md가 저장됩니다.

### 2. Agent Team 파이프라인

**Before (v1.0.0):**

```
lead → Task(implementer) → 결과 수신
lead → Task(spec-reviewer, 구현 결과 전달) → 결과 수신
lead → Task(quality-reviewer, 리뷰 결과 전달) → 결과 수신
```

모든 소통이 lead를 거쳤습니다. 리뷰어가 구현자에게 피드백을 주려면 lead가 중계해야 했고, 그 과정에서 컨텍스트가 손실되었습니다.

**After (v1.1.0):**

```
TeamCreate("impl-feature")
  ├─ implementer (general-purpose) ←→ spec-reviewer (Explore)
  │                                 ←→ quality-reviewer (Explore)
  └─ lead: 완료 알림만 수신
```

TeamCreate로 팀을 구성하면, 에이전트들이 **SendMessage로 직접 소통**합니다:

```
implementer 구현 완료
  → SendMessage("spec-reviewer", "Task 1 구현 완료. 리뷰 부탁.")
spec-reviewer 리뷰 완료
  → SendMessage("quality-reviewer", "스펙 준수 확인. 품질 리뷰 부탁.")
quality-reviewer 승인
  → SendMessage("lead", "Task 1 파이프라인 통과.")
```

리뷰에서 이슈 발견 시에도 직접 피드백:

```
spec-reviewer 이슈 발견
  → SendMessage("implementer", "반환 타입이 스펙과 다릅니다. file:line 참조.")
implementer 수정 후
  → SendMessage("spec-reviewer", "수정 완료. 재리뷰 부탁.")
```

lead가 중계할 필요 없이, **피드백 루프가 즉시 작동**합니다.

### 3. 리뷰어는 코드를 수정할 수 없다

리뷰어 에이전트의 타입을 **Explore**로 설정했습니다.

```
implementer: general-purpose (Edit/Write 가능)
spec-reviewer: Explore (읽기 전용 — Edit/Write 물리적 차단)
quality-reviewer: Explore (읽기 전용 — Edit/Write 물리적 차단)
```

Explore 타입은 Edit, Write, NotebookEdit 도구가 제공되지 않습니다. 리뷰어가 "이렇게 고치는 게 나을 것 같은데" 하며 직접 코드를 수정하는 일이 **원천 차단**됩니다.

리뷰어의 역할은 코드를 읽고 판단하는 것이지, 수정하는 것이 아닙니다.

### 4. Agent Team 병렬 실행 + 소통

**Before (v1.0.0):**

```
Task(agent-a, "Domain A 수정") → fire-and-forget
Task(agent-b, "Domain B 수정") → fire-and-forget
Task(agent-c, "Domain C 수정") → fire-and-forget
... 전부 끝날 때까지 대기 ...
결과 수집
```

에이전트들이 서로의 존재를 몰랐습니다. Domain A를 수정하다가 Domain B에 영향을 주는 문제를 발견해도, 알릴 방법이 없었습니다.

**After (v1.1.0):**

```
TeamCreate("parallel-fix")

agent-a: Domain A 작업 중 관련 이슈 발견
  → SendMessage("agent-b", "abort flow에서 batch state 관련 이슈 발견. 확인 필요.")
agent-b: 알림 수신 후 자체적으로 대응

agent-a 완료 → SendMessage("lead", "Domain A 완료. 변경 파일: ...")
agent-b 완료 → SendMessage("lead", "Domain B 완료. agent-a 알림 반영함.")
agent-c 완료 → SendMessage("lead", "Domain C 완료.")
```

팀 안에서 에이전트들이 **관련 이슈를 발견하면 즉시 공유**할 수 있습니다.

### 5. Task 의존성 분석

executing-plans 스킬이 이제 **impl.md의 Task 간 의존성을 분석**합니다:

```
의존성 분석 결과:
- 독립 그룹 A: Task 1 (DB 스키마), Task 3 (프론트엔드) — 병렬 가능
- 순차 체인 B: Task 2 → Task 4 (Task 1 이후)
- 순차 체인 C: Task 5 (Task 2, 3 이후)

추천: Agent Team 병렬 (독립 Task를 동시 실행, 의존 Task는 순차)
```

분석 결과에 따라 3가지 실행 방식 중 최적을 추천합니다:

| 방식 | 적합한 경우 |
|------|------------|
| Agent Team 파이프라인 | Task 간 의존성이 많을 때 (순차 + 리뷰) |
| Agent Team 병렬 | 독립 Task가 50% 이상일 때 (동시 실행) |
| 수동 실행 (워크트리) | 단계별 직접 제어가 필요할 때 |

### 6. 백그라운드 병렬 검증

**Before (v1.0.0):**

```
npm test        → 대기... → 완료
npm run lint    → 대기... → 완료
npm run build   → 대기... → 완료
(총 3번 순차 대기)
```

**After (v1.1.0):**

```bash
# 3개 검증을 백그라운드로 동시 실행
Bash(command: "npm test", run_in_background: true)        # → task_id_1
Bash(command: "npm run lint", run_in_background: true)     # → task_id_2
Bash(command: "npm run build", run_in_background: true)    # → task_id_3

# 각 결과 수집
TaskOutput(task_id: task_id_1)  # 테스트 결과
TaskOutput(task_id: task_id_2)  # 린터 결과
TaskOutput(task_id: task_id_3)  # 빌드 결과
```

검증 명령들은 서로 독립적이므로, `run_in_background`로 동시 실행하여 **대기 시간을 1/3로 단축**합니다.

## 변경된 스킬 전체 목록

| 스킬 | 변경 내용 |
|------|----------|
| **writing-plans** | EnterPlanMode/ExitPlanMode 통합 |
| **subagent-driven-development** | TeamCreate + SendMessage 파이프라인, Explore 리뷰어 |
| **dispatching-parallel-agents** | Agent Team 병렬 + 에이전트 간 소통 |
| **executing-plans** | Task 의존성 분석 + 3가지 실행 모드 |
| **verification** | 백그라운드 병렬 검증 |
| **using-github-superpowers** | Plan Mode, Agent Team 설명 추가 |

## 업그레이드

```bash
/plugin update github-superpowers
```

기존 설정(`.github/github-superpowers.json`)은 변경 없이 그대로 사용됩니다.

## 철학의 확장

v1.0.0의 4가지 원칙에 하나가 추가되었습니다:

- **Test-Driven Development** — 테스트 먼저, 항상
- **GitHub as Source of Truth** — GitHub Issue로 진행 상태 추적
- **자동화된 연결** — 설계 → 구현 → GitHub 자동 연동
- **Evidence over Claims** — 성공 선언 전 백그라운드 병렬 검증
- **Agent Team Orchestration** — 에이전트 간 직접 소통으로 협업 ← NEW

AI 에이전트가 혼자 일하는 시대에서, **팀으로 협업하는 시대**로 넘어갑니다.

---

**Links:**
- GitHub: https://github.com/junsik/github-superpowers
- CHANGELOG: [CHANGELOG.md](../CHANGELOG.md)
- 기반: [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent
