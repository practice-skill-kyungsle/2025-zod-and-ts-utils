

## 핵심 유틸리티 타입

### Partial - 모든 속성을 선택적으로
```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; }

// 실무: API 업데이트 요청
function updateUser(id: number, data: Partial<User>) {
  // 일부 필드만 업데이트
}
```

### Required - 모든 속성을 필수로
```typescript
interface Config {
  apiUrl?: string;
  timeout?: number;
}

type RequiredConfig = Required<Config>;
// { apiUrl: string; timeout: number; }
```

### Pick - 특정 속성만 선택
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type UserPublicInfo = Pick<User, "id" | "name" | "email">;
// { id: number; name: string; email: string; }
```

### Omit - 특정 속성 제외
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type UserWithoutPassword = Omit<User, "password">;
// { id: number; name: string; email: string; }
```

### Readonly - 읽기 전용으로 만들기
```typescript
interface State {
  count: number;
  items: string[];
}

type ReadonlyState = Readonly<State>;
// { readonly count: number; readonly items: string[]; }
```

## 유니온 타입 유틸리티

### Exclude - 유니온에서 특정 타입 제외
```typescript
type Status = "pending" | "approved" | "rejected" | "draft";
type PublishedStatus = Exclude<Status, "draft">;
// "pending" | "approved" | "rejected"

// 실무: 시스템 전용 액션 제외
type AllActions = "create" | "read" | "update" | "delete" | "system_refresh";
type UserActions = Exclude<AllActions, "system_refresh">;
```

### Extract - 유니온에서 특정 타입만 추출
```typescript
type Status = "pending" | "approved" | "rejected" | 200 | 404;
type StringStatus = Extract<Status, string>;
// "pending" | "approved" | "rejected"
```

## 함수 관련 유틸리티

### ReturnType - 함수 반환 타입
```typescript
function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string; }
```

### Parameters - 함수 매개변수 타입
```typescript
function createUser(name: string, age: number, email: string) {
  // ...
}

type CreateUserParams = Parameters<typeof createUser>;
// [string, number, string]
```

## 객체 타입 유틸리티

### Record - 키-값 쌍 객체 생성
```typescript
type Role = "admin" | "user" | "guest";
type Permissions = Record<Role, string[]>;
// { admin: string[]; user: string[]; guest: string[]; }

// 실무: 상태 관리
type LoadingStates = Record<string, boolean>;
const loading: LoadingStates = {
  users: true,
  posts: false
};
```

### keyof - 객체 키 타입 추출
```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type UserKeys = keyof User;
// "id" | "name" | "email"

// 실무: 동적 속성 접근
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

## 조건부 타입

### 기본 조건부 타입
```typescript
type IsString<T> = T extends string ? true : false;

type Test1 = IsString<"hello">; // true
type Test2 = IsString<42>; // false
```

### NonNullable - null/undefined 제거
```typescript
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>;
// string
```

## 실무 필수 패턴

### API 응답 타입
```typescript
// 성공/실패 분기
type ApiResponse<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

// 페이지네이션
interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
}
```

### 폼 데이터 타입
```typescript
// 원본 데이터
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

// 생성 시 (id, createdAt 제외)
type CreateUserInput = Omit<User, "id" | "createdAt">;

// 수정 시 (일부만 수정 가능)
type UpdateUserInput = Partial<Omit<User, "id" | "createdAt">>;
```

### 상태 관리 타입
```typescript
// 로딩 상태 관리
interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

// 액션 타입 정의
type Action<T extends string, P = void> = P extends void
  ? { type: T }
  : { type: T; payload: P };
```

### 유틸리티 조합
```typescript
interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  stock: number;
  createdAt: Date;
  updatedAt: Date;
}

// 목록 표시용 (일부 필드만)
type ProductListItem = Pick<Product, "id" | "name" | "price">;

// 수정 폼용 (시스템 필드 제외, 선택적)
type ProductEditForm = Partial<Omit<Product, "id" | "createdAt" | "updatedAt">>;

// 생성 폼용 (시스템 필드 제외, 필수)
type ProductCreateForm = Omit<Product, "id" | "createdAt" | "updatedAt">;
```

### 타입 가드와 함께 사용
```typescript
// 배열 타입 체크
function isArray<T>(value: T | T[]): value is T[] {
  return Array.isArray(value);
}

// null 체크
function isNotNull<T>(value: T | null): value is T {
  return value !== null;
}

// 실무: 필터링에 활용
const values = [1, null, 2, undefined, 3];
const filtered = values.filter(isNotNull); // number[]
```

### Discriminated Union (구별된 유니온)
```typescript
type Result<T> = 
  | { status: "success"; data: T }
  | { status: "error"; error: string }
  | { status: "loading" };

function handleResult<T>(result: Result<T>) {
  switch (result.status) {
    case "success":
      console.log(result.data); // T 타입
      break;
    case "error":
      console.log(result.error); // string 타입
      break;
    case "loading":
      console.log("Loading...");
      break;
  }
}
```

### as const와 함께 사용
```typescript
const ROUTES = {
  HOME: "/",
  ABOUT: "/about",
  CONTACT: "/contact"
} as const;

type Route = typeof ROUTES[keyof typeof ROUTES];
// "/" | "/about" | "/contact"
```

이 가이드는 실무에서 자주 사용되는 TypeScript 유틸리티 타입들을 정리한 것입니다. 복잡한 타입 체조보다는 실제 프로젝트에서 바로 활용 가능한 패턴들에 초점을 맞췄습니다.
