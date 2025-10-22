좋아요! 기초부터 차근차근 설명해드릴게요.

---

# Zod의 Union과 Discriminated Union

## 1. Union (유니온) - "이거 아니면 저거"

**유니온은 여러 타입 중 하나**를 의미합니다. "A **또는** B" 개념이에요.

### 기본 예제

```typescript
import { z } from 'zod';

// 문자열 또는 숫자
const StringOrNumber = z.union([
  z.string(),
  z.number()
]);

// 검증
StringOrNumber.parse("hello");  // ✅ 통과
StringOrNumber.parse(42);       // ✅ 통과
StringOrNumber.parse(true);     // ❌ 에러!

// 타입 추론
type StringOrNumber = z.infer<typeof StringOrNumber>;
// type StringOrNumber = string | number
```

### 실생활 예제 - 사용자 입력

```typescript
// 사용자가 이메일 또는 전화번호로 로그인
const LoginInput = z.union([
  z.string().email(),           // 이메일
  z.string().regex(/^\d{10}$/)  // 10자리 전화번호
]);

LoginInput.parse("user@example.com");  // ✅
LoginInput.parse("0101234567");        // ✅ (10자리)
LoginInput.parse("hello");             // ❌
```

### or 메서드로도 사용 가능

```typescript
// 이 두 가지는 완전히 같습니다
z.union([z.string(), z.number()])
z.string().or(z.number())
```

### 객체 유니온

```typescript
const Pet = z.union([
  z.object({
    species: z.string(),
    name: z.string()
  }),
  z.object({
    species: z.string(),
    age: z.number()
  })
]);

// 둘 다 가능
Pet.parse({ species: "dog", name: "멍멍이" });  // ✅
Pet.parse({ species: "cat", age: 3 });         // ✅
```

### Union의 동작 방식

Zod는 **순서대로 각 스키마를 시도**합니다:

```typescript
const Schema = z.union([
  z.string(),
  z.number(),
  z.boolean()
]);

Schema.parse("hello");
// 1. string 시도 → 성공! ✅

Schema.parse(true);
// 1. string 시도 → 실패
// 2. number 시도 → 실패
// 3. boolean 시도 → 성공! ✅
```

---

## 2. Discriminated Union (판별 유니온) - "태그로 구분하는 유니온"

**공통 필드**로 어떤 타입인지 구분할 수 있는 특별한 유니온입니다.

### 왜 필요한가? - Union의 문제점

```typescript
// 일반 union 사용
const Animal = z.union([
  z.object({
    type: z.literal("dog"),
    bark: z.string()
  }),
  z.object({
    type: z.literal("cat"),
    meow: z.string()
  })
]);

// 문제: 모든 스키마를 순서대로 시도 (느림)
Animal.parse({ type: "dog", bark: "왈왈" });
// 1단계: cat 스키마 시도 → 실패
// 2단계: dog 스키마 시도 → 성공
```

### Discriminated Union의 해결책

```typescript
// discriminatedUnion 사용
const Animal = z.discriminatedUnion("type", [
  z.object({
    type: z.literal("dog"),
    bark: z.string()
  }),
  z.object({
    type: z.literal("cat"),
    meow: z.string()
  })
]);

// 'type' 필드를 보고 바로 올바른 스키마 선택! (빠름)
Animal.parse({ type: "dog", bark: "왈왈" });
// type: "dog" 확인 → 즉시 dog 스키마 적용 ✅
```

### 실생활 예제 1 - API 응답

```typescript
const ApiResponse = z.discriminatedUnion("status", [
  z.object({
    status: z.literal("success"),
    data: z.string()
  }),
  z.object({
    status: z.literal("error"),
    error: z.string(),
    code: z.number()
  })
]);

// 성공 응답
ApiResponse.parse({
  status: "success",
  data: "처리 완료"
});  // ✅

// 에러 응답
ApiResponse.parse({
  status: "error",
  error: "권한 없음",
  code: 403
});  // ✅

// 타입 추론도 똑똑해집니다
type ApiResponse = z.infer<typeof ApiResponse>;

function handleResponse(response: ApiResponse) {
  if (response.status === "success") {
    // TypeScript가 자동으로 타입을 좁혀줌
    console.log(response.data);  // ✅ data 필드 접근 가능
    // console.log(response.error);  // ❌ 에러 - error 필드 없음
  } else {
    console.log(response.error);  // ✅ error 필드 접근 가능
    console.log(response.code);   // ✅ code 필드 접근 가능
    // console.log(response.data);  // ❌ 에러 - data 필드 없음
  }
}
```

### 실생활 예제 2 - 결제 수단

```typescript
const Payment = z.discriminatedUnion("method", [
  z.object({
    method: z.literal("card"),
    cardNumber: z.string(),
    cvv: z.string()
  }),
  z.object({
    method: z.literal("bank"),
    accountNumber: z.string(),
    bankCode: z.string()
  }),
  z.object({
    method: z.literal("kakao"),
    kakaoId: z.string()
  })
]);

// 카드 결제
Payment.parse({
  method: "card",
  cardNumber: "1234-5678-9012-3456",
  cvv: "123"
});  // ✅

// 은행 계좌 결제
Payment.parse({
  method: "bank",
  accountNumber: "110-123-456789",
  bankCode: "004"
});  // ✅
```

### 실생활 예제 3 - 알림 타입

```typescript
const Notification = z.discriminatedUnion("type", [
  z.object({
    type: z.literal("email"),
    to: z.string().email(),
    subject: z.string(),
    body: z.string()
  }),
  z.object({
    type: z.literal("sms"),
    phoneNumber: z.string(),
    message: z.string()
  }),
  z.object({
    type: z.literal("push"),
    deviceToken: z.string(),
    title: z.string(),
    message: z.string()
  })
]);

function sendNotification(notification: z.infer<typeof Notification>) {
  if (notification.type === "email") {
    // 이메일 발송 로직
    console.log(`이메일 발송: ${notification.to}`);
    console.log(`제목: ${notification.subject}`);
  } else if (notification.type === "sms") {
    // SMS 발송 로직
    console.log(`SMS 발송: ${notification.phoneNumber}`);
  } else {
    // 푸시 발송 로직
    console.log(`푸시 알림: ${notification.title}`);
  }
}
```

---

## Union vs Discriminated Union 비교

| 특징 | Union | Discriminated Union |
|------|-------|---------------------|
| **사용 시기** | 타입을 구분하는 공통 필드가 없을 때 | 공통 필드(discriminator)로 구분 가능할 때 |
| **성능** | 모든 스키마를 순서대로 시도 (느림) | discriminator로 즉시 매칭 (빠름) |
| **에러 메시지** | 모든 실패한 스키마의 에러 나열 | 명확한 discriminator 에러 |
| **타입 추론** | 일반적인 유니온 타입 | 자동 타입 좁히기(narrowing) |

### 언제 뭘 쓸까?

```typescript
// ✅ Union 사용: 구조가 완전히 다른 경우
const Input = z.union([
  z.string(),
  z.number(),
  z.boolean()
]);

// ✅ Discriminated Union 사용: 공통 태그 필드가 있는 경우
const Shape = z.discriminatedUnion("kind", [
  z.object({ kind: z.literal("circle"), radius: z.number() }),
  z.object({ kind: z.literal("square"), size: z.number() }),
  z.object({ kind: z.literal("rectangle"), width: z.number(), height: z.number() })
]);
```

---

## 핵심 정리

### Union
- "A 또는 B" - 여러 타입 중 하나
- 순서대로 시도
- 간단한 타입 조합에 적합

### Discriminated Union
- 공통 필드(태그)로 구분
- 즉시 매칭 (성능 좋음)
- 객체들의 유니온에 적합
- TypeScript 타입 좁히기 지원

**팁**: 객체들의 유니온을 만들고 공통 필드가 있다면, 무조건 `discriminatedUnion`을 쓰세요! 성능도 좋고 타입도 안전합니다.
