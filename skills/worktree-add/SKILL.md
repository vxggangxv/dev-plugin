---
name: worktree-add
description: Git worktree를 .worktree 디렉토리에 생성
argument-hint: <브랜치명>
allowed-tools: Bash(git *), Bash(mkdir *), Bash(cd *), Read, Grep, Edit
disable-model-invocation: true
---
name: worktree-add

# Git Worktree 생성

`.worktree/` 디렉토리 안에 git worktree를 생성합니다.

브랜치명: $ARGUMENTS

## 수행 절차

다음 순서대로 작업을 수행하세요:

### 1. 인자 확인
- `$ARGUMENTS`가 비어있으면 사용자에게 브랜치명을 물어보세요.

### 2. 프로젝트 루트 결정 (CRITICAL)
현재 위치가 `.worktree` 디렉토리 내부인지 확인합니다:
- `pwd` 결과에 `/.worktree/`가 포함되어 있으면 → **워크트리 내부**
  - `pwd | sed 's|/.worktree/.*||'` 로 메인 레포 루트(`<REPO_ROOT>`)를 구합니다.
  - (주의: `git worktree list`는 서브모듈 환경에서 git 디렉토리 경로를 반환하므로 사용 불가)
- `/.worktree/`가 포함되어 있지 않으면 → **일반 위치**
  - 현재 디렉토리(`pwd`)를 `<REPO_ROOT>`로 사용합니다.

이후 모든 경로는 `<REPO_ROOT>` 기준으로 작업합니다.

### 3. .gitignore 확인 및 업데이트
- `<REPO_ROOT>/.gitignore` 파일을 확인합니다.
- `.worktree` 항목이 없으면 파일 끝에 추가합니다.
- `.gitignore` 파일이 없으면 새로 생성하고 `.worktree`를 추가합니다.

### 4. .worktree 디렉토리 생성
- `<REPO_ROOT>/.worktree/` 디렉토리가 없으면 생성합니다: `mkdir -p <REPO_ROOT>/.worktree`

### 5. Git Worktree 생성
- 절대경로로 worktree를 생성합니다: `git worktree add <REPO_ROOT>/.worktree/<브랜치명> -b <브랜치명>`
- 이미 해당 브랜치가 존재하면 `-b` 플래그 없이 실행합니다: `git worktree add <REPO_ROOT>/.worktree/<브랜치명> <브랜치명>`
- 이미 해당 worktree가 존재하면 사용자에게 알립니다.

### 6. 결과 보고
완료 후 다음 정보를 출력합니다. `cd` 명령은 실행하지 말고 사용자가 직접 복사해서 실행할 수 있도록 출력만 합니다:

```
Worktree 생성 완료
  브랜치: <브랜치명>
  경로: <REPO_ROOT>/.worktree/<브랜치명>
  현재 worktree 목록: (git worktree list 결과)

이동하려면 아래 명령어를 실행하세요:
  cd <REPO_ROOT>/.worktree/<브랜치명>
```
