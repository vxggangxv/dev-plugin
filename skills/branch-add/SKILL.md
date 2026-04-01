---
name: branch-add
description: 기본 브랜치 최신화 후 새 브랜치 생성
argument-hint: <브랜치명>
allowed-tools: Bash(git *)
disable-model-invocation: true
---
name: branch-add

# 새 브랜치 생성

기본 브랜치(main/master)를 최신화한 후 새 브랜치를 생성합니다.

브랜치명: $ARGUMENTS

## 수행 절차

다음 순서대로 작업을 수행하세요:

### 1. 인자 확인
- `$ARGUMENTS`가 비어있으면 사용자에게 브랜치명을 물어보세요.

### 2. 기본 브랜치 감지
- `git remote show origin` 또는 `git symbolic-ref refs/remotes/origin/HEAD`로 기본 브랜치를 자동 감지합니다.
- 실패하면 `main` → `master` 순서로 존재 여부를 확인합니다.

### 3. 작업 상태 확인
- `git status --porcelain`으로 커밋되지 않은 변경사항이 있는지 확인합니다.
- 변경사항이 있으면 사용자에게 경고하고, 진행 여부를 확인합니다.
  - stash할지, 그대로 진행할지 선택하게 합니다.

### 4. 기본 브랜치 최신화
- `git fetch origin <기본브랜치>`로 원격 기본 브랜치를 최신화합니다.

### 5. 새 브랜치 생성
- `git checkout -b <브랜치명> origin/<기본브랜치>`로 원격 기본 브랜치 기준 새 브랜치를 생성합니다.
- 이미 해당 브랜치가 존재하면 사용자에게 알리고 중단합니다.

### 6. [NEW] sibling 디렉토리 감지 및 브랜치 생성
- 현재 디렉토리 이름이 `frontend` 또는 `backend`인지 확인합니다. (`basename $(pwd)`)
- 맞다면 sibling 디렉토리를 결정합니다:
  - 현재가 `frontend`이면 sibling은 `../backend`
  - 현재가 `backend`이면 sibling은 `../frontend`
- sibling 디렉토리가 존재하고 git repo인지 확인합니다. (`git -C <sibling> rev-parse --git-dir 2>/dev/null`)
- git repo라면 해당 디렉토리에서도 동일하게 수행합니다:
  1. `git fetch origin <기본브랜치>`
  2. `git checkout -b <브랜치명> origin/<기본브랜치>`
  - 이미 브랜치가 존재하면 경고만 출력하고 계속 진행합니다.
- 완료 후 원래 디렉토리로 복귀합니다.

### 7. 결과 보고
완료 후 다음 정보를 출력합니다:

```
Branch 생성 완료
  기본 브랜치: <기본브랜치> (최신화됨)
  새 브랜치: <브랜치명>
  현재 디렉토리: <현재 디렉토리> ✓
  sibling 디렉토리: <sibling 디렉토리> ✓  (sibling에도 생성된 경우)
```
