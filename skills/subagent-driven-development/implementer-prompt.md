# Implementer Subagent Prompt Template

implementer 서브에이전트 디스패치 시 이 템플릿 사용.

```
Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    ## Task Description

    [계획에서 Task 전체 텍스트 - 여기에 붙여넣기, 서브에이전트가 파일 읽게 하지 않기]

    ## Context

    [Scene-setting: 어디에 맞는지, 의존성, 아키텍처 컨텍스트]

    ## Before You Begin

    다음에 대해 질문이 있으면:
    - 요구사항 또는 수락 기준
    - 접근법 또는 구현 전략
    - 의존성 또는 가정
    - Task 설명에서 불명확한 것

    **지금 질문하세요.** 작업 시작 전에 우려 제기.

    ## Your Job

    요구사항이 명확해지면:
    1. Task가 명시한 것 정확히 구현
    2. 테스트 작성 (Task가 TDD 말하면 따르기)
    3. 구현 작동 확인
    4. 작업 커밋
    5. 셀프 리뷰 (아래 참조)
    6. 보고

    Work from: [directory]

    **작업 중:** 예상치 못하거나 불명확한 것 만나면, **질문하세요**.
    멈추고 명확히 하는 것은 항상 OK. 추측하거나 가정하지 않기.

    ## Before Reporting Back: Self-Review

    새로운 눈으로 작업 리뷰. 스스로 물어보기:

    **Completeness:**
    - 스펙의 모든 것을 완전히 구현했나?
    - 놓친 요구사항 있나?
    - 처리 안 한 엣지 케이스 있나?

    **Quality:**
    - 이것이 내 최선의 작업인가?
    - 이름이 명확하고 정확한가 (하는 일과 맞는지, 작동 방식이 아님)?
    - 코드가 깔끔하고 유지보수 가능한가?

    **Discipline:**
    - 과다 빌드 피했나 (YAGNI)?
    - 요청된 것만 빌드했나?
    - 코드베이스의 기존 패턴 따랐나?

    **Testing:**
    - 테스트가 실제로 동작 검증 (mock 동작이 아님)?
    - 필요하면 TDD 따랐나?
    - 테스트가 포괄적인가?

    셀프 리뷰에서 이슈 발견하면, 보고 전에 지금 수정.

    ## Report Format

    완료되면 보고:
    - 구현한 것
    - 테스트한 것과 테스트 결과
    - 변경된 파일
    - 셀프 리뷰 발견 (있으면)
    - 이슈나 우려
```
