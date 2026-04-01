---
name: kickoff
description: Quick Kickoff Command
argument-hint: <기능명>
disable-model-invocation: true
---
name: kickoff

# Quick Kickoff Command
기능명: "$ARGUMENTS"

이 커맨드는 작업 내용을 입력받아 **Plan Mode로 진입**하여 계획을 세우고, 생성된 plan.md를 기반으로 **spec.md를 역추출**합니다.

---
name: kickoff

## 1. 초기화

1. `$ARGUMENTS`에서 기능명을 추출하십시오.
   - 첫 번째 줄: 기능명
   - 나머지 줄: 작업 내용 설명

2. 폴더명을 생성하십시오.
   - 형식: `YYYYMMDD-기능명`
   - 예시: `20260105-workspace-member-usage`
   - 오늘 날짜를 사용하십시오.

3. `specs/[폴더명]/` 폴더가 없으면 생성하십시오.

4. 동일한 기능명으로 시작하는 폴더가 이미 존재하면:
   - 기존 `spec.md`, `plan.md`, `context.md`를 로드
   - 현재 상황을 요약하고 계속 진행할지 새로 시작할지 물어보십시오.

---
name: kickoff

## 2. Plan Mode 진입

**Claude Code의 Plan Mode로 진입하십시오.**

Plan Mode에서 다음을 수행:
- 코드베이스 탐색 (Glob, Grep, Read 도구 활용)
- 기존 패턴과 아키텍처 파악
- 구현 전략 설계
- **계획을 `specs/[폴더명]/plan.md`에 작성**
- 사용자에게 계획 제시 및 승인 요청

---
name: kickoff

## 3. spec.md 역추출 (Plan Mode 완료 후)

Plan Mode가 완료되면:

1. **`specs/[폴더명]/plan.md` 파일을 읽으십시오.**

2. plan.md 내용을 분석하여 `specs/[폴더명]/spec.md`를 역추출 작성하십시오:

```markdown
# [기능명] Specification

## User Story
- (plan.md의 Problem 섹션에서 추출)

## Acceptance Criteria
- [ ] (plan.md의 Milestones에서 핵심 목표 추출)
- [ ] (각 Phase의 최종 결과물 기준)

## Technical Constraints
- (plan.md의 Selected Strategy에서 기술적 제약 추출)

## Data Structure / API
- (plan.md에서 언급된 데이터 구조/API)

## Out of Scope
- (plan.md에 포함되지 않은 명시적 제외 항목)
```

3. `specs/[폴더명]/context.md` 초기화:

```markdown
# [기능명] Context

## Current Status
- Phase: Planning Complete
- Last Updated: [날짜]

## Session History
- [날짜]: Initial planning completed
```

---
name: kickoff

## 4. 마무리

모든 문서 생성 후 다음과 같이 출력:

```
Documents created:
  - specs/[폴더명]/plan.md (구현 계획) - from Plan Mode
  - specs/[폴더명]/spec.md (요구사항) - extracted from plan.md
  - specs/[폴더명]/context.md (상태 추적)

예시: specs/20260108-workspace-member-usage/

Ready to implement Phase 1?
```

**중단**: 사용자 승인을 받은 후에만 코드 작성을 시작하십시오.
