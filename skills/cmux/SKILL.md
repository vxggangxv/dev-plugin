---
name: cmux
description: cmux 터미널 환경에서 워크스페이스/서피스 관리, 멀티에이전트 오케스트레이션, 브라우저 자동화, 프로그레스 리포팅을 수행하는 스킬
trigger: cmux
user_invocable: true
---

# cmux 터미널 스킬

cmux 환경을 감지하고, cmux CLI를 활용하여 터미널 세션 관리, 병렬 에이전트 실행, 브라우저 자동화, 상태 리포팅을 수행한다.

## 사전 조건

1. cmux 환경 감지 확인:
```bash
[ -n "${CMUX_WORKSPACE_ID:-}" ] && echo "cmux 환경" || echo "cmux 아님"
```

2. cmux 환경이 아니면 일반 터미널 명령으로 폴백

## API 레퍼런스

상세 API 문서는 `~/.claude/skills/cmux/references/API.md`를 참조할 것. 사용 전 반드시 해당 파일을 읽어 최신 명령어와 파라미터를 확인한다.

## 핵심 워크플로우

### 1. 현재 상태 파악

```bash
cmux identify --json                    # 현재 workspace, pane, surface 확인
cmux list-workspaces --json             # 전체 워크스페이스 목록
cmux tree                               # 워크스페이스 전체 트리 구조 (서피스 포함)
cmux list-pane-surfaces --pane <ref>    # 특정 페인 내 서피스 목록
```

### 2. 병렬 작업 실행

서브에이전트를 cmux 스플릿에 배치하여 병렬 실행:

```bash
# 스플릿 생성
cmux new-split right
cmux new-split down

# 각 서피스에 명령 전송
cmux send --surface surface:<id> "claude\n"

# 결과 수집
cmux read-screen --surface surface:<id> --lines 50

# 정리
cmux close-surface --surface surface:<id>
```

### 3. 프로그레스 리포팅

작업 진행 상황을 사이드바에 표시:

```bash
cmux set-status task "작업 중" --icon hammer
cmux set-progress 0.3 --label "분석 중..."
cmux log --level info --source "agent" -- "Step 1 완료"
cmux notify --title "완료" --body "모든 작업 완료"
```

### 4. 브라우저 자동화

```bash
cmux new-pane --type browser --url <url>
cmux browser <surface> wait --load-state complete
cmux browser <surface> snapshot --interactive
cmux browser <surface> click '<element_ref>'
cmux browser <surface> fill '<element_ref>' 'text'
```

### 5. 화면 읽기 (터미널 출력 확인)

```bash
cmux read-screen --surface <ref> --lines <n>
```

## 주의사항

- cmux 환경 외부(`CMUX_WORKSPACE_ID` 미설정)에서는 cmux 명령을 실행하지 않는다
- 서피스/워크스페이스 ID를 먼저 확인한 후 명령을 보낸다
- `send` 명령으로 텍스트 전송 시 `\n`을 포함해야 Enter가 입력된다
- 브라우저 패널 생성 후 `wait --load-state complete`로 로드 완료를 기다린다
