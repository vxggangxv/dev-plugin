---
description: Recap Command
argument-hint: <기능명>
disable-model-invocation: true
---

# Recap Command
기능명: "$ARGUMENTS"

이 커맨드는 **이미 작업이 완료된 상태**에서 현재 변경사항을 분석하여 `plan.md`, `spec.md`, `context.md`를 역으로 생성합니다.

---

## 1. 초기화

1. `$ARGUMENTS`에서 기능명을 추출하십시오.
   - 첫 번째 줄: 기능명
   - 나머지 줄: 작업 내용 설명 (선택사항)

2. 폴더명을 생성하십시오.
   - 형식: `YYYYMMDD-기능명`
   - 예시: `20260108-workspace-member-usage`
   - 오늘 날짜를 사용하십시오.

3. `specs/[폴더명]/` 폴더를 생성하십시오.

---

## 1.5 코드베이스 컨텍스트 파악 (Codebase Context)

- `specs/codebase/index.md`를 읽어 전체 feature 목록 파악
- 변경된 파일들이 어떤 feature에 속하는지 식별
- 관련 feature가 있다면:
  - `specs/codebase/features/[관련feature].md` 읽기
  - 기존 구조, 의존성, API 정보 파악
  - 작업 내용이 이 컨텍스트와 어떻게 연결되는지 분석

---

## 2. 현재 작업 내용 분석

다음 명령어들을 실행하여 현재 작업 내용을 파악하십시오:

```bash
# 변경된 파일 목록
git status

# staged 및 unstaged 변경사항
git diff HEAD

# 최근 커밋들 (현재 브랜치에서 작업한 내용)
git log --oneline -10
```

변경된 파일들을 읽고 분석하십시오:
- 어떤 기능이 추가/수정되었는지
- 어떤 패턴이 사용되었는지
- 테스트가 포함되어 있는지

---

## 3. 문서 생성

### 3.1 plan.md 생성

분석된 작업 내용을 기반으로 `specs/[폴더명]/plan.md`를 작성하십시오:

```markdown
# [기능명] Implementation Plan

## Problem
- (변경사항에서 추론한 해결하려던 문제)

## Selected Strategy
- (실제 구현에서 사용된 접근 방식)
- (사용된 패턴, 라이브러리 등)

## Milestones

### Phase 1: [완료된 단계]
- [x] Task 1.1: ...
- [x] Task 1.2: ...
- [x] Verification: (테스트 존재 여부)

### Phase 2: [완료된 단계]
- [x] Task 2.1: ...
- [x] Verification: (테스트 존재 여부)

(실제 작업 내용을 Phase로 분리하여 기록)
```

### 3.2 spec.md 생성

`specs/[폴더명]/spec.md`를 작성하십시오:

```markdown
---
name: [spec-name]
related_features: [1.5에서 파악한 관련 feature 목록]
status: completed
created: YYYY-MM-DD
---

# [기능명] Specification

## User Story
- (구현된 기능에서 추론한 사용자 스토리)

## Acceptance Criteria
- [x] (구현된 기능 기준)
- [x] (테스트로 검증된 기준)

## Technical Constraints
- (사용된 기술 스택, 제약사항)

## Data Structure / API
- (추가/수정된 모델, API 엔드포인트)

## Implementation Notes
- (구현 중 발견한 특이사항, 주의점)
```

### 3.3 context.md 생성

`specs/[폴더명]/context.md`를 작성하십시오:

```markdown
# [기능명] Context

## Current Status
- Phase: Completed
- Last Updated: [날짜]

## Summary
- (작업 완료 요약)

## Files Changed
- (변경된 파일 목록)

## Related Commits
- (관련 커밋 해시 및 메시지)

## Session History
- [날짜]: Implementation completed, documentation created via /recap
```

---

## 4. 코드베이스 인덱스 업데이트 확인

작업 내용이 기존 feature의 구조를 변경했다면:
- `specs/codebase/features/[관련feature].md` 업데이트 필요 여부 확인
- 새로운 컴포넌트, API, 타입이 추가되었다면 해당 정보 반영
- 사용자에게 업데이트 여부 확인 후 진행

---

## 5. 마무리

모든 문서 생성 후 다음과 같이 출력:

```
Documentation created from existing work:
  - specs/[폴더명]/plan.md (구현 계획 - 역추출)
  - specs/[폴더명]/spec.md (요구사항 - 역추출)
  - specs/[폴더명]/context.md (완료 상태)

예시: specs/20260108-workspace-member-usage/

Files analyzed:
  - [변경된 파일 목록]

Commits included:
  - [관련 커밋 목록]
```
