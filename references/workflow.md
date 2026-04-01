# Spec Workflow: 자가 평가 및 개선 사이클

## 전체 흐름

```
spec-plan → spec-execute → spec-write → spec-code-review → spec-testing
                                ↑                                   |
                                └───── (문제 발견 시) ──────────────┘
```

---

## Phase 1: 계획 (spec-plan)

사용자 요청을 분석하고, 불명확한 부분을 질문으로 확인한 후 구현 계획을 수립한다.

- 코드 기반 검증 → 질문 → 계획 제시
- 계획 승인 후 spec-execute로 자동 이어짐

## Phase 2: 실행 (spec-execute)

스펙 문서(spec.md, plan.md, context.md)를 생성/갱신하고 개발 에이전트로 구현한다.

- 스펙 문서 생성/갱신
- 작업 영역 판별 → 개발 에이전트 spawn
- 개발 완료 후 spec-write로 자동 이어짐

## Phase 3: 문서 갱신 (spec-write)

개발 결과를 반영하여 스펙 문서를 갱신한다.

- 변경된 파일 목록, 완료된 작업 반영
- context.md 상태 업데이트
- 갱신 완료 후 spec-code-review로 자동 이어짐

## Phase 4: 코드 리뷰 (spec-code-review)

code-reviewer 에이전트를 spawn하여 변경된 코드를 리뷰한다.

- 정확성/보안/성능/품질 관점 리뷰
- 리뷰 기준: `~/.claude/skills/spec/references/code-review-checklist.md`
- 결과: blocker / issue / suggestion 분류

## Phase 5: 테스트 (spec-testing)

테스터 에이전트를 spawn하여 테스트를 실행한다.

- 작업 영역에 맞는 테스터 에이전트 선택
- spec.md의 Test Plan 기반 테스트

---

## 자가 평가 및 개선 사이클

### 진입 조건

코드 리뷰 또는 테스트에서 문제가 발견되면 자동으로 개선 사이클에 진입한다.

### 판단 기준

| 단계 | 통과 조건 | 재진입 조건 |
|------|----------|------------|
| **spec-code-review** | blocker 0개 | blocker 1개 이상 |
| **spec-testing** | 전체 테스트 통과 | 테스트 실패 1건 이상 |

### 개선 사이클 흐름

```
[문제 발견]
     ↓
[문제 분석 및 원인 파악]
     ↓
[수정 작업 수행] ← 개발 에이전트 재실행
     ↓
[spec-write] ← 수정 내용 반영
     ↓
[spec-code-review] ← 재리뷰
     ↓
[spec-testing] ← 재테스트
     ↓
[통과?] → Yes → 완료
   ↓ No
[개선 사이클 재진입] (최대 3회)
```

### 최대 반복 횟수

- 개선 사이클은 **최대 3회**까지 반복한다.
- 3회 반복 후에도 해결되지 않으면 사용자에게 보고하고 수동 판단을 요청한다.

### 개선 사이클 보고 형식

```
━━━━━━━━━━━━━━━━━━━━━━━━━
개선 사이클 [N/3]회차
━━━━━━━━━━━━━━━━━━━━━━━━━

발견된 문제:
  - [blocker/issue/test failure 목록]

수정 내용:
  - [수정된 파일 및 변경 사항]

재리뷰 결과:
  - blocker: N개 | issue: N개

재테스트 결과:
  - 통과: N개 | 실패: N개

상태: [통과 / 재시도 필요 / 사용자 판단 필요]
━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 워크플로우 완료 조건

모든 단계를 통과하면 최종 보고를 출력한다:

```
━━━━━━━━━━━━━━━━━━━━━━━━━
Spec Workflow 완료
━━━━━━━━━━━━━━━━━━━━━━━━━

  기능: [기능명]
  스펙: specs/[폴더명]/
  개선 사이클: [N]회 수행
  코드 리뷰: blocker 0 | issue N | suggestion N
  테스트: 전체 통과

  수정 파일:
    - [파일 목록]
━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 개별 스킬 단독 실행

각 스킬은 워크플로우의 일부로 자동 연결되지만, 단독으로도 실행 가능하다:

| 명령 | 단독 실행 |
|------|----------|
| `/spec-plan` | 계획만 수립 |
| `/spec-execute` | 전체 워크플로우 실행 (plan → execute → write → review → test) |
| `/spec-write` | 스펙 문서만 작성/갱신 |
| `/spec-code-review` | 현재 변경사항만 리뷰 |
| `/spec-testing` | 현재 변경사항만 테스트 |

`/spec-execute`가 전체 워크플로우의 오케스트레이터 역할을 한다.
