
# zod 에서 object와 record의 차이

## `z.object()` - 고정된 키 구조

`z.object()`는 **특정 키들을 명시적으로 정의**할 때 사용합니다. 각 키의 이름과 타입을 정확히 지정합니다.

```typescript
const UserSchema = z.object({
  name: z.string(),
  age: z.number(),
  email: z.string().email()
});

// ✅ 통과
UserSchema.parse({
  name: "John",
  age: 30,
  email: "john@example.com"
});

// ❌ 실패 - 정의되지 않은 키
UserSchema.parse({
  name: "John",
  age: 30,
  email: "john@example.com",
  nickname: "Johnny" // 에러!
});
```

## `z.record()` - 동적 키 구조

`z.record()`는 **키가 동적**이고 런타임에 결정될 때 사용합니다. 키-값 쌍의 패턴만 정의합니다.

```typescript
// 모든 키는 string, 모든 값은 number
const ScoresSchema = z.record(z.number());

// ✅ 통과 - 어떤 키든 가능
ScoresSchema.parse({
  math: 95,
  science: 88,
  english: 92
});

// 키와 값 타입 모두 지정
const MetadataSchema = z.record(z.string(), z.boolean());

MetadataSchema.parse({
  isActive: true,
  isVerified: false,
  hasAccess: true
});
```

## 주요 차이점

| 특성 | `z.object()` | `z.record()` |
|------|-------------|--------------|
| **키** | 미리 정의된 고정 키 | 동적 키 (런타임에 결정) |
| **사용 사례** | 구조가 명확한 데이터 | 키를 미리 알 수 없는 딕셔너리/맵 |
| **타입 안정성** | 각 키마다 다른 타입 가능 | 모든 값이 같은 타입 |
| **추가 키** | 기본적으로 거부 | 허용됨 |

## 실전 예시

```typescript
// object - API 응답 스키마
const ApiResponseSchema = z.object({
  status: z.number(),
  data: z.object({
    id: z.string(),
    name: z.string()
  }),
  meta: z.record(z.string()) // 메타데이터는 동적
});

// record - 언어별 번역
const TranslationsSchema = z.record(z.string());
// { "en": "Hello", "ko": "안녕", "ja": "こんにちは" }

// record - 사용자 ID를 키로 하는 권한 맵
const PermissionsSchema = z.record(
  z.string(), // userId
  z.object({
    canRead: z.boolean(),
    canWrite: z.boolean()
  })
);
```

**요약**: 키를 미리 알고 있다면 `z.object()`, 키가 동적이라면 `z.record()`를 사용하세요!


# infer

```js
import { z } from 'zod';

// 1. 먼저 Zod 스키마 정의
const UserSchema = z.object({
  name: z.string(),
  age: z.number(),
  email: z.string().email(),
});

// 2. 스키마로부터 타입 추론
type User = z.infer<typeof UserSchema>;

// 결과적으로 User 타입은 다음과 같이 추론됨:
// type User = {
//   name: string;
//   age: number;
//   email: string;
// }
```

Zod 스키마를 타입으로 변환시켜주는 역할을 한다


# as const

```js
const config = {
  NAVIGATE: RouteObjSchema,
  KAKAO_INVITE: z.undefined(),
};

// TypeScript 추론 타입:
// const config: {
//   NAVIGATE: ZodObject<...>;
//   KAKAO_INVITE: ZodUndefined;
// }

type Keys = keyof typeof config;
// type Keys = string  ❌ 너무 넓은 타입!
```

```js
const config = {
  NAVIGATE: RouteObjSchema,
  KAKAO_INVITE: z.undefined(),
} as const;

// TypeScript 추론 타입:
// const config: {
//   readonly NAVIGATE: typeof RouteObjSchema;
//   readonly KAKAO_INVITE: ZodUndefined;
// }

type Keys = keyof typeof config;
// type Keys = "NAVIGATE" | "KAKAO_INVITE"  ✅ 정확한 리터럴 타입!
```

객체의 키를 원래 string 으로 간주하는데 as const 로 인해 해당 객체의 key type을 조회시, 정확한 객체의 키들의 순회로 타입을 얻어낼 수 있다



# 'in' in type

```js
export type DisplayActions = {
  [T in keyof typeof DisplayActionsSchema]: z.infer
    (typeof DisplayActionsSchema)[T]
  >;
};
```

```js
// [T in Keys]: Value 형태
// "Keys의 각 요소 T에 대해, T를 키로 하고 Value를 값으로 하는 타입"

type Example = {
  [T in 'a' | 'b' | 'c']: string;
}

// 결과:
// type Example = {
//   a: string;
//   b: string;
//   c: string;
// }
```

```js
// 예시 1: 간단한 매핑
type Person = {
  name: string;
  age: number;
};

type ReadonlyPerson = {
  [K in keyof Person]: Person[K]
}
```
