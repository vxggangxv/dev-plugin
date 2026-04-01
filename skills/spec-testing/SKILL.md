---
name: spec-testing
description: 테스트 실행 스킬. "spec-testing", "스펙 테스팅", "스펙 테스트", "spec testing", "spec-testing app", "spec-testing web", "spec-testing back" 등으로 트리거. 인자로 web/app/back을 받아 해당 테스터 에이전트를 실행. 영역 뒤에 추가 텍스트가 있으면 시나리오 기반 테스트, 없으면 작업 내용 기반 자동 테스트.
---

# 테스팅 스킬

테스트를 작성하고 실행하는 스킬. 적절한 테스터 에이전트를 선택하여 실행한다.

**워크플로우 참조**: `~/.claude/skills/spec/references/workflow.md`

## 1. 인자 파싱

스킬 호출 시 인자를 확인한다:
- `/spec-testing web` → Web 테스터
- `/spec-testing app` → App 테스터 (모바일)
- `/spec-testing back` → Backend 테스터
- `/spec-testing` (인자 없음) → 자동 판별

### 시나리오 기반 vs 자동 판별

영역(web/app/back) 뒤에 추가 텍스트가 있으면 **시나리오 기반**, 없으면 **작업 내용 기반 자동**.

| 입력 | 동작 |
|------|------|
| `spec-testing app` | git diff 기반 변경 파일 분석 → 자동 테스트 작성 |
| `spec-testing app 로그인 후 프로필 수정` | "로그인 후 프로필 수정" 시나리오 기반 테스트 작성 |
| `spec-testing web 상품 검색 후 장바구니 추가` | 시나리오 기반 E2E 테스트 작성 |
| `spec-testing back 결제 API 실패 케이스` | 시나리오 기반 백엔드 테스트 작성 |

시나리오가 주어지면:
- 변경 파일 분석 대신 **시나리오 내용을 기반으로** 테스트를 작성
- 시나리오에 해당하는 소스 코드를 탐색하여 테스트 대상 파악
- 에이전트에 시나리오 텍스트를 그대로 전달

## 2. 자동 판별 (인자 없는 경우)

현재 세션의 작업 내용을 기반으로 테스트 영역을 판별한다:

### 판별 기준

| 영역 | 판별 조건 |
|------|----------|
| **back** | API, 서버, DB, 비즈니스 로직, NestJS, FastAPI, 마이그레이션 |
| **web** | React, Next.js, 컴포넌트, 페이지, UI, CSS, 웹 프론트엔드 |
| **app** | React Native, Expo, 모바일, iOS, Android, 앱 |

### 판별 방법
1. `git diff --name-only`로 변경된 파일 목록 확인
2. 변경된 파일의 경로/확장자로 영역 판별
3. 세션에서 논의된 내용 분석
4. 판별 불가 시 사용자에게 확인

### 복수 영역 해당 시
- 해당 영역별로 테스터 에이전트를 **각각** 실행한다

## 3. 테스터 에이전트 실행

### 컨텍스트 준비

테스터 에이전트에 전달할 정보를 수집한다:

1. **specs 폴더 확인**: `specs/` 아래에 현재 작업 관련 spec이 있으면 로드
2. **변경 파일 목록**: `git diff --name-only`
3. **변경 내용 요약**: 현재 세션에서 수행한 작업 요약

### 에이전트 선택 및 실행

| 인자 | 에이전트 파일 | 설명 |
|------|-------------|------|
| `web` | `~/.claude/agents/web-tester.md` | Playwright 기반 웹 E2E + Integration 테스트 |
| `app` | `~/.claude/agents/app-tester.md` | Maestro 기반 모바일 UI 테스트 |
| `back` | `~/.claude/agents/back-tester.md` | 백엔드 단위/통합 테스트 |

Agent 도구를 사용하여 에이전트를 spawn한다:

```
Agent 도구 호출:
- subagent_type: general-purpose
- prompt: 에이전트 파일 내용 + 테스트 컨텍스트(변경 파일, spec 내용, 작업 요약)
```

## 4. 결과 판정

테스트 결과를 판정한다:

- **전체 통과** → 워크플로우 완료
- **실패 존재** → 개선 사이클 진입 필요 (workflow.md 참조)

## 5. 결과 출력

```
━━━━━━━━━━━━━━━━━━━━━━━━━
테스트 실행 결과
━━━━━━━━━━━━━━━━━━━━━━━━━

  영역: [web/app/back]
  에이전트: [사용된 에이전트]
  결과: 통과 N개 | 실패 N개

  실패 항목:
    - [실패 테스트 목록]

  판정: [통과 / 개선 필요]
━━━━━━━━━━━━━━━━━━━━━━━━━
```
