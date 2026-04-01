# Frontend Development Workflow

컴포넌트 중심 개발과 사용자 인터랙션 테스트 기반 프론트엔드 워크플로우.

---

## 1. 핵심 원칙

### 1.1 컴포넌트 주도 개발 (Component-Driven Development)

```
단위 컴포넌트 → 복합 컴포넌트 → 페이지 → 사용자 흐름
```

| 단계 | 설명 |
|------|------|
| **단위 컴포넌트** | 독립적으로 동작하는 최소 UI 단위 |
| **복합 컴포넌트** | 단위 컴포넌트 조합으로 구성 |
| **페이지** | 복합 컴포넌트를 배치하고 데이터 연결 |
| **사용자 흐름** | 페이지 간 이동과 전체 시나리오 검증 |

### 1.2 테스트 전략 (Testing Trophy)

| 레벨 | 도구 | 목적 | 비중 |
|------|------|------|------|
| **Static** | TypeScript, ESLint | 타입 오류, 코드 품질 | 항상 |
| **Unit** | Vitest/Jest | 유틸 함수, 훅, 순수 로직 | 낮음 |
| **Integration** | Testing Library | 컴포넌트 렌더링, 사용자 인터랙션 | **높음** |
| **E2E** | Playwright | 전체 사용자 흐름, 크로스 브라우저 | 핵심 경로 |

> **핵심**: Integration 테스트에 가장 많은 비중. 사용자가 실제로 보고 상호작용하는 방식으로 테스트.

### 1.3 사용자 중심 테스트 원칙

- **구현 세부사항이 아닌 동작을 테스트**: `getByRole`, `getByText` 사용. `getByTestId`는 최후 수단
- **사용자 이벤트 시뮬레이션**: `fireEvent`보다 `userEvent` 우선 사용
- **비동기 처리**: `waitFor`, `findBy*` 쿼리로 비동기 상태 변화 대기
- **접근성 기반 쿼리 우선순위**: role > label > text > testid

### 1.4 상태 관리 테스트

- **로컬 상태**: 컴포넌트 내부 상태 변화를 사용자 인터랙션으로 검증
- **전역 상태**: Provider로 감싸서 통합 테스트
- **서버 상태**: MSW(Mock Service Worker)로 API 모킹 후 데이터 흐름 검증
- **URL 상태**: 라우터 모킹으로 쿼리 파라미터/경로 변화 검증

---

## 2. 워크플로우 규칙

### 2.1 컴포넌트 개발 순서

1. **타입 정의**: Props, 상태 인터페이스 먼저 정의
2. **테스트 작성**: 렌더링 → 인터랙션 → 엣지케이스 순
3. **구현**: 테스트 통과를 위한 최소 구현
4. **스타일링**: 기능 완성 후 UI 다듬기
5. **접근성 검증**: 키보드 네비게이션, 스크린리더 호환

### 2.2 Integration 테스트 패턴

```typescript
// 좋은 예: 사용자 행동 기반
test('사용자가 폼을 제출하면 성공 메시지가 표시된다', async () => {
  render(<Form />);
  await userEvent.type(screen.getByLabelText('이름'), '홍길동');
  await userEvent.click(screen.getByRole('button', { name: '제출' }));
  expect(await screen.findByText('제출 완료')).toBeInTheDocument();
});

// 나쁜 예: 구현 세부사항 테스트
test('setState가 호출된다', () => {
  // 이런 테스트는 작성하지 않음
});
```

### 2.3 E2E 테스트 (Playwright)

**작성 기준**
- 핵심 사용자 흐름만 E2E로 커버 (로그인, 결제, 핵심 CRUD)
- 나머지는 Integration 테스트로 충분

**Auth 처리**
- 재사용 가능한 테스터 로그인 헬퍼 함수 생성
- 필요 시 `auth.setup.ts`에서 1회 로그인 후 상태 저장하여 재사용
- Auth 로그인 불가 시: `test/` 페이지를 별도 생성하여 검증

**패턴**
```typescript
// auth.setup.ts - 로그인 상태 저장
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('test@example.com');
  await page.getByLabel('Password').fill('password');
  await page.getByRole('button', { name: 'Login' }).click();
  await page.context().storageState({ path: '.auth/user.json' });
});
```

### 2.4 API 모킹 전략

| 도구 | 용도 |
|------|------|
| **MSW** | Integration 테스트에서 API 모킹 (브라우저/Node 모두 지원) |
| **Playwright route** | E2E 테스트에서 특정 API 응답 제어 |

```typescript
// MSW 핸들러
export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([{ id: 1, name: '홍길동' }]);
  }),
];
```

### 2.5 Context Isolation Protocol

- 한 번에 **하나의 컴포넌트/페이지**에 집중
- `specs/[feature]/` 폴더의 spec.md, plan.md, context.md를 Source of Truth로 사용
- 백엔드 워크플로우와 동일한 Phase-Based Implementation 적용

### 2.6 Phase-Based Implementation

#### Phase 1: Planning (No Code)
- `plan.md` 생성 — 컴포넌트 구조, 상태 설계, API 연동 계획
- 구현을 3~5개의 단계로 분리
- 사용자 승인 후 진행

#### Phase 2: Execution
- 컴포넌트 단위로 테스트 → 구현 → 검증
- 한 번에 하나의 컴포넌트만 수정
- 수정 후 즉시 관련 테스트 실행

#### Phase 3: Completion
1. `context.md`에 완료 내용 기록
2. `plan.md` 해당 단계 완료 처리
3. `spec.md` 요구사항 위반 여부 검토

---

## 3. 코드 품질 체크리스트

- [ ] TypeScript strict 모드 에러 없음
- [ ] ESLint 경고/에러 없음
- [ ] 접근성 (a11y) 기본 규칙 준수
- [ ] 반응형 레이아웃 확인 (필요 시)
- [ ] 로딩/에러/빈 상태 처리

---

## 4. 커밋 규율

- 타입체크 + 린트 + 테스트 모두 통과 시에만 커밋
- 컴포넌트 구조 변경과 동작 변경 별도 커밋
- 스타일 변경은 로직 변경과 분리
