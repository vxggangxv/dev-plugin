---
name: features
description: Feature Manager Command
argument-hint: <기능명>
disable-model-invocation: true
---
name: features

# Feature Manager Command
Target Feature: "$ARGUMENTS"

## 1. 초기화 및 확인 (Initialization Check)
- `specs/$ARGUMENTS/` 폴더가 존재하는지 확인하십시오.

## 1.5 코드베이스 컨텍스트 파악 (Codebase Context)
- `specs/codebase/index.md`를 읽어 전체 feature 목록 파악
- `$ARGUMENTS`와 관련된 feature가 있다면:
  - `specs/codebase/features/[관련feature].md` 읽기
  - 의존성, API 연동, 핵심 타입 정보 파악
  - 이 정보를 바탕으로 작업 컨텍스트 구성
- 관련 feature가 없다면:
  - 신규 feature로 판단하고 CASE A 진행

## 2. 분기 처리 (Branching Logic)

### [CASE A: 폴더가 없는 경우 - 신규 생성 모드]
- 폴더가 없다면 **즉시 "Spec Interviewer" 페르소나를 채택하십시오.**
- 사용자에게 "새로운 기능 '$ARGUMENTS'에 대한 스펙 작성을 시작하겠습니다."라고 알리십시오.
- **코드베이스 컨텍스트 활용**: 관련 feature가 있다면 해당 feature의 구조, API, 타입 정보를 참조하여 질문하십시오.
- `spec.md` 작성을 위해 **한 번에 하나씩** 핵심 질문을 던지십시오 (재귀적 프롬프트):
  1. 기능의 핵심 목적은 무엇인가요? (User Story)
  2. 성공적인 결과의 기준은 무엇인가요? (Acceptance Criteria)
  3. 기술적 제약 사항이나 필수 라이브러리가 있나요?
  4. 예상되는 데이터 구조나 API 응답 형태는 무엇인가요?
- **중요:** 사용자가 충분히 답변할 때까지 `spec.md`를 생성하지 말고 대화를 이어가십시오. 정보가 충분하면 승인을 받고 파일을 생성하십시오.
- **spec.md 생성 시 YAML frontmatter 포함:**
  ```yaml
  ---
  name: [spec-name]
  related_features: [관련 feature 목록]
  status: planned
  created: YYYY-MM-DD
  ---
  ```

### [CASE B: 폴더가 있는 경우 - 로드 모드]
- `specs/$ARGUMENTS/` 내의 `spec.md`, `plan.md`, `context.md`를 순서대로 읽으십시오.
- **코드베이스 컨텍스트 연결**: spec.md의 `related_features`를 참조하여 관련 feature 문서도 함께 로드하십시오.
- 현재 진행 상황을 요약하고 다음 작업을 대기하십시오.
