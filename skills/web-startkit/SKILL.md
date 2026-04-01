---
name: web-startkit
description: Web Startkit Command
disable-model-invocation: true
---
name: web-startkit

# Web Startkit Command

Next.js와 shadcn/ui 기반의 웹 스타터킷 프로젝트를 생성합니다.

## Technology Stack

- **Framework**: Next.js (App Router)
- **UI Library**: shadcn/ui
- **Styling**: Tailwind CSS
- **Type Safety**: TypeScript
- **Data Fetching**: TanStack Query (React Query)
- **State Management**: Jotai
- **Package Manager**: npm

## Project Structure

```
project/
├── app/                           # Next.js App Router
│   ├── layout.tsx                # Root layout
│   ├── page.tsx                  # Home page
│   └── globals.css               # Global styles
├── components/                    # ⚠️ UI 컴포넌트만 위치 (도메인 로직 금지)
│   ├── ui/                       # shadcn/ui 컴포넌트
│   └── shared/                   # 여러 feature에서 공유하는 공통 컴포넌트
├── features/                      # Domain-based feature modules
│   └── {domain}/                 # 도메인별 feature 모듈
│       ├── {ComponentName}.tsx   # Feature 컴포넌트
│       ├── _api/                 # API 정의
│       │   ├── types.ts          # Type 정의
│       │   └── queries.ts        # React Query options
│       ├── _hooks/               # Feature hooks
│       │   └── use{Name}.ts
│       ├── _stores/              # Feature state (Jotai)
│       │   └── {name}.ts
│       └── _utils/               # Feature utilities
│           └── {name}.ts
├── lib/                           # Global utilities & providers
│   ├── utils.ts                  # Global helper functions
│   ├── providers.tsx             # Global providers
│   ├── api/                      # ⚠️ 공통 API 설정만 (도메인 서비스 금지)
│   │   ├── client.ts             # Axios 인스턴스 설정
│   │   └── types.ts              # 공통 API 타입
│   ├── i18n/                     # 다국어 관련 (선택사항)
│   └── stores/                   # Global state (Jotai atoms)
└── public/                        # Static assets
```

### ⚠️ 컴포넌트 배치 규칙 (중요)

| 컴포넌트 유형   | 위치                 | 설명                                   |
| --------------- | -------------------- | -------------------------------------- |
| shadcn/ui       | `components/ui/`     | shadcn 라이브러리 컴포넌트만           |
| 공통 UI         | `components/shared/` | 여러 feature에서 재사용하는 순수 UI    |
| 도메인 컴포넌트 | `features/{domain}/` | **비즈니스 로직이 있는 모든 컴포넌트** |

```
❌ 금지: components/video/, components/search/, components/auth/
✅ 허용: features/video/, features/search/, features/auth/
```

### ⚠️ API/Hooks 배치 규칙 (중요)

| 유형                  | 위치                                  | 설명                           |
| --------------------- | ------------------------------------- | ------------------------------ |
| 공통 API 클라이언트   | `lib/api/client.ts`                   | Axios 설정, 인터셉터 등        |
| 공통 타입             | `lib/api/types.ts`                    | 여러 feature에서 공유하는 타입 |
| 도메인 API 서비스     | `features/{domain}/_api/`             | **도메인별 API 호출 로직**     |
| 도메인 API 엔드포인트 | `features/{domain}/_api/endpoints.ts` | **도메인별 API URL 상수**      |
| 도메인 Hooks          | `features/{domain}/_hooks/`           | **도메인별 커스텀 훅**         |

```
❌ 금지: lib/api/services/videoService.ts, lib/hooks/useVideos.ts
✅ 허용: features/video/_api/videoService.ts, features/video/_hooks/useVideos.ts
```

## Setup Instructions

1. **Initialize Next.js Project**

   ```bash
   npx create-next-app@latest project-name --typescript --tailwind --app --no-src-dir --import-alias "@/*"
   cd project-name
   ```

2. **Install shadcn/ui**

   ```bash
   npx shadcn@latest init
   ```

3. **Install Dependencies**
   ```bash
   npm install @tanstack/react-query
   npm install jotai
   npm install axios
   npm install react-hot-toast
   npm install class-variance-authority clsx tailwind-merge
   ```

---
name: web-startkit

## 코딩 규약

### 필수 규칙

- ✅ TypeScript 타입 정의 필수 (`type` 사용, `interface` 지양)
- ✅ 절대 경로 import (`@/`) 사용, 상대 경로 금지
- ✅ 컴포넌트는 export default로 작성
- ✅ 모든 컴포넌트는 React.memo로 감싸서 export

### 명명 규칙

**컴포넌트**

- PascalCase 사용
- export default 필수
- React.memo로 감싸서 export
- 파일명: `ComponentName.tsx`

```typescript
// ✅ Good - memo 사용
import { memo } from "react";

type MyComponentProps = {
  name: string;
};

const MyComponent = ({ name }: MyComponentProps) => {
  return <div>{name}</div>;
};

export default memo(MyComponent);
```

```typescript
// ❌ Bad - memo 없이 export
const MyComponent = ({ name }: MyComponentProps) => {
  return <div>{name}</div>;
};

export default MyComponent;
```

**함수 & 변수**

- 함수: `handleUploadImage`, `fetchUserData`
- Boolean: `isLoading`, `hasError`, `canEdit`
- Prop 함수: `onUploadImage`, `onSubmit`
- 상수: `UPPER_SNAKE_CASE`

**매개변수**

- 2개 이상이면 객체로 묶기

```typescript
// ❌ Bad
function updateUser(id: string, name: string, email: string, age: number) {}

// ✅ Good
function updateUser({ id, name, email, age }: UpdateUserParams) {}
```

### 파일 명명

- 컴포넌트: `ComponentName.tsx`
- Feature utilities: `_utils/{name}.ts` (camelCase)
- Feature hooks: `_hooks/use{Name}.ts`
- Feature API: `_api/{name}.ts`
- Global utilities: `lib/{name}.ts`

### 스타일링

```typescript
import { cn } from "@/lib/utils";
<div className={cn("mx-auto", "pb-4", isExpanded && "flex")} />;
```

---
name: web-startkit

## Features 아키텍처

### Feature 모듈 구조

각 feature는 독립적인 도메인 컨텍스트로, 해당 도메인에 필요한 모든 것을 포함:

- **Components**: 해당 feature에서만 사용되는 컴포넌트
- **\_utils**: feature 전용 유틸리티 함수
- **\_hooks**: feature 전용 커스텀 hooks
- **\_api**: feature 전용 API 호출 함수
- **\_stores**: feature 전용 상태 (Jotai atoms)

### Global vs Feature

**Global (lib/)**:

- 프로젝트 전체에서 사용되는 유틸리티
- 공통 헬퍼 함수 (cn, formatDate 등)
- 전역 providers, constants
- 전역 상태 (lib/stores/)

**Feature (features/)**:

- 특정 도메인에 종속된 로직
- 해당 feature 내에서만 사용
- 도메인별 응집도를 높임
- Feature 전용 상태 (\_stores/)

---
name: web-startkit

## 상태 관리 (Jotai)

### 전역 상태 정의

```typescript
// lib/stores/user.ts
import { atom } from "jotai";

type User = {
  id: string;
  name: string;
  email: string;
};

export const userAtom = atom<User | null>(null);
export const isAuthenticatedAtom = atom((get) => get(userAtom) !== null);
```

### Feature 전용 상태

```typescript
// features/auth/_stores/auth.ts
import { atom } from "jotai";

export const loginModalOpenAtom = atom(false);
export const authErrorAtom = atom<string | null>(null);
```

### 상태 사용

```typescript
// 컴포넌트에서 사용
import { useAtom, useAtomValue, useSetAtom } from "jotai";
import { userAtom } from "@/lib/stores/user";

// 읽기 + 쓰기
const [user, setUser] = useAtom(userAtom);

// 읽기만
const user = useAtomValue(userAtom);

// 쓰기만
const setUser = useSetAtom(userAtom);
```

### 파일 구조

- **Global Store**: `lib/stores/{name}.ts` - 앱 전역에서 사용
- **Feature Store**: `features/{name}/_stores/{name}.ts` - 특정 feature에서만 사용

---
name: web-startkit

## Hooks 가이드라인

### 비즈니스 로직 분리

- 500줄 이상 컴포넌트는 hooks 파일로 분리
- 파일명: `_hooks/use{ComponentName}.ts`
- Hook 이름: `use{ComponentName}`
- 위치: feature 폴더 내 `_hooks/` 디렉토리

### 원칙

- 단일 책임: 하나의 hook은 하나의 책임
- 타입 안전성: 매개변수와 반환값 타입 명시
- 최적화: 필요시 useMemo, useCallback 사용

---
name: web-startkit

## API 가이드 (React Query)

### API 클라이언트

프로젝트는 두 개의 Axios 인스턴스를 제공합니다:

| 클라이언트        | 위치                | 용도                                          |
| ----------------- | ------------------- | --------------------------------------------- |
| `apiClient`       | `lib/api/client.ts` | **인증이 필요한 API** (기본 사용)             |
| `apiPublicClient` | `lib/api/client.ts` | **인증이 불필요한 API** (로그인, 회원가입 등) |

#### apiClient (인증 포함)

- `user_token`과 `company`를 자동으로 추가
- GET/DELETE: query parameter로 추가
- POST/PUT/PATCH: request body에 추가
- 에러 핸들링 및 토스트 메시지 자동 표시

#### apiPublicClient (인증 불필요)

- 인증 정보를 추가하지 않음
- 로그인, 회원가입, 공개 API 등에 사용
- 에러 핸들링은 동일하게 적용

### 파일 구조

```
features/{name}/
├── _api/
│   ├── queries.ts  # React Query options 정의
│   └── types.ts    # Type 정의 (Interface 금지)
```

### queries.ts 작성 규칙

- GET: queryKey + queryFn 정의
- POST/PATCH/DELETE: mutationFn만 정의

#### 인증이 필요한 API (apiClient)

```typescript
// features/user/_api/queries.ts
import apiClient from "@/lib/api/client";

export const userQueries = {
  // GET 요청 - user_token이 query parameter로 자동 추가됨
  list: {
    queryKey: ["users"],
    queryFn: async () => {
      const { data } = await apiClient.get("/users");
      return data;
    },
  },
  detail: (id: string) => ({
    queryKey: ["users", id],
    queryFn: async () => {
      const { data } = await apiClient.get(`/users/${id}`);
      return data;
    },
  }),
};

export const userMutations = {
  // POST/PATCH/DELETE 요청 - user_token이 body에 자동 추가됨
  create: {
    mutationFn: async (data: CreateUserData) => {
      const { data: responseData } = await apiClient.post("/users", data);
      return responseData;
    },
  },
  update: {
    mutationFn: async ({ id, ...data }: UpdateUserData) => {
      const { data: responseData } = await apiClient.patch(
        `/users/${id}`,
        data
      );
      return responseData;
    },
  },
  delete: {
    mutationFn: async (id: string) => {
      await apiClient.delete(`/users/${id}`);
    },
  },
};
```

#### 인증이 불필요한 API (apiPublicClient)

```typescript
// features/auth/_api/queries.ts
import { apiPublicClient } from "@/lib/api/client";

type LoginData = {
  email: string;
  password: string;
};

type RegisterData = {
  email: string;
  password: string;
  name: string;
};

export const authMutations = {
  // 로그인 - user_token 없이 요청
  login: {
    mutationFn: async (data: LoginData) => {
      const { data: responseData } = await apiPublicClient.post(
        "/auth/login",
        data
      );
      return responseData;
    },
  },
  // 회원가입 - user_token 없이 요청
  register: {
    mutationFn: async (data: RegisterData) => {
      const { data: responseData } = await apiPublicClient.post(
        "/auth/register",
        data
      );
      return responseData;
    },
  },
};
```

> **참고**:
>
> - `apiClient`는 인증 토큰을 자동으로 추가합니다.
> - `apiPublicClient`는 인증 토큰 없이 요청합니다.
> - 두 클라이언트 모두 에러 핸들링과 토스트 메시지가 자동으로 처리됩니다.

### 컴포넌트에서 사용

```typescript
// features/user/_hooks/useUsers.ts
import { useQuery, useMutation } from "@tanstack/react-query";
import { userQueries, userMutations } from "../_api/queries";

export function useUsers() {
  const { data } = useQuery(userQueries.list);
  const { mutate: createUser } = useMutation(userMutations.create);

  return { data, createUser };
}
```

---
name: web-startkit

## shadcn/ui 사용

### 컴포넌트 추가

```bash
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add input
```

### 사용 예시

```typescript
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";

export default function Example() {
  return (
    <Card>
      <Button variant="default">Click me</Button>
    </Card>
  );
}
```

---
name: web-startkit

## 예시 코드

### Feature 예시: auth

프로젝트 생성 시 auth feature 예시를 포함할 수 있습니다:

- `features/auth/LoginForm.tsx` - 로그인 폼 컴포넌트
- `features/auth/_hooks/useAuth.ts` - 인증 관련 hook
- `features/auth/_api/queries.ts` - 인증 API 정의
- `features/auth/_stores/auth.ts` - 인증 상태 관리
- `features/auth/_utils/validation.ts` - 유효성 검증 유틸리티

이 구조를 참고하여 새로운 feature를 추가할 수 있습니다.

---
name: web-startkit

## 시작하기

### 개발 서버 실행

```bash
npm run dev
```

http://localhost:3000 에서 확인

### 빌드

```bash
npm run build
npm run start
```

### Lint

```bash
npm run lint
```

---
name: web-startkit

## 커밋 규칙

모든 커밋 메시지는 **한국어**로 작성합니다.

```
✅ 로그인 기능 추가
✅ 사용자 프로필 컴포넌트 개선
✅ API 에러 핸들링 개선
```

---
name: web-startkit

## Tasks

When this command is executed:

1. 사용자에게 프로젝트 이름과 위치 확인
2. Next.js 프로젝트 초기화
3. shadcn/ui 설정
4. 필수 dependencies 설치 (react-query, jotai, cn utils 등)
5. 기본 폴더 구조 생성 (features/, lib/)
6. lib/utils.ts, lib/providers.tsx 생성
7. 예시 feature 모듈 생성 (선택사항)
8. 설치 완료 및 다음 단계 안내
9. **npm run dev 실행 및 검증**
   - 개발 서버 실행
   - 정상 작동 확인 (에러 없이 실행되는지 체크)
   - 문제 발생 시 해결 후 재검증

Remember:

- TypeScript strict mode 사용
- 절대 경로 (`@/`) 사용
- Features 기반 도메인 구조
- `_` prefix로 private 폴더 표시
- React Query로 API 관리
- Jotai로 상태 관리
- **프로젝트 설정 완료 후 반드시 npm run dev로 검증**
