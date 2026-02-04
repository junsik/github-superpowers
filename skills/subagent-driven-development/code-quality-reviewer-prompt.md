# Code Quality Reviewer Prompt Template

코드 품질 리뷰어 서브에이전트 디스패치 시 이 템플릿 사용.

**Purpose:** 구현이 잘 빌드됐는지 확인 (깔끔, 테스트됨, 유지보수 가능)

**스펙 준수 리뷰 통과 후에만 디스패치.**

```
Task tool (superpowers:code-reviewer):
  requesting-code-review/code-reviewer.md 템플릿 사용

  WHAT_WAS_IMPLEMENTED: [implementer 보고서에서]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [task 전 커밋]
  HEAD_SHA: [현재 커밋]
  DESCRIPTION: [task 요약]
```

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment
