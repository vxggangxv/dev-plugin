# 서브에이전트 시스템 가이드

## use [agentname] 키워드 트리거 (개별 에이전트 실행)

**"use [에이전트명]"** 또는 **"use @[파일경로]"** 키워드가 포함되면 해당 에이전트를 즉시 실행합니다.

### 에이전트 파일 경로
- **기본 경로**: `~/.claude/agents/[agentname].md`

### 호출 방식
| 방식 | 예시 |
|-----|------|
| 단축형 | `use codebase-explorer` → `~/.claude/agents/codebase-explorer.md` |
| 명시적 | `use @~/.claude/agents/codebase-explorer.md` |

### 트리거 예시
```
use codebase-explorer
loginType 정의된 위치 찾아줘

use @~/.claude/agents/docs-researcher.md
Next.js 14 App Router 문서 확인해줘
```

### 실행 동작
1. `use [agentname]` 또는 `use @[경로]` 감지
2. 에이전트 프롬프트 파일 읽기 (단축형은 `~/.claude/agents/`에서 탐색)
3. 해당 에이전트 역할에 맞게 작업 수행
4. 에이전트 출력 형식에 따라 결과 보고

---

## 에이전트 실행 방법 (CRITICAL)

에이전트 실행 시 반드시 **Agent 도구**를 사용하여 서브에이전트를 spawn합니다.

### 왜 Agent 도구를 사용해야 하는가?

- **메인 컨텍스트 절약**: 서브에이전트가 별도 컨텍스트에서 작업 수행
- **병렬 처리 가능**: 독립적인 작업은 여러 에이전트 동시 실행
- **역할 분리**: 각 에이전트가 전문 영역에 집중

### 실행 원칙: 병렬 우선, 의존성 있을 때만 순차

**기본 규칙**:
- 독립적인 작업 → 병렬 실행 (속도 최적화)
- 의존성 있는 작업 → 순차 실행 (정확성 보장)

### 의존성 판단 체크리스트

다음 중 **하나라도 해당**하면 순차 실행 필요:
- [ ] 작업 A의 출력이 작업 B의 입력으로 필요
- [ ] 실행 순서가 최종 결과에 영향
- [ ] 같은 파일/코드를 동시 수정하여 충돌 가능성
- [ ] 데이터베이스 스키마 변경 → 코드 업데이트 관계
- [ ] 백엔드 API 타입 정의 → 프론트엔드 타입 업데이트

**위 조건에 해당하지 않으면 → 병렬 실행**

### 실행 절차

1. **에이전트 프롬프트 파일 읽기**: `~/.claude/agents/[에이전트명].md`
2. **Agent 도구 호출**:
   ```
   Agent 도구 파라미터:
   - subagent_type: "general-purpose"
   - description: "에이전트명 - 작업 요약"
   - prompt: |
       [에이전트 프롬프트 파일 내용]

       ---
       ## 현재 작업
       [구체적인 작업 내용 및 컨텍스트]

       ## 작업 대상 파일
       [수정할 파일 경로들]

       ## 참조 정보
       [spec.md, plan.md 등 관련 정보]
   ```

### 예시: frontend-web-dev 에이전트 실행

```
1. Read: ~/.claude/agents/frontend-web-dev.md
2. Agent 도구 호출:
   - subagent_type: "general-purpose"
   - description: "frontend-web-dev - Microsoft 로그인 버튼 추가"
   - prompt: |
       [frontend-web-dev.md 내용]

       ---
       ## 현재 작업
       LoginOrg.tsx에 Microsoft 로그인 버튼 추가

       ## 작업 대상 파일
       - src/components/organisms/auth/LoginOrg.tsx

       ## 요구사항
       - 버튼 위치: Google, Kakao 아래
       - 버튼 텍스트: "Microsoft 계정으로 계속하기"
       - provider ID: 'microsoft-entra-id'
```

### 병렬 실행 (기본 권장)

**독립적인 작업은 단일 메시지에 여러 Agent 도구 호출**로 병렬 실행:

```
[use dev 감지] → [독립적 작업 2개 판별]
     ↓
[동시에 Task 호출: backend-dev, frontend-web-dev] → 병렬 실행
     ↓
[두 결과 수집 후 통합 보고]
```

**병렬 실행 예시**:
- 백엔드 API 엔드포인트 + 프론트 UI 컴포넌트
- 문서 리서치 + 코드베이스 탐색
- 여러 독립적인 기능 개발
- 서로 다른 파일 동시 수정

### 순차 실행 (의존성 있을 때만)

**의존성이 있는 경우에만 순차 실행**:

```
[use dev 감지] → [영역 판별: 의존성 있는 작업]
     ↓
[Task: backend-dev 에이전트 spawn] → 결과 대기
     ↓
[결과 확인 후 Task: frontend-web-dev 에이전트 spawn] → 결과 대기
     ↓
[메인에서 통합 결과 보고]
```

**순차 실행 예시**:
- DB 스키마 변경 → TypeScript 타입 업데이트
- 백엔드 API 인터페이스 변경 → 프론트 API 클라이언트 수정
- 같은 파일의 다른 부분 수정
- 설정 파일 변경 → 해당 설정 사용하는 코드 수정

---

## 서브에이전트 구조

```
┌─────────────────────────────────────┐
│           작업 실행 시              │
│      "use agents" 키워드 감지       │
└──────────────┬──────────────────────┘
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│ 리서치 │ │ 플래닝 │ │  개발  │
│ 그룹   │ │        │ │ 그룹   │
└────────┘ └────────┘ └────────┘
```

## 사용 가능한 에이전트

| 에이전트 | 위치 | 역할 |
|---------|------|------|
| codebase-explorer | .claude/agents/codebase-explorer.md | 프로젝트 코드 탐색 |
| docs-researcher | .claude/agents/docs-researcher.md | 외부 문서 탐색 (Context7 MCP) |
| planner | .claude/agents/planner.md | 작업 계획 수립 |
| backend-dev | .claude/agents/backend-dev.md | 백엔드 개발 |
| frontend-web-dev | .claude/agents/frontend-web-dev.md | 웹 프론트 개발 |
| frontend-app-dev | .claude/agents/frontend-app-dev.md | 앱 프론트 개발 |
| debugger | .claude/agents/debugger.md | 에러 분석 및 버그 추적 |

## 주의사항

- 에이전트 프롬프트 파일(`.claude/agents/*.md`)을 참조하여 해당 역할에 맞게 동작
- 각 에이전트의 출력 형식 준수
- 개발 에이전트는 기존 프로젝트 패턴/컨벤션 따르기
- **속도 최적화**: 가능한 한 병렬 실행으로 작업 시간 단축
- **의존성 관리**: 순차 실행 필요 시 명확한 이유 명시
- 이슈 발생 시 사용자에게 알림
