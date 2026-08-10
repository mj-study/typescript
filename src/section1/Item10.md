# 아이템10. 객체 래퍼 타입 피하기

> JavaScript의 원시 타입(string, number, boolean 등)에는 메서드가 있는 것처럼 보이지만, 실제로는 래퍼 객체(String, Number, Boolean)로 감싸서 메서드를 제공함
> TypeScript에서는 원시 타입과 래퍼 타입을 구분하므로, 항상 원시 타입(소문자)을 사용해야 함

## 1. 원시 타입과 래퍼 객체

JavaScript에는 7가지 원시 타입이 있습니다:
- `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`

원시 타입은 불변(immutable)이며 메서드를 가지지 않습니다. 그런데 어떻게 `string`에서 메서드를 호출할 수 있을까요?

```ts
// string은 원시 타입인데 메서드가 동작한다?
'hello'.charAt(0); // 'h'
'hello'.toUpperCase(); // 'HELLO'
```

### 래퍼 객체의 자동 변환

JavaScript는 원시 타입에서 메서드를 호출할 때, 자동으로 래퍼 객체로 변환합니다:

```ts
// 실제로 일어나는 일
const s = 'hello';
s.charAt(0);
// 내부적으로: (new String(s)).charAt(0)
// 메서드 호출 후 래퍼 객체는 버려짐
```

✅ 원시 타입의 메서드 호출 시, JavaScript 엔진이 자동으로 래퍼 객체를 생성하고 메서드 실행 후 버림

## 2. 원시 타입 vs 래퍼 객체 타입

| 원시 타입 | 래퍼 객체 타입 |
|----------|---------------|
| `string` | `String` |
| `number` | `Number` |
| `boolean` | `Boolean` |
| `symbol` | `Symbol` |
| `bigint` | `BigInt` |

### 래퍼 객체의 특이한 동작

```ts
// 래퍼 객체는 원시 값과 다르게 동작
const strPrimitive = 'hello';
const strObject = new String('hello');

console.log(typeof strPrimitive); // "string"
console.log(typeof strObject);    // "object"

console.log(strPrimitive === 'hello'); // true
console.log(strObject === 'hello');    // false (객체와 원시값 비교)
```

```ts
// 래퍼 객체에 속성 할당이 가능 (하지만 권장하지 않음)
const strObject = new String('hello');
strObject.custom = 'value'; // ok (하지만 X)

const strPrimitive = 'hello';
strPrimitive.custom = 'value'; // 에러는 없지만 속성이 저장되지 않음
console.log(strPrimitive.custom); // undefined
```

✅ 래퍼 객체는 `typeof`가 `"object"`이고, 원시 값과 `===` 비교 시 `false`

## 3. TypeScript에서의 타입 구분

TypeScript는 원시 타입과 래퍼 타입을 명확히 구분합니다:

### 잘못된 사용 - 래퍼 타입 사용

```ts
// X - 래퍼 타입 사용
function getLength(s: String): number {
  return s.length;
}

getLength('hello');           // ok
getLength(new String('hi'));  // ok
```

### 올바른 사용 - 원시 타입 사용

```ts
// ok - 원시 타입 사용
function getLength(s: string): number {
  return s.length;
}

getLength('hello');           // ok
getLength(new String('hi'));  // Error: 'String' 형식의 인수는 'string' 형식의 매개변수에 할당될 수 없습니다
```

✅ `string`은 `String`에 할당 가능하지만, `String`은 `string`에 할당 불가

## 4. 할당 가능성의 비대칭

```ts
// string → String: ok
const s1: String = 'hello'; // ok

// String → string: Error
const s2: string = new String('hello');
// Error: 'String' 형식은 'string' 형식에 할당할 수 없습니다.
// 'string'은 기본 개체이지만 'String'은 래퍼 개체입니다.
```

```ts
function toUpper(s: string): string {
  return s.toUpperCase();
}

toUpper('hello');              // ok
toUpper(new String('hello'));  // Error
```

✅ 원시 타입 `string`을 기대하는 곳에 래퍼 객체 `String`을 전달하면 에러

## 5. 흔한 실수 패턴

### 타입 선언에서 대문자 사용

```ts
// X - 잘못된 타입 선언
interface User {
  name: String;   // X
  age: Number;    // X
  active: Boolean; // X
}

// ok - 올바른 타입 선언
interface User {
  name: string;   // ok
  age: number;    // ok
  active: boolean; // ok
}
```

### 제네릭에서의 실수

```ts
// X - 래퍼 타입 사용
function identity<T extends String>(value: T): T {
  return value;
}

// ok - 원시 타입 사용
function identity<T extends string>(value: T): T {
  return value;
}
```

### 배열과 함께 사용할 때

```ts
// X - 래퍼 타입 배열
const names: String[] = ['Alice', 'Bob']; // X

// ok - 원시 타입 배열
const names: string[] = ['Alice', 'Bob']; // ok
```

✅ 인터페이스, 제네릭, 배열 등 모든 곳에서 원시 타입(소문자)을 사용

## 6. new 없이 래퍼 사용하기

`new` 키워드 없이 래퍼 함수를 호출하면 원시 값을 반환합니다:

```ts
// new 없이 호출 - 타입 변환 함수로 사용
const num = Number('42');     // 42 (number 타입)
const str = String(123);      // "123" (string 타입)
const bool = Boolean(1);      // true (boolean 타입)

// new와 함께 호출 - 래퍼 객체 생성 (X)
const numObj = new Number('42');  // Number 객체 (X - 피해야 함)
```

```ts
// 타입 변환에는 래퍼 함수 사용 가능 (new 없이)
const input = '3.14';
const parsed = Number(input); // ok - 원시 number 반환

console.log(typeof parsed);   // "number"
console.log(parsed + 1);      // 4.14
```

✅ `new Number()` 대신 `Number()`를 사용하면 원시 값으로 변환

## 7. Symbol과 BigInt의 특수성

`Symbol`과 `BigInt`는 `new` 키워드로 호출할 수 없습니다:

```ts
// Symbol - new 불가
const sym = Symbol('description');     // ok
const symObj = new Symbol('desc');     // Error: Symbol is not a constructor

// BigInt - new 불가
const big = BigInt(100);               // ok
const big2 = 100n;                     // ok (리터럴 문법)
const bigObj = new BigInt(100);        // Error: BigInt is not a constructor
```

✅ `Symbol`과 `BigInt`는 래퍼 객체를 생성할 수 없으므로 혼동 가능성이 낮음

## 8. 실무에서 자주 발생하는 상황

### API 응답 타입 정의

```ts
// X - 래퍼 타입 사용
interface ApiResponse {
  id: Number;
  message: String;
  success: Boolean;
}

// ok - 원시 타입 사용
interface ApiResponse {
  id: number;
  message: string;
  success: boolean;
}
```

### 함수 매개변수와 반환 타입

```ts
// X - 래퍼 타입
function formatName(first: String, last: String): String {
  return `${first} ${last}`;
}

// ok - 원시 타입
function formatName(first: string, last: string): string {
  return `${first} ${last}`;
}
```

### 유틸리티 타입과 함께 사용

```ts
// X
type StringKeys = keyof String; // 래퍼 객체의 메서드들이 나옴

// ok
type Keys = keyof { name: string; age: number }; // "name" | "age"
```

## 핵심 정리

> 타입스크립트에서 객체 래퍼 타입(`String`, `Number`, `Boolean` 등)은 사용하지 말고, 항상 원시 타입(`string`, `number`, `boolean`)을 사용해야 함
> 원시 타입은 래퍼 타입에 할당 가능하지만, 래퍼 타입은 원시 타입에 할당 불가능

**실무 가이드:**
- 타입 선언 시 항상 소문자(`string`, `number`, `boolean`)를 사용
- 자동완성에서 대문자 버전이 나와도 선택하지 말 것
- `new String()`, `new Number()` 등 래퍼 객체 생성자는 사용하지 말 것
- 타입 변환이 필요하면 `new` 없이 `String()`, `Number()` 함수로 호출
