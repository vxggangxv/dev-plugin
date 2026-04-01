---
name: spec-execute
description: 스펙 문서 작성/갱신 후 개발 에이전트로 구현까지 자동 진행. "스펙 실행", "spec execute", "스펙 작성하고 개발", "스펙문서 갱신", "스펙 갱신", "spec 갱신" 등으로 트리거. /spec-execute로도 호출 가능.
---

# Spec → Execute 워크플로우

세션 컨텍스트를 기반으로 스펙 문서를 작성/갱신하고, 바로 개발 에이전트로 구현까지 진행한다.
**질문 없이 즉시 실행한다.**

**워크플로우 참조**: `~/.claude/skills/spec/references/workflow.md`
**리뷰 체크리스트**: `~/.claude/skills/spec/references/code-review-checklist.md`

기능명: $ARGUMENTS

---

## Phase 0: 분기 판단

1. `$ARGUMENTS`에서 기능명 추출. 비어있으면 세션 컨텍스트에서 추론.
2. `specs/` 하위에서 기능명과 일치하는 폴더 탐색
   - **폴더가 없는 경우** → Phase 1A (신규 생성)
   - **폴더가 있는 경우** → Phase 1B (갱신)
3. "갱신", "업데이트", "update" 키워드가 포함된 경우 → 기존 폴더를 찾아 Phase 1B 강제 진행

---

## Phase 1A: 스펙 신규 생성

1. 폴더명: `YYYYMMDD-기능명`
2. `specs/codebase/index.md`를 읽어 관련 feature 파악
3. 세션에서 논의된 내용(요구사항, 기술 결정, 제약사항, 변경 파일)을 수집
4. 다음 세 파일을 즉시 생성 (확인 없이):

### spec.md
```markdown
---
name: [기능명]
related_features: [관련 feature 목록]
status: planned
created: YYYY-MM-DD
---

# [기능명] Specification

## User Story
- As a [사용자 유형]
- I want [원하는 기능]
- So that [얻고자 하는 가치]

## Acceptance Criteria
- [ ] [세션에서 논의된 기준 1]
- [ ] [세션에서 논의된 기준 2]

## Technical Constraints
- [사용 기술 스택]
- [제약사항]

## Data Structure / API
- [논의된 데이터 구조]
- [API 엔드포인트]

## Implementation Notes
- [세션에서 발견한 특이사항]
```

### plan.md
```markdown
# [기능명] Implementation Plan

## Overview
[한 줄 요약]

## Tasks
1. [ ] [작업 1] - [대상 파일/모듈]
2. [ ] [작업 2] - [대상 파일/모듈]
3. [ ] [작업 3] - [대상 파일/모듈]

## Technical Approach
- [선택된 방식과 이유]

## Dependencies
- [관련 feature/모듈]

## Estimated Impact
- [수정되는 파일/모듈]
- [주의할 점]
```

### context.md
```markdown
# [기능명] Context

## Current Status
- Phase: Planning
- Last Updated: YYYY-MM-DD

## Summary
- [세션에서 수행한 작업 요약]

## Files Changed
- (아직 없음)

## Session History
- YYYY-MM-DD: 스펙 문서 생성 via spec-execute
```

---

## Phase 1B: 스펙 갱신

기존 `specs/[폴더명]/` 내의 **spec.md, plan.md, context.md** 세 파일을 모두 읽고, 세션에서 새로 논의/변경된 내용을 반영하여 갱신한다.

1. `specs/[폴더명]/spec.md`, `plan.md`, `context.md` 순서대로 읽기
2. 세션에서 변경된 내용 수집:
   - 새로운 요구사항, 수정된 acceptance criteria
   - 변경된 기술 결정, 추가된 제약사항
   - 완료된 작업, 새로 발견된 작업
   - 변경된 파일 목록 (git diff 참조)
3. **세 파일 모두 갱신**:
   - `spec.md`: 요구사항, acceptance criteria, 기술 제약 업데이트. status 변경 (planned → in_progress → completed)
   - `plan.md`: 완료된 작업 체크, 새 작업 추가, 접근 방식 변경 반영
   - `context.md`: Current Status 업데이트, Files Changed 추가, Session History에 갱신 이력 추가

---

## Phase 2: 개발 실행

스펙 문서 생성/갱신 즉시 개발 에이전트를 실행한다.

1. `spec.md`와 `plan.md`를 기반으로 작업 영역 자동 판별
   - 백엔드 → `backend-dev`
   - 웹 프론트 → `frontend-web-dev`
   - 앱 프론트 → `frontend-app-dev`
   - 복합 → 병렬 또는 순차 실행

2. `~/.claude/skills/use-agents/references/AGENT_SYSTEM.md`의 실행 방법을 따라 에이전트 실행

3. 에이전트에게 전달할 컨텍스트:
   - `specs/[폴더명]/spec.md` 전체 내용
   - `specs/[폴더명]/plan.md` 전체 내용
   - 작업 대상 파일 목록
   - 세션에서 논의된 기술적 결정사항

---

---

## Phase 3: 스펙 문서 갱신 (spec-write)

개발 에이전트 작업 완료 후, 변경 내용을 스펙 문서에 반영한다.

1. 변경된 파일 목록, 완료된 작업 반영
2. context.md 상태 업데이트
3. 갱신 완료 후 Phase 4로 이어짐

---

## Phase 4: 코드 리뷰 (spec-code-review)

code-reviewer 에이전트를 spawn하여 변경된 코드를 리뷰한다.

1. `~/.claude/agents/code-reviewer.md` 읽기
2. `~/.claude/skills/spec/references/code-review-checklist.md` 읽기
3. Agent 도구로 code-reviewer 에이전트 spawn
4. 리뷰 결과 판정:
   - **blocker 0개** → Phase 5로 진행
   - **blocker 1개+** → 개선 사이클 진입 (Phase 6)

---

## Phase 5: 테스트 실행 (spec-testing)

1. `plan.md`에서 테스트 시나리오 확인
2. 작업 영역에 맞는 테스터 에이전트 실행:
   - 백엔드 → `spec-testing back`
   - 웹 프론트 → `spec-testing web`
   - 앱 프론트 → `spec-testing app`
3. plan.md의 테스트 시나리오가 있으면 시나리오 기반으로 전달
4. 테스트 결과 판정:
   - **전체 통과** → 완료 보고
   - **실패 존재** → 개선 사이클 진입 (Phase 6)

---

## Phase 6: 자가 평가 및 개선 사이클

코드 리뷰 또는 테스트에서 문제가 발견되면 자동으로 개선 사이클에 진입한다.
상세 규칙은 `~/.claude/skills/spec/references/workflow.md`를 따른다.

### 흐름

```
[문제 발견] → [원인 파악] → [수정 (개발 에이전트)]
     → [spec-write] → [spec-code-review] → [spec-testing]
     → [통과?] → Yes → 완료
        ↓ No
     [재진입] (최대 3회)
```

### 규칙
- 최대 **3회** 반복
- 3회 후에도 미해결 시 사용자에게 보고하고 수동 판단 요청
- 각 사이클마다 개선 보고 출력

---

## 실행 흐름

```
/spec-execute [기능명]
     ↓
[Phase 0] 폴더 존재 확인 → 신규 생성 or 갱신
     ↓
[Phase 1/1B] spec.md + plan.md + context.md 생성/갱신
     ↓
[Phase 2] 작업 영역 판별 → 개발 에이전트 실행
     ↓
[Phase 3] spec-write → 스펙 문서 갱신
     ↓
[Phase 4] spec-code-review → 코드 리뷰
     ↓
[Phase 5] spec-testing → 테스트 실행
     ↓
[통과?] → Yes → 완료 보고
   ↓ No
[Phase 6] 개선 사이클 → Phase 3으로 재진입 (최대 3회)
```

## 완료 보고

```
━━━━━━━━━━━━━━━━━━━━━━━━━
Spec → Execute 완료
━━━━━━━━━━━━━━━━━━━━━━━━━

  기능: [기능명]
  모드: [신규 생성 | 갱신]
  스펙: specs/[폴더명]/
    - spec.md
    - plan.md
    - context.md
  에이전트: [사용된 에이전트]
  개선 사이클: [N]회 수행
  코드 리뷰: blocker 0 | issue N | suggestion N
  테스트: 전체 통과
  수정 파일: [파일 목록]
━━━━━━━━━━━━━━━━━━━━━━━━━
```
