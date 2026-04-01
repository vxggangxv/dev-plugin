---
description: 코드베이스 feature indexing 및 spec 메타데이터 추가
disable-model-invocation: true
allowed-tools: Read, Write, Glob, Grep, Edit, Task
---

# Codebase Index Command

코드베이스 feature indexing 및 spec 메타데이터 관리를 수행합니다.
**프론트엔드와 백엔드 프로젝트 모두 지원합니다.**

---

## 0. 프로젝트 타입 감지

### 0.1 프론트엔드 프로젝트 감지
다음 중 하나가 존재하면 프론트엔드:
- `src/features/` 디렉토리
- `package.json` + `next.config.js` 또는 `vite.config.ts`

### 0.2 백엔드 프로젝트 감지
다음 중 하나가 존재하면 백엔드:
- `app/router/` 디렉토리
- `pyproject.toml` + `app/main.py`

---

## 1. 디렉토리 구조 (공통)

```
specs/codebase/
├── index.md                  # 전체 도메인/feature 목록 + 메타데이터
├── dependencies.md           # 도메인/feature 간 의존성 그래프 (Mermaid)
├── api-mapping.md            # API 엔드포인트 ↔ 도메인/feature 매핑
└── features/
    └── [domain-name].md      # 개별 도메인/feature 문서
```

---

## 2. 프론트엔드 프로젝트 (src/features/ 기반)

### 2.1 Feature 탐색
- `src/features/` 폴더의 모든 feature 디렉토리 탐색
- 각 feature의 구조 분석:
  - `_api/` - API 타입 및 쿼리
  - `_hooks/` - 커스텀 훅
  - `_stores/` - Jotai atoms
  - `_utils/` - 유틸리티
  - 컴포넌트 파일들

### 2.2 Feature 문서 템플릿 (프론트엔드)
```markdown
# [Feature Name]

## 기본 정보
- **purpose**: 기능 목적
- **status**: stable | active | deprecated
- **complexity**: low | medium | high

## 구조
- **entry_point**: 진입점 컴포넌트
- **components**: 주요 컴포넌트 목록
- **hooks**: 커스텀 훅
- **stores**: Jotai atoms

## 의존성
- **depends_on**: 의존하는 feature
- **used_by**: 이 feature를 사용하는 곳

## API 연동
- **endpoints**: 사용하는 API 엔드포인트
- **mutations**: 사용하는 mutations

## 핵심 타입
- 주요 타입 정의 및 설명

## 컨텍스트
- **related_specs**: 관련 spec 문서
- **notes**: 작업 시 참고사항
```

---

## 3. 백엔드 프로젝트 (app/ 기반)

### 3.1 도메인 탐색
- `app/router/service/v1/`, `app/router/service/v2/` - API 라우터
- `app/service/` - 서비스 레이어
- `app/repo/` - 레포지토리 레이어
- `app/model/` - SQLModel 정의
- `app/schema/` - Pydantic 스키마

### 3.2 도메인 매핑 규칙
```
*_router.py     → 도메인 라우터
*_service.py    → 도메인 서비스
*_repo.py       → 도메인 레포지토리
*.py (model/)   → 도메인 모델
```

### 3.3 Feature 문서 템플릿 (백엔드)
```markdown
# [Domain Name]

## 기본 정보
- **purpose**: 도메인 목적
- **status**: active | deprecated
- **complexity**: low | medium | high

## 구조
- **router**: `app/router/service/v1/xxx_router.py`
- **service**: `app/service/xxx_service.py`
- **repo**: `app/repo/xxx_repo.py`
- **model**: `app/model/xxx.py`
- **schema**: `app/schema/xxx.py`

## 의존성
- **depends_on**: 의존하는 도메인
- **used_by**: 이 도메인을 사용하는 곳
- **external**: 외부 서비스 연동 (Elasticsearch, Kafka, S3 등)

## API 연동
| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/v1/xxx/` | 목록 조회 |
| POST | `/v1/xxx/` | 생성 |
| ... | ... | ... |

## 핵심 타입

### Model
\`\`\`python
class Xxx(SQLModel, table=True):
    id: int
    name: str
    # ...
\`\`\`

### Enum
\`\`\`python
class XxxStatus(str, Enum):
    ACTIVE = "active"
    # ...
\`\`\`

## 주요 서비스 메서드
\`\`\`python
class XxxService:
    async def get_xxx(...) -> Xxx
    async def create_xxx(...) -> Xxx
    # ...
\`\`\`

## 컨텍스트
- **related_specs**: 관련 spec 문서
- **notes**: 작업 시 참고사항
```

---

## 4. Spec Metadata (공통)

### 4.1 기존 Spec 탐색
- `/specs/` 폴더의 모든 spec 디렉토리 탐색
- 각 `spec.md` 파일의 YAML frontmatter 확인

### 4.2 Frontmatter 없는 파일 처리
YAML frontmatter가 없는 spec.md에 다음 형식 추가:
```yaml
---
name: [spec-name]
related_features: [관련 도메인/feature 목록]
status: completed | in_progress | planned
created: YYYY-MM-DD
---
```

### 4.3 Related Features 추출 (프론트엔드)
- `hunet_mba_v2`, `session`, `cases`, `admin`, `case_builder`, `tutor`, `war-room`

### 4.4 Related Features 추출 (백엔드)
- `meeting`, `workspace`, `team`, `chat`, `chat_agent`, `notification`
- `transcript`, `summary`, `user_settings`, `audit_log`, `stt`
- `upload`, `prompt`, `comment`, `oauth`, `slack`, `notion`, `google`

---

## 5. 실행 절차

### 5.1 Phase 1: 프로젝트 타입 감지
1. `src/features/` 또는 `app/router/` 존재 여부 확인
2. 프론트엔드/백엔드 결정

### 5.2 Phase 2: 디렉토리 생성
```bash
mkdir -p specs/codebase/features
```

### 5.3 Phase 3: 도메인/Feature 문서 생성
- 각 도메인/feature에 대해 문서 생성
- 기존 문서가 있으면 업데이트

### 5.4 Phase 4: Index 문서 업데이트
- `specs/codebase/index.md` - 전체 목록
- `specs/codebase/api-mapping.md` - API 매핑
- `specs/codebase/dependencies.md` - 의존성 그래프

### 5.5 Phase 5: Spec Frontmatter 추가
- `specs/*/spec.md` 파일들 순회
- frontmatter 없는 파일에 추가

---

## 6. 결과 출력

### 6.1 생성된 파일 목록
```
✅ Created/Updated:
- specs/codebase/index.md
- specs/codebase/dependencies.md
- specs/codebase/api-mapping.md
- specs/codebase/features/[name].md (x N)

📝 Metadata added:
- specs/[spec-name]/spec.md (x M)
```

### 6.2 검증 항목
- [ ] 모든 도메인/feature가 index.md에 등록됨
- [ ] dependencies.md의 Mermaid 다이어그램 유효
- [ ] 모든 spec.md에 frontmatter 존재

---

## 사용법

```
/features-index
```

이 커맨드는 다음을 수행합니다:
1. 프로젝트 타입 감지 (프론트엔드/백엔드)
2. 도메인/feature 분석하여 문서 생성
3. 기존 spec 파일에 메타데이터 추가
4. 결과 요약 출력
