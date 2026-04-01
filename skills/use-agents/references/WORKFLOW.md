# 서브에이전트 시스템 사용 가이드

## 두 가지 사용 방식

### 방식 1: 플랜 모드 → use agents 실행

Claude Code의 플랜 모드로 직접 계획을 세운 후, 실행 시 에이전트 시스템 활용

```bash
# 1. 플랜 모드에서 계획 수립 (기존 방식)
claude --plan "결제 기능을 추가하고 싶어"

# 2. 플랜 확정 후 에이전트로 실행
use agents 실행해줘
```

**설정**: `CLAUDE_MD_ADDITION.md` 내용을 프로젝트 `CLAUDE.md`에 추가

---

### 방식 2: /use-agents 커맨드로 전체 진행

처음부터 에이전트 시스템이 플래닝부터 실행까지 담당

```bash
/use-agents 인증 시스템을 만들거야
```

**흐름**:
1. planner가 요구사항 분석 & 질문
2. 사용자 답변
3. 코드베이스/문서 탐색
4. 플랜 초안 제시 & **승인 요청**
5. 승인 후 개발 에이전트 실행

**설정**: `.claude/commands/use-agents.md` 배치

---

## 설치 방법

```bash
# 1. 압축 해제
unzip claude-agents.zip

# 2. 프로젝트에 복사
mkdir -p .claude/commands
mkdir -p .claude/agents

cp claude-agents/commands/*.md .claude/commands/
cp claude-agents/agents/*.md .claude/agents/

# 3. 방식 1 사용 시: CLAUDE.md에 내용 추가
cat claude-agents/CLAUDE_MD_ADDITION.md >> CLAUDE.md
```

---

## 에이전트 목록

| 에이전트 | 파일 | 역할 |
|---------|------|------|
| codebase-explorer | agents/codebase-explorer.md | 프로젝트 코드 탐색 |
| docs-researcher | agents/docs-researcher.md | 외부 문서 탐색 (Context7 MCP) |
| planner | agents/planner.md | 계획 수립 & 사용자 소통 |
| backend-dev | agents/backend-dev.md | 백엔드 개발 (NestJS, FastAPI) |
| frontend-web-dev | agents/frontend-web-dev.md | 웹 프론트 개발 (Next.js, React) |
| frontend-app-dev | agents/frontend-app-dev.md | 앱 프론트 개발 (React Native) |

---

## 방식별 비교

| 항목 | 방식 1 (use agents) | 방식 2 (/use-agents) |
|-----|--------------------|--------------------|
| 플래닝 | Claude Code 플랜 모드 | planner 에이전트 |
| 시작점 | 플랜 완료 후 | 처음부터 |
| 제어권 | 사용자가 플랜 직접 수립 | 에이전트가 플랜 제안 |
| 적합한 상황 | 이미 구체적인 계획이 있을 때 | 러프한 아이디어만 있을 때 |

---

## 사용 예시

### 예시 1: 방식 1 사용

```bash
# Step 1: 플랜 모드로 계획
claude --plan "사용자 프로필 수정 기능 추가"

# (플랜 모드에서 직접 계획 수립...)
# 플랜:
# 1. PATCH /api/users/:id API 만들기
# 2. EditProfileForm 웹 컴포넌트
# 3. EditProfileScreen 앱 스크린

# Step 2: 에이전트로 실행
use agents 실행해줘
```

### 예시 2: 방식 2 사용

```bash
# 한 번에 시작
/use-agents 사용자 프로필 수정 기능을 추가하고 싶어

# planner 응답:
# "프로필에서 어떤 항목을 수정 가능하게 할까요?
#  1. 이름, 이메일?
#  2. 프로필 이미지?
#  3. 비밀번호 변경도 포함?"

# 사용자: "1, 2번만. 비밀번호는 별도 화면에서"

# planner가 플랜 제시 후 승인 요청
# 승인하면 자동으로 개발 에이전트들 실행
```

---

## Context7 MCP 설정

docs-researcher 에이전트가 사용합니다.

```json
// .mcp.json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp"]
    }
  }
}
```

---

## 커스터마이징

### 에이전트 수정

각 `.claude/agents/*.md` 파일을 프로젝트에 맞게 수정:

- 기술 스택 변경
- 코딩 컨벤션 추가
- 체크리스트 수정

### 새 에이전트 추가

1. `.claude/agents/` 에 새 `.md` 파일 생성
2. 역할, 작업 방식, 출력 형식 정의
3. planner.md의 에이전트 매핑 테이블에 추가

---

## 트러블슈팅

### Q: 에이전트가 프로젝트 구조를 모를 때
→ codebase-explorer가 먼저 실행되도록 하거나, CLAUDE.md에 프로젝트 구조 설명 추가

### Q: Context7가 문서를 못 찾을 때
→ 정확한 라이브러리명 사용 (예: `@nestjs/jwt`, `react-query`)

### Q: 플랜 승인 없이 실행될 때
→ planner.md의 "승인 필수" 규칙 확인, 필요시 강조 문구 추가
