---
name: spec-code-review
description: 코드 리뷰 스킬. "코드 리뷰", "코드리뷰", "리뷰해줘", "review", "/spec-code-review" 등으로 트리거. code-reviewer 에이전트를 spawn하여 변경된 코드를 정확성/보안/성능/품질 관점에서 체계적으로 점검한다.
---

# 코드 리뷰 스킬

code-reviewer 에이전트를 spawn하여 변경된 코드를 체계적으로 리뷰한다.

**워크플로우 참조**: `~/.claude/skills/spec/references/workflow.md`
**리뷰 체크리스트**: `~/.claude/skills/spec/references/code-review-checklist.md`

---

## 실행 절차

1. `~/.claude/agents/code-reviewer.md` 파일을 읽는다.
2. `~/.claude/skills/spec/references/code-review-checklist.md` 파일을 읽는다.
3. Agent 도구를 사용하여 code-reviewer 에이전트를 spawn한다.

```
Agent 도구 파라미터:
- subagent_type: "general-purpose"
- description: "code-reviewer - 코드 리뷰"
- prompt: |
    [code-reviewer.md 내용]

    ---
    ## 리뷰 체크리스트
    [code-review-checklist.md 내용]

    ---
    ## 현재 작업
    현재 작업 디렉토리의 변경된 코드를 리뷰해주세요.

    ## 작업 디렉토리
    [현재 작업 디렉토리 경로]

    ## 추가 컨텍스트
    [사용자가 제공한 추가 정보가 있으면 포함]
```

4. 에이전트의 리뷰 결과를 사용자에게 그대로 전달한다.

## 결과 판정

리뷰 결과를 판정한다:

- **blocker 0개** → "리뷰 통과. 커밋/푸시 가능합니다." + issue/suggestion 목록
- **blocker 1개+** → 개선 사이클 진입 필요 (workflow.md 참조)
