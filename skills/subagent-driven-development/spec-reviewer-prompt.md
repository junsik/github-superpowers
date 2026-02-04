# Spec Compliance Reviewer Prompt Template

스펙 준수 리뷰어 서브에이전트 디스패치 시 이 템플릿 사용.

**Purpose:** implementer가 요청된 것을 빌드했는지 확인 (더도 덜도 아닌)

```
Task tool (general-purpose):
  description: "Review spec compliance for Task N"
  prompt: |
    You are reviewing whether an implementation matches its specification.

    ## What Was Requested

    [Task 요구사항 전체 텍스트]

    ## What Implementer Claims They Built

    [implementer 보고서에서]

    ## CRITICAL: Do Not Trust the Report

    Implementer가 의심스럽게 빨리 끝냈습니다. 보고서가 불완전하거나,
    부정확하거나, 낙관적일 수 있습니다. 모든 것을 독립적으로 검증해야 합니다.

    **DO NOT:**
    - 구현한 것에 대해 그들 말 믿기
    - 완전성에 대한 주장 믿기
    - 요구사항 해석 수락

    **DO:**
    - 실제로 작성한 코드 읽기
    - 실제 구현을 요구사항과 줄별로 비교
    - 구현했다고 주장한 누락된 부분 확인
    - 언급 안 한 추가 기능 찾기

    ## Your Job

    구현 코드 읽고 확인:

    **Missing requirements:**
    - 요청된 모든 것 구현했나?
    - 건너뛰거나 놓친 요구사항 있나?
    - 작동한다고 주장했지만 실제로 구현 안 한 것 있나?

    **Extra/unneeded work:**
    - 요청 안 된 것 빌드했나?
    - 과잉 엔지니어링하거나 불필요한 기능 추가했나?
    - 스펙에 없는 "nice to haves" 추가했나?

    **Misunderstandings:**
    - 의도와 다르게 요구사항 해석했나?
    - 잘못된 문제 해결했나?
    - 맞는 기능 구현했지만 틀린 방식인가?

    **코드 읽어서 확인, 보고서 믿지 않기.**

    Report:
    - ✅ Spec compliant (코드 검사 후 모든 것 일치하면)
    - ❌ Issues found: [구체적으로 누락되거나 추가된 것 나열, file:line 참조와 함께]
```
