# cmux API 레퍼런스

## 개요

cmux는 AI 코딩 에이전트를 위한 macOS 네이티브 터미널. Ghostty(libghostty) 기반 렌더링, 수직 탭, 스플릿 페인, 내장 브라우저, Unix 소켓 API를 제공한다.

- 공식 사이트: https://cmux.com
- GitHub: https://github.com/manaflow-ai/cmux
- API 문서: https://cmux.com/docs/api

---

## 계층 구조

```
Window
  └── Workspace (사이드바 탭)
        └── Pane (스플릿 영역)
              └── Surface (페인 내 탭)
                    └── Panel (터미널 또는 브라우저)
```

| 컴포넌트 | 식별자 | 설명 |
|----------|--------|------|
| Window | - | 네이티브 macOS 윈도우 (Cmd+Shift+N) |
| Workspace | `CMUX_WORKSPACE_ID` | 사이드바 항목, 1개 이상의 Pane 포함 |
| Pane | Pane ID | 수평/수직 분할 영역 |
| Surface | `CMUX_SURFACE_ID` | Pane 내 개별 탭 (터미널/브라우저 세션) |
| Panel | Panel ID (내부) | Surface 내 실제 콘텐츠 |

참조 포맷: `workspace:1`, `pane:1`, `surface:2`

---

## 환경변수

| 변수 | 용도 |
|------|------|
| `CMUX_WORKSPACE_ID` | 현재 워크스페이스 ID (자동 설정) |
| `CMUX_SURFACE_ID` | 현재 서피스 ID (자동 설정) |
| `CMUX_SOCKET_PATH` | 소켓 경로 오버라이드 |
| `CMUX_SOCKET_ENABLE` | 소켓 활성화/비활성화 (1/0, true/false, on/off) |
| `CMUX_SOCKET_MODE` | 접근 모드 (cmuxOnly, allowAll, off) |
| `TERM_PROGRAM` | `ghostty`로 설정됨 |
| `TERM` | `xterm-ghostty`로 설정됨 |

---

## cmux 환경 감지

```bash
# cmux 내부에서 실행 중인지 확인
[ -n "${CMUX_WORKSPACE_ID:-}" ] && [ -n "${CMUX_SURFACE_ID:-}" ] && echo "Inside cmux"

# 소켓 사용 가능 여부
SOCK="${CMUX_SOCKET_PATH:-/tmp/cmux.sock}"
[ -S "$SOCK" ] && echo "Socket available"

# CLI 사용 가능 여부
command -v cmux &>/dev/null && echo "cmux CLI available"
```

---

## CLI 공통 옵션

| 옵션 | 설명 |
|------|------|
| `--socket PATH` | 커스텀 소켓 경로 |
| `--json` | JSON 출력 |
| `--window ID` | 특정 윈도우 지정 |
| `--workspace ID` | 특정 워크스페이스 지정 |
| `--surface ID` | 특정 서피스 지정 |
| `--id-format refs\|uuids\|both` | 식별자 포맷 |

---

## Workspace 관리

```bash
cmux list-workspaces [--json]              # 모든 워크스페이스 목록
cmux new-workspace [--command "cmd"]       # 새 워크스페이스 생성 (초기 명령 지정 가능)
cmux current-workspace [--json]            # 현재 활성 워크스페이스
cmux select-workspace --workspace <id>     # 워크스페이스 전환
cmux rename-workspace "name"               # 워크스페이스 이름 변경
cmux close-workspace --workspace <id>      # 워크스페이스 닫기
```

### Socket API

```json
{"id":"req","method":"workspace.list","params":{}}
{"id":"req","method":"workspace.create","params":{}}
{"id":"req","method":"workspace.current","params":{}}
{"id":"req","method":"workspace.select","params":{"workspace_id":"<id>"}}
{"id":"req","method":"workspace.close","params":{"workspace_id":"<id>"}}
```

---

## Split / Surface / Pane 관리

```bash
cmux new-split <left|right|up|down>                    # 현재 페인 분할
cmux new-surface [--type terminal|browser] [--pane <ref>] [--url URL]  # 페인 내 새 탭
cmux new-pane [--direction <dir>] [--type terminal|browser] [--url URL]

cmux tree [--all] [--workspace <ref>]                   # 워크스페이스 전체 트리 구조 (서피스 포함)
cmux list-panes [--workspace <ref>]                     # 워크스페이스 내 페인 목록
cmux list-pane-surfaces [--pane <ref>]                  # 페인 내 서피스 목록

cmux focus-pane --pane <id>                             # 페인 포커스
cmux close-surface [--surface <id>]                     # 서피스 닫기
cmux move-surface --surface <ref> [--pane <ref>] [--index <n>]  # 서피스 이동
cmux rename-tab [--surface <ref>] "name"                # 서피스 탭 이름 변경
```

### Socket API

```json
{"id":"req","method":"surface.split","params":{"direction":"right"}}
{"id":"req","method":"surface.list","params":{}}
{"id":"req","method":"surface.focus","params":{"surface_id":"<id>"}}
```

---

## 입력 (Text / Key)

```bash
cmux send "echo hello"                                  # 포커스된 터미널에 텍스트 전송
cmux send --surface <id> "text to send"                 # 특정 서피스에 텍스트 전송
cmux send-key enter                                     # 포커스된 터미널에 키 전송
cmux send-key --surface <id> <key>                      # 특정 서피스에 키 전송
```

**지원 키**: enter, tab, escape, backspace, delete, up, down, left, right, ctrl-c

### Socket API

```json
{"id":"req","method":"surface.send_text","params":{"text":"echo hello\n"}}
{"id":"req","method":"surface.send_text","params":{"surface_id":"<id>","text":"command"}}
{"id":"req","method":"surface.send_key","params":{"key":"enter"}}
{"id":"req","method":"surface.send_key","params":{"surface_id":"<id>","key":"enter"}}
```

---

## 화면 읽기

```bash
cmux read-screen --surface <ref> [--lines <n>]          # 서피스의 마지막 n줄 읽기
cmux identify [--json]                                   # 현재 포커스 컨텍스트 (workspace, pane, surface)
```

---

## 알림 / 상태 / 프로그레스

### 데스크탑 알림

```bash
cmux notify --title "Title" --body "Body" [--subtitle "S"]
cmux list-notifications [--json]
cmux clear-notifications
```

### 사이드바 상태

```bash
cmux set-status <key> <value> [--icon <sf_symbol_name>] [--color <#hex>]
cmux clear-status <key>
cmux list-status
```

### 프로그레스 바

```bash
cmux set-progress <0.0-1.0> [--label "text"]
cmux clear-progress
```

### 로그

```bash
cmux log [--level <info|progress|success|warning|error>] [--source "name"] [--] "message"
cmux list-log [--limit N]
cmux clear-log
```

### 사이드바 전체 상태

```bash
cmux sidebar-state [--workspace ID]
```

### 비주얼 알림

```bash
cmux trigger-flash --surface <id>                       # 서피스에 파란색 플래시 보더
```

---

## 유틸리티

```bash
cmux ping                                               # 응답성 확인
cmux capabilities [--json]                              # 지원 메서드 및 접근 모드 목록
cmux identify [--json]                                  # 포커스 컨텍스트
```

---

## 브라우저 자동화

모든 브라우저 명령: `cmux browser <surface> <subcommand> [args...]`

### 열기 / 네비게이션

```bash
cmux new-pane --type browser --url <url>                # 브라우저 패널 생성
cmux browser <surface> open-split --direction <dir>     # 브라우저 스플릿 생성
cmux browser <surface> navigate <url>                   # URL 이동
cmux browser <surface> open <url>                       # URL 열기
cmux browser <surface> back|forward|reload              # 네비게이션
cmux browser <surface> url                              # 현재 URL
cmux browser <surface> get title                        # 페이지 타이틀
```

### DOM 검사

```bash
cmux browser <surface> snapshot [--selector <sel>] [--max-depth <n>] [--interactive]
cmux browser <surface> get text <selector>
cmux browser <surface> get html <selector>
cmux browser <surface> get value <selector>
cmux browser <surface> get attr <selector> --attr <name>
cmux browser <surface> get count <selector>
cmux browser <surface> get box <selector>
cmux browser <surface> get styles <selector>
cmux browser <surface> is visible|enabled|checked <selector>
```

### 요소 상호작용

```bash
cmux browser <surface> click <selector>
cmux browser <surface> dblclick <selector>
cmux browser <surface> hover <selector>
cmux browser <surface> focus <selector>
cmux browser <surface> scroll-into-view <selector>
cmux browser <surface> scroll [--dy <pixels>]
```

### 폼 입력

```bash
cmux browser <surface> type <selector> "text"           # 텍스트 추가
cmux browser <surface> fill <selector> "text"           # 클리어 후 입력
cmux browser <surface> check|uncheck <selector>         # 체크박스
cmux browser <surface> select <selector> "value"        # 드롭다운
cmux browser <surface> press <key>                      # 키 입력
```

### 요소 찾기 (Locators)

```bash
cmux browser <surface> find role <role> [--name <name>]
cmux browser <surface> find text "text" [--exact]
cmux browser <surface> find label|placeholder "text" [--exact]
cmux browser <surface> find testid "id"
cmux browser <surface> find first|nth <index> <selector>
```

### JavaScript / 스크린샷

```bash
cmux browser <surface> eval "document.title"
cmux browser <surface> screenshot [--out <path>]
```

### 대기 (Wait)

```bash
cmux browser <surface> wait <selector>                  # 요소 대기
cmux browser <surface> wait --text "text"               # 텍스트 대기
cmux browser <surface> wait --url "url"                 # URL 대기
cmux browser <surface> wait --load-state <state>        # 페이지 로드 대기
cmux browser <surface> wait --function "js expr"        # JS 조건 대기
```

### 콘솔 / 에러

```bash
cmux browser <surface> console list
cmux browser <surface> errors list
```

### 탭 / 쿠키 / 스토리지

```bash
cmux browser <surface> tab list
cmux browser <surface> tab new [<url>]
cmux browser <surface> cookies get [--domain <d>]
cmux browser <surface> cookies set <name> <value>
cmux browser <surface> storage local get [<key>]
cmux browser <surface> storage local set <key> <val>
```

### 네트워크 / 에뮬레이션

```bash
cmux browser <surface> viewport <width> <height>
cmux browser <surface> offline true|false
cmux browser <surface> geolocation <lat> <lng>
cmux browser <surface> network route <pattern> [--abort] [--body <resp>]
cmux browser <surface> network requests
```

---

## Socket API 사용법

### 소켓 경로

| 빌드 | 경로 |
|------|------|
| Release | `/tmp/cmux.sock` |
| Debug | `/tmp/cmux-debug.sock` |
| Tagged debug | `/tmp/cmux-debug-<tag>.sock` |

`CMUX_SOCKET_PATH` 환경변수로 오버라이드 가능.

### 요청 포맷

뉴라인 종료 JSON, 필수 필드: `id`, `method`, `params`

```json
{"id":"req-1","method":"workspace.list","params":{}}
```

### 접근 모드

| 모드 | 동작 |
|------|------|
| cmuxOnly | cmux가 spawn한 프로세스만 접속 (기본값) |
| allowAll | 모든 로컬 프로세스 접속 가능 |
| off | 소켓 비활성화 |

### Python 예제

```python
import json, socket, os

SOCKET_PATH = os.environ.get("CMUX_SOCKET_PATH", "/tmp/cmux.sock")

def cmux_request(method, params=None):
    payload = {"id": "req", "method": method, "params": params or {}}
    sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    sock.connect(SOCKET_PATH)
    sock.sendall(json.dumps(payload).encode() + b"\n")
    response = json.loads(sock.recv(65536).decode())
    sock.close()
    return response

# 예시
workspaces = cmux_request("workspace.list")
cmux_request("surface.send_text", {"text": "echo hello\n"})
```

### Shell 예제

```bash
SOCK="${CMUX_SOCKET_PATH:-/tmp/cmux.sock}"
printf '{"id":"req","method":"workspace.list","params":{}}\n' | nc -U "$SOCK"
```

---

## 워크플로우 패턴

### 1. 병렬 서브에이전트 (Fan-out into splits)

```bash
# 스플릿 레이아웃 생성
cmux new-split right --workspace workspace:1
cmux new-split down --workspace workspace:1 --surface surface:7

# 에이전트 실행
cmux send --surface surface:7 "claude\n"
cmux send --surface surface:8 "claude\n"

# 작업 할당
cmux send --surface surface:7 "read and understand the project\n"
cmux send --surface surface:8 "analyse the code\n"

# 결과 수집
cmux read-screen --surface surface:7
cmux read-screen --surface surface:8

# 정리
cmux close-surface --surface surface:7
cmux close-surface --surface surface:8
```

### 2. 워크스페이스 탭 기반 독립 작업

```bash
cmux new-workspace --command "cd /project && npm test"
cmux new-workspace --command "cd /project && npm run build"
```

### 3. 테스트 & 수정 사이클

```bash
cmux send --surface <id> "npm test\n"
# ... 대기 ...
cmux read-screen --surface <id> --lines 50
# 결과 분석 후 코드 수정, 재실행
```

### 4. 프로그레스 리포팅

```bash
cmux set-status task "Building" --icon hammer
cmux set-progress 0.5 --label "Compiling..."
cmux log --level info --source "build" -- "Step 3/6 complete"
cmux notify --title "Build Done" --body "All tests passed"
```

### 5. 브라우저 자동화

```bash
cmux new-pane --type browser --url https://example.com
sleep 1
cmux browser surface:2 wait --load-state complete
cmux browser surface:2 snapshot --interactive
cmux browser surface:2 click 'e14'
cmux browser surface:2 fill 'e10' 'search text'
cmux browser surface:2 screenshot --out /tmp/result.png
```

---

## 안전 규칙

- `CMUX_WORKSPACE_ID`가 설정되어 있을 때만 cmux 명령 사용
- 서피스/워크스페이스 참조가 존재하는지 먼저 확인
- 비동기 작업에 적절한 에러 핸들링 적용
- cmux 환경 외부에서는 cmux 명령 시도하지 않음
