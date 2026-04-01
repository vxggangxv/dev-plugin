---
name: pr
description: 브랜치 간 PR 생성 및 머지. /pr [source] to [target] 또는 /pr to [target] (현재 브랜치 자동 사용). [merge] [squash] [delete] 옵션. 쉼표로 다중 PR 지원.
argument-hint: [source] to <target> [merge] [squash] [delete], [source] to <target> [merge] ...
disable-model-invocation: true
---

# PR 생성 및 머지

`$ARGUMENTS`를 파싱하여 GitHub PR을 생성하고, 옵션에 따라 머지/삭제까지 진행한다.
**질문 없이 즉시 실행한다.**

---

## 인자 파싱

### 1단계: 다중 PR 분리

`$ARGUMENTS`를 `,`(쉼표)로 분리한다. 각 항목이 독립적인 PR 명령이 된다.

- 쉼표가 없으면 → 단일 PR 명령
- 쉼표가 있으면 → 각 항목을 trim 후 개별 PR 명령으로 처리

**예시:**
- `main to develop merge, main to product merge` → 2개 PR 명령
- `to develop merge, to product merge` → 2개 PR 명령 (둘 다 현재 브랜치가 source)
- `main to develop` → 1개 PR 명령
- `to develop` → 1개 PR 명령 (현재 브랜치가 source)

### 2단계: 개별 PR 명령 파싱

각 PR 명령의 형식: `[source-branch] to [target-branch] [옵션들]`

1. `to` 키워드를 기준으로 분리
   - `to` 앞: source branch (PR의 head)
     - **비어있으면**: `git branch --show-current`로 현재 브랜치를 source로 사용
   - `to` 뒤: target branch (PR의 base) + 옵션 플래그들
2. target branch 뒤의 키워드에서 옵션 추출 (순서 무관):
   - `merge` → 머지 실행 (기본 merge 방식)
   - `squash` → 스쿼시 머지 실행 (`merge` 대신 squash 방식 사용)
   - `delete` → 머지 후 source 브랜치 삭제
3. 브랜치명 앞뒤 공백 제거

**옵션 조합 규칙:**
- `squash`는 `merge`를 포함한다 (squash만 써도 머지 진행)
- `delete`는 `merge` 또는 `squash`가 있을 때만 의미가 있다 (머지 없이 delete만 있으면 무시)
- `merge`와 `squash`가 동시에 있으면 `squash` 우선

**예시:**
- `to develop` → 현재 브랜치 → develop PR 생성만
- `to main merge` → 현재 브랜치 → main PR 생성 + 머지
- `to main squash delete` → 현재 브랜치 → main PR 생성 + 스쿼시 머지 + 현재 브랜치 삭제
- `main to develop` → PR 생성만
- `main to develop merge` → PR 생성 + 기본 머지
- `main to develop squash` → PR 생성 + 스쿼시 머지
- `main to develop merge delete` → PR 생성 + 머지 + source 브랜치 삭제
- `main to develop squash delete` → PR 생성 + 스쿼시 머지 + source 브랜치 삭제
- `feature/login to main squash delete` → PR 생성 + 스쿼시 머지 + feature/login 삭제

---

## 실행 단계

**다중 PR인 경우 각 PR 명령에 대해 아래 Step 0~3을 순차 반복한다.**
하나의 PR이 실패해도 나머지는 계속 진행한다.

### Step 0-1: 민감 파일 검사

`git log [target-branch]..[source-branch] --name-only`로 변경 파일을 확인하고, 다음 패턴이 있으면 **경고 후 사용자에게 보고**:
- `.env`, `.env.*`
- `credentials.*`, `secrets.*`
- `*.pem`, `*.key`

### Step 0-2: 작업 무관 변경사항 점검

`git diff [target-branch]...[source-branch]`를 분석하여 다음 항목을 점검합니다.
문제가 발견되면 **해당 내용을 사용자에게 보고하고 진행 여부를 확인**합니다.

#### 점검 항목

**로컬 환경 값 유출**
- `0.0.0.0`, `127.0.0.1`, `localhost` 등 로컬 도메인/IP가 코드에 하드코딩된 경우
- 예: `http://0.0.0.0:8000`, `http://localhost:3000`
- 설정 파일(`config`, `settings`, `.env.example`)이나 테스트 코드에서의 사용은 허용

**공통 코드 의도치 않은 변경**
- `APIRouter()` 파라미터 변경 (prefix, tags 등)
- 공통 미들웨어, 의존성 주입 함수 변경
- `__init__.py`의 export 변경
- 예: `APIRouter(prefix="/v1/subscriptions")` → `APIRouter(prefix="/v1/subs")` 같은 변경

**설정/인프라 파일 의도치 않은 변경**
- `alembic.ini`, `pyproject.toml`, `requirements.txt`, `Dockerfile` 등 설정 파일이 변경된 경우
- 현재 작업과 관련 없는 설정 변경이면 경고

**디버깅 코드 잔존**
- `print()`, `console.log()`, `breakpoint()`, `pdb.set_trace()` 등 디버깅 코드
- 주석 처리된 코드 블록 추가 (`# TODO`, `# FIXME`는 허용)

#### 점검 결과 보고 형식

문제 발견 시:
```
⚠️ 작업 무관 변경사항 감지

1. [파일명:라인] 로컬 도메인 사용 → 0.0.0.0:8000
2. [파일명:라인] APIRouter prefix 변경 → 다른 API에 영향 가능
3. [파일명:라인] print() 디버깅 코드 잔존

계속 진행하시겠습니까? (이 변경사항들이 의도된 것이라면 진행)
```

문제 없으면 다음 단계로 진행합니다.

### Step 1: PR 생성

```bash
# 변경 내역 확인
git log [target-branch]..[source-branch] --oneline

# PR 생성
gh pr create --base [target-branch] --head [source-branch] --title "[타이틀]" --body "[본문]"
```

- **타이틀**: `[source-branch] → [target-branch]` 형식, 또는 커밋이 1개면 해당 커밋 메시지 사용
- **본문**: `git log [target-branch]..[source-branch] --oneline` 결과를 포함한 변경 요약

### Step 2: 머지 (merge 또는 squash 플래그가 있을 때만)

PR 생성이 성공한 후 실행:

```bash
# squash 옵션인 경우
gh pr merge [PR번호 또는 URL] --squash

# merge 옵션인 경우 (기본)
gh pr merge [PR번호 또는 URL] --merge
```

- PR 생성 결과에서 PR 번호/URL을 추출하여 사용

### Step 3: 브랜치 삭제 (delete 플래그가 있고 머지가 성공했을 때만)

머지 완료 후 실행:

```bash
# 리모트 브랜치 삭제
git push origin --delete [source-branch]

# 로컬 브랜치 삭제 (존재하는 경우)
git branch -d [source-branch] 2>/dev/null
```

---

## 완료 보고

각 PR 결과를 개별 블록으로 출력한다. 다중 PR이면 모두 끝난 후 한번에 보고한다.

```
━━━━━━━━━━━━━━━━━━━━━━━━━
PR 완료 (1/N)

  방향: [source] → [target]
  PR: [PR URL]
  머지: [merge | squash | 해당 없음]
  브랜치 삭제: [완료 | 해당 없음]
━━━━━━━━━━━━━━━━━━━━━━━━━
PR 완료 (2/N)

  방향: [source] → [target]
  PR: [PR URL]
  머지: [merge | squash | 해당 없음]
  브랜치 삭제: [완료 | 해당 없음]
━━━━━━━━━━━━━━━━━━━━━━━━━
```

단일 PR이면 기존처럼 `(1/N)` 없이 출력해도 된다.

---

## 에러 처리

- 브랜치가 존재하지 않으면: 에러 메시지 출력 후 해당 PR 건너뛰기 (다중 PR 시 나머지 계속 진행)
- 이미 PR이 존재하면: 기존 PR URL을 안내하고, merge/squash 플래그가 있으면 기존 PR을 머지
- 머지 충돌이 있으면: 충돌 사실을 알리고 해당 PR 중단 (강제 머지하지 않음)
- 브랜치 삭제 실패: 경고 출력하되 전체 작업은 성공으로 처리
