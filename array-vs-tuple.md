# TypeScript의 Tuple (튜플)

튜플을 처음부터 차근차근 설명해드릴게요!

---

## 1. Array vs Tuple - 근본적인 차이

### Array (배열) - "같은 타입의 요소들"

```typescript
// 배열: 길이가 가변적, 모든 요소가 같은 타입
const numbers: number[] = [1, 2, 3];
numbers.push(4);  // ✅ 가능
numbers.push(5);  // ✅ 가능
// 길이: 자유롭게 변경 가능

const names: string[] = ["철수", "영희"];
names[0];  // string 타입
names[1];  // string 타입
names[99]; // string | undefined 타입
```

### Tuple (튜플) - "정해진 순서의 서로 다른 타입들"

```typescript
// 튜플: 길이가 고정, 각 위치마다 다른 타입 가능
const person: [string, number] = ["철수", 25];
//            ^^^^^^^^^^^^^^^^   ^^^^^^^^^^^^^
//            타입 정의            값

person[0];  // string 타입 (정확히 알고 있음)
person[1];  // number 타입 (정확히 알고 있음)
person[2];  // ❌ 에러! 인덱스 2는 존재하지 않음
```

---

## 2. Tuple 기본 사용법

### 기본 선언

```typescript
// 이름, 나이
type Person = [string, number];

const user1: Person = ["김철수", 30];     // ✅
const user2: Person = ["이영희", 25];     // ✅
const user3: Person = [30, "박민수"];     // ❌ 순서가 틀림
const user4: Person = ["최지수"];         // ❌ 길이가 틀림
```

### 여러 타입 조합

```typescript
// 위도, 경도
type Coordinate = [number, number];
const seoul: Coordinate = [37.5665, 126.9780];

// ID, 이름, 활성화 여부
type User = [number, string, boolean];
const admin: User = [1, "관리자", true];

// 다양한 타입 혼합
type Mixed = [string, number, boolean, string[]];
const data: Mixed = ["hello", 42, true, ["a", "b", "c"]];
```

---

## 3. 실생활 예제

### 예제 1: 함수 리턴값 (useState 패턴)

```typescript
// React의 useState와 비슷한 패턴
function useState<T>(initial: T): [T, (value: T) => void] {
  let state = initial;
  
  const setState = (value: T) => {
    state = value;
  };
  
  return [state, setState];  // 튜플 반환!
}

const [count, setCount] = useState(0);
//     ^^^^^  ^^^^^^^^
//     number  (value: number) => void

setCount(10);      // ✅
setCount("hello"); // ❌ 타입 에러
```

### 예제 2: 여러 값 반환

```typescript
// 사용자 정보 조회 - 성공/실패 여부와 데이터를 함께 반환
function getUser(id: number): [boolean, string | null] {
  if (id > 0) {
    return [true, "김철수"];   // 성공
  }
  return [false, null];        // 실패
}

const [success, userName] = getUser(1);
if (success) {
  console.log(userName);  // string | null
}
```

### 예제 3: RGB 색상

```typescript
type RGB = [number, number, number];

const red: RGB = [255, 0, 0];
const green: RGB = [0, 255, 0];
const blue: RGB = [0, 0, 255];

function createColor(r: number, g: number, b: number): RGB {
  return [r, g, b];
}

const purple = createColor(128, 0, 128);
```

### 예제 4: 데이터베이스 결과

```typescript
// [성공여부, 데이터, 에러메시지]
type QueryResult<T> = [boolean, T | null, string | null];

function query(sql: string): QueryResult<any> {
  try {
    const data = { id: 1, name: "철수" };
    return [true, data, null];
  } catch (error) {
    return [false, null, "쿼리 실행 실패"];
  }
}

const [ok, data, error] = query("SELECT * FROM users");
if (ok) {
  console.log(data);
} else {
  console.log(error);
}
```

---

## 4. Optional과 Rest Elements

### Optional 요소 (선택적 요소)

```typescript
// 이름은 필수, 나이는 선택
type PersonOptional = [string, number?];

const p1: PersonOptional = ["철수", 30];   // ✅
const p2: PersonOptional = ["영희"];       // ✅ 나이 생략 가능
```

### Rest Elements (나머지 요소)

```typescript
// 첫 번째는 문자열, 나머지는 숫자들
type StringAndNumbers = [string, ...number[]];

const data1: StringAndNumbers = ["hello"];              // ✅
const data2: StringAndNumbers = ["hello", 1, 2, 3];     // ✅
const data3: StringAndNumbers = ["hello", 1, 2, 3, 4, 5]; // ✅

// 최소 2개 이상 보장
type AtLeastTwo = [number, number, ...number[]];

const nums1: AtLeastTwo = [1];           // ❌ 최소 2개 필요
const nums2: AtLeastTwo = [1, 2];        // ✅
const nums3: AtLeastTwo = [1, 2, 3, 4];  // ✅
```

### 실전 예제 - 로그 함수

```typescript
// 첫 번째는 로그 레벨, 나머지는 메시지들
type LogEntry = ['INFO' | 'WARN' | 'ERROR', ...string[]];

function log(...args: LogEntry) {
  const [level, ...messages] = args;
  console.log(`[${level}]`, ...messages);
}

log('INFO', '서버 시작됨');                    // ✅
log('ERROR', '에러 발생:', '상세 내용');        // ✅
log('WARN');                                  // ❌ 메시지가 없어도 됨? 
                                              // (실제로는 타입상 가능)
```

---

## 5. Array vs Tuple 상세 비교

```typescript
// 배열
const arr: string[] = ["a", "b", "c"];
arr.length;  // number 타입 (정확한 값 모름)
arr[0];      // string 타입
arr[10];     // string | undefined 타입
arr.push("d"); // ✅ 가능

// 튜플
const tuple: [string, number] = ["hello", 42];
tuple.length;  // 2 타입 (정확히 2)
tuple[0];      // string 타입 (정확히 string)
tuple[1];      // number 타입 (정확히 number)
tuple[2];      // ❌ 타입 에러
tuple.push("x"); // ⚠️ 런타임에는 가능하지만 타입 시스템에서는 권장 안 함
```

---

## 6. Zod에서 Tuple 사용

### Zod Tuple 스키마

```typescript
import { z } from 'zod';

// [문자열, 숫자] 튜플
const PersonSchema = z.tuple([
  z.string(),
  z.number()
]);

PersonSchema.parse(["철수", 30]);    // ✅
PersonSchema.parse(["영희", "20"]);  // ❌ 두 번째는 숫자여야 함
PersonSchema.parse(["민수"]);        // ❌ 길이가 2여야 함

type Person = z.infer<typeof PersonSchema>;
// type Person = [string, number]
```

### Rest elements와 함께

```typescript
// 첫 번째는 문자열, 나머지는 숫자들
const Schema = z.tuple([z.string()]).rest(z.number());

Schema.parse(["hello", 1, 2, 3]);  // ✅
Schema.parse(["hello"]);           // ✅
Schema.parse([1, 2, 3]);           // ❌ 첫 번째는 문자열이어야 함
```

---

## 7. Zod의 discriminatedUnion이 튜플을 요구하는 이유

이제 여기서 핵심이 나옵니다!

```typescript
// Zod의 discriminatedUnion 시그니처
z.discriminatedUnion<Discriminator, Options>(
  discriminator: Discriminator,
  options: [Options[number], ...Options[number][]]
  //       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  //       튜플! 최소 1개 이상 보장
)
```

### 배열로는 안 되는 이유

```typescript
// ❌ 배열 타입 - 빈 배열 가능
const schemas: Array<ZodSchema> = [];
z.discriminatedUnion('type', schemas);  // 런타임 에러 가능!

// ✅ 튜플 타입 - 최소 1개 보장
const schemas: [ZodSchema, ...ZodSchema[]] = []; // ❌ 타입 에러!
const schemas: [ZodSchema, ...ZodSchema[]] = [schema1]; // ✅
```

### 실제 사용

```typescript
// 배열로 만든 후
const schemaArray = ['a', 'b', 'c'].map(createSchema);

// 튜플로 캐스팅 (최소 1개 보장)
z.discriminatedUnion(
  'type',
  schemaArray as [ZodSchema, ...ZodSchema[]]
);
```

---

## 8. 핵심 정리

### Tuple의 특징
1. **길이 고정**: 정확한 개수 지정
2. **위치별 타입**: 각 인덱스마다 다른 타입 가능
3. **타입 안전성**: 컴파일 타임에 인덱스 체크
4. **구조 분해**: `const [a, b] = tuple` 패턴

### 언제 사용?
- ✅ 함수에서 여러 값 반환
- ✅ 고정된 구조의 데이터 (좌표, RGB 등)
- ✅ 타입이 다른 값들의 조합
- ✅ 최소 개수 보장이 필요할 때

### Array vs Tuple
| | Array | Tuple |
|---|-------|-------|
| **길이** | 가변 | 고정 (또는 최소 개수) |
| **타입** | 모두 동일 | 위치마다 다를 수 있음 |
| **사용** | 동일한 항목의 리스트 | 서로 다른 항목의 조합 |

이제 튜플이 뭔지, 왜 Zod가 튜플을 요구하는지 이해되셨나요?
