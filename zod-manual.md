

## 개요
Zod는 TypeScript-first 스키마 선언 및 검증 라이브러리입니다. 런타임 검증과 TypeScript 타입을 동시에 제공합니다.

## 기본 타입 (Primitives)

### 문자열 (String)
```typescript
z.string()
z.string().min(5)
z.string().max(20)
z.string().length(10)
z.string().email()
z.string().url()
z.string().uuid()
z.string().cuid()
z.string().regex(/^\d+$/)
z.string().includes("test")
z.string().startsWith("https://")
z.string().endsWith(".com")
z.string().datetime()
z.string().ip()
z.string().trim()
z.string().toLowerCase()
z.string().toUpperCase()
```

### 숫자 (Number)
```typescript
z.number()
z.number().int()
z.number().positive()
z.number().nonnegative()
z.number().negative()
z.number().nonpositive()
z.number().multipleOf(5)
z.number().finite()
z.number().safe()
z.number().min(5)
z.number().max(100)
z.number().gt(5)  // greater than
z.number().gte(5) // greater than or equal
z.number().lt(5)  // less than
z.number().lte(5) // less than or equal
```

### 불리언 (Boolean)
```typescript
z.boolean()
```

### 날짜 (Date)
```typescript
z.date()
z.date().min(new Date("2020-01-01"))
z.date().max(new Date("2030-01-01"))
```

### BigInt
```typescript
z.bigint()
z.bigint().positive()
z.bigint().negative()
z.bigint().nonnegative()
z.bigint().nonpositive()
z.bigint().multipleOf(5n)
```

### Symbol
```typescript
z.symbol()
```

### Undefined, Null, Void
```typescript
z.undefined()
z.null()
z.void() // undefined를 허용
```

### Any, Unknown
```typescript
z.any()
z.unknown()
```

### Never
```typescript
z.never() // 아무것도 허용하지 않음
```

## 리터럴 타입 (Literals)

```typescript
z.literal("test")
z.literal(42)
z.literal(true)
const Direction = z.literal("up").or(z.literal("down"))
```

## 배열 (Arrays)

```typescript
z.array(z.string())
z.array(z.number()).min(1)
z.array(z.string()).max(5)
z.array(z.string()).length(3)
z.array(z.string()).nonempty()
z.string().array() // z.array(z.string())의 축약형
```

## 객체 (Objects)

### 기본 객체
```typescript
const User = z.object({
  name: z.string(),
  age: z.number(),
  email: z.string().email()
})
```

### 객체 메서드
```typescript
// shape: 스키마 가져오기
User.shape.name // z.string() 반환

// keyof: 키 유니온 타입
const UserKeys = User.keyof() // z.union([z.literal("name"), z.literal("age"), z.literal("email")])

// extend: 확장
const ExtendedUser = User.extend({
  address: z.string()
})

// merge: 병합
const Address = z.object({ street: z.string() })
const UserWithAddress = User.merge(Address)

// pick: 선택
const NameOnly = User.pick({ name: true })

// omit: 제외
const UserWithoutEmail = User.omit({ email: true })

// partial: 모든 필드를 선택적으로
const PartialUser = User.partial()

// deepPartial: 중첩된 객체까지 선택적으로
const DeepPartialUser = User.deepPartial()

// required: 모든 필드를 필수로
const RequiredUser = User.required()

// passthrough: 정의되지 않은 키 통과
const PassthroughUser = User.passthrough()

// strict: 정의되지 않은 키 에러 (기본값)
const StrictUser = User.strict()

// strip: 정의되지 않은 키 제거
const StrippedUser = User.strip()

// catchall: 정의되지 않은 모든 키에 대한 스키마
const CatchallUser = User.catchall(z.string())
```

## 튜플 (Tuples)

```typescript
z.tuple([z.string(), z.number(), z.boolean()])

// rest를 사용한 가변 길이 튜플
z.tuple([z.string(), z.number()]).rest(z.boolean())

// with generic
type Tuple<T, N extends number> = // T: 요소 타입, N: 길이
```

## 유니온 (Unions)

```typescript
z.union([z.string(), z.number()])
// or 메서드 사용
z.string().or(z.number())
```

## 판별 유니온 (Discriminated Unions)

```typescript
const Response = z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("error"), error: z.string() })
])
```

## 교차 타입 (Intersections)

```typescript
const Person = z.object({ name: z.string() })
const Employee = z.object({ employeeId: z.number() })
const EmployeePerson = z.intersection(Person, Employee)
// and 메서드 사용
const EmployeePerson2 = Person.and(Employee)
```

## 레코드 (Records)

```typescript
z.record(z.string()) // 모든 값이 string
z.record(z.string(), z.number()) // 키는 string, 값은 number
```

## 맵 (Maps)

```typescript
z.map(z.string(), z.number())
```

## 세트 (Sets)

```typescript
z.set(z.string())
z.set(z.number()).min(1).max(5)
```

## Enum

```typescript
// Zod enum
const Status = z.enum(["pending", "approved", "rejected"])

// Native enum
enum NativeStatus { Pending, Approved, Rejected }
const ZodNativeEnum = z.nativeEnum(NativeStatus)
```

## Optional & Nullable

```typescript
z.string().optional() // string | undefined
z.string().nullable() // string | null
z.string().nullish() // string | null | undefined
```

## Default 값

```typescript
z.string().default("default value")
z.number().default(() => Math.random()) // 함수로 default 제공
```

## Catch (에러 시 대체 값)

```typescript
z.string().catch("fallback")
```

## Preprocessing

```typescript
z.preprocess(
  (val) => String(val).toUpperCase(),
  z.string()
)
```

## Transform

```typescript
z.string().transform((val) => val.length)
z.string().transform((val) => val.toUpperCase())
```

## Pipe (연속 검증)

```typescript
z.string()
  .transform((val) => val.length)
  .pipe(z.number().min(5))
```

## Refinements (사용자 정의 검증)

```typescript
// refine
z.string().refine((val) => val.length > 5, {
  message: "String must be longer than 5 characters"
})

// superRefine (더 세밀한 제어)
z.object({
  password: z.string(),
  confirmPassword: z.string()
}).superRefine((data, ctx) => {
  if (data.password !== data.confirmPassword) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Passwords don't match",
      path: ["confirmPassword"]
    });
  }
})
```

## Coercion (형변환)

```typescript
z.coerce.string() // String()으로 변환
z.coerce.number() // Number()로 변환
z.coerce.boolean() // Boolean()으로 변환
z.coerce.date() // new Date()로 변환
```

## Promise

```typescript
z.promise(z.string()) // Promise<string>
```

## Function

```typescript
z.function()
  .args(z.string(), z.number())
  .returns(z.boolean())

// implement로 구현 제공
const myFunction = z.function()
  .args(z.string())
  .returns(z.number())
  .implement((x) => x.length)
```

## Lazy (재귀적 스키마)

```typescript
const Node: z.ZodType<any> = z.lazy(() =>
  z.object({
    value: z.string(),
    children: z.array(Node)
  })
)
```

## Effects (부작용)

```typescript
z.string()
  .refine((val) => val.length > 5)
  .transform((val) => val.toUpperCase())
```

## 브랜딩 (Branding)

```typescript
const UserId = z.string().brand<"UserId">()
type UserId = z.infer<typeof UserId>
```

## 파싱 메서드

```typescript
const schema = z.string()

// parse: 실패 시 에러 발생
schema.parse("hello") // "hello"
schema.parse(123) // throws ZodError

// safeParse: Result 객체 반환
const result = schema.safeParse("hello")
if (result.success) {
  console.log(result.data)
} else {
  console.log(result.error)
}

// parseAsync: 비동기 파싱
await schema.parseAsync("hello")

// safeParseAsync: 비동기 safe 파싱
const asyncResult = await schema.safeParseAsync("hello")
```

## 타입 추론

```typescript
const User = z.object({
  name: z.string(),
  age: z.number()
})

// 스키마에서 TypeScript 타입 추론
type User = z.infer<typeof User>

// input 타입 추론 (transform 전)
type UserInput = z.input<typeof User>

// output 타입 추론 (transform 후)
type UserOutput = z.output<typeof User>
```

## 에러 처리

```typescript
try {
  schema.parse(data)
} catch (err) {
  if (err instanceof z.ZodError) {
    console.log(err.errors) // 에러 배열
    console.log(err.format()) // 포맷된 에러
    console.log(err.flatten()) // 평탄화된 에러
  }
}

// 커스텀 에러 메시지
z.string({
  required_error: "Name is required",
  invalid_type_error: "Name must be a string"
}).min(3, { message: "Name must be at least 3 characters" })
```

## 유틸리티

```typescript
// instanceof 체크
if (schema instanceof z.ZodString) {
  // string 스키마
}

// isOptional 체크
schema.isOptional()

// isNullable 체크
schema.isNullable()

// unwrap: optional/nullable 제거
z.string().optional().unwrap() // z.string() 반환
```

## 환경 변수 검증 예시

```typescript
const envSchema = z.object({
  NODE_ENV: z.enum(["development", "production", "test"]),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1)
})

const env = envSchema.parse(process.env)
```

이 가이드는 Zod의 주요 기능과 메서드를 포괄적으로 다루고 있습니다. 각 메서드는 체이닝이 가능하며, 복잡한 검증 로직을 구성할 수 있습니다.
