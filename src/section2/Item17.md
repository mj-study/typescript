# 아이템17. 숫자 인덱스 시그니처 지양하기

> JavaScript에서 객체의 키는 항상 문자열(또는 심벌)이며, 배열의 숫자 인덱스도 내부적으로는 문자열로 변환된다.
> TypeScript의 숫자 인덱스 시그니처는 이를 가린 채 오해를 만들기 때문에, 배열 대용으로 쓰지 말고 `Array`, 튜플, `ArrayLike`를 사용한다.

## 1. JavaScript에서 객체의 키는 문자열이다

```ts
const obj = { 1: 'one', 2: 'two' };

Object.keys(obj); // ['1', '2'] — 숫자 키가 문자열로 변환됨
obj[1]; // OK: 'one'
obj['1']; // OK: 'one' — 같은 속성
```

배열도 객체이므로 인덱스는 문자열 키로 동작한다.

```ts
const arr = ['a', 'b', 'c'];

arr[0]; // 'a'
arr['0']; // 'a' — 문자열로 접근해도 동일
Object.keys(arr); // ['0', '1', '2'] — 문자열 배열
```

✅ 런타임에 "숫자 키"는 존재하지 않는다. 항상 문자열로 변환된다.

## 2. TypeScript의 Array 타입 선언

TypeScript는 숫자 키를 허용하되, 타입 체크를 위해 숫자 인덱스 시그니처를 따로 둔다.

```ts
interface Array<T> {
  // ...
  [n: number]: T;
}
```

이 덕분에 잘못된 키 타입을 컴파일 시점에 잡아낸다.

```ts
const xs = [1, 2, 3];

xs[0]; // OK
xs['1']; // Error: 'string' 형식의 식을 인덱스로 사용할 수 없음 (noImplicitAny)
```

✅ 숫자 인덱스 시그니처는 실제 동작이 아니라 타입 체크용 가짜 추상화다.

## 3. 배열을 순회할 때 `for-in`의 함정

`for-in`은 문자열 키를 순회하므로 숫자 연산 시 문제가 생긴다.

```ts
const xs = [1, 2, 3];

for (const i in xs) {
  console.log(typeof i); // 'string'
  console.log(xs[i]); // OK: 하지만 i는 string
  const next = i + 1; // '01', '11', '21' — 문자열 연결 (의도와 다름)
}
```

`for-in` 대신 `for-of`나 `forEach`를 사용한다.

```ts
// OK: 값 순회
for (const x of xs) {
  console.log(x);
}

// OK: 인덱스가 필요하면 forEach
xs.forEach((x, i) => {
  console.log(i + 1, x); // i는 number
});

// OK: 클래식 for문
for (let i = 0; i < xs.length; i++) {
  console.log(xs[i]);
}
```

✅ 배열 순회에는 `for-in`을 쓰지 말고, 값만 필요하면 `for-of`, 인덱스도 필요하면 `forEach`를 사용한다.

## 4. 배열 대신 숫자 인덱스 시그니처를 직접 정의하지 않기

```ts
// X: 배열처럼 쓰려는 의도로 숫자 인덱스 시그니처를 정의
interface MyList {
  [index: number]: string;
}

const list: MyList = { 0: 'a', 1: 'b' };
list.length; // Error: 'length' 속성이 MyList 형식에 없음
list.map((s) => s); // Error: 'map' 속성이 MyList 형식에 없음
```

배열의 메서드(`length`, `map`, `push` 등)를 쓰려면 `Array`를 써야 한다.

```ts
// OK: 일반 배열
const list2: string[] = ['a', 'b'];
list2.length; // OK
list2.map((s) => s.toUpperCase()); // OK
```

✅ 순서 있는 컬렉션이 필요하면 숫자 인덱스 시그니처 대신 `Array`나 튜플을 사용한다.

## 5. 대안 1: 튜플

길이가 고정되고 위치마다 타입이 다르다면 튜플을 사용한다.

```ts
const point: [number, number] = [10, 20];
point[0]; // OK: number
point[2]; // Error: 길이가 '2'인 튜플 형식에 인덱스 '2'가 없음

const entry: [string, number] = ['age', 30]; // OK
```

✅ 길이와 위치별 타입이 고정되어 있으면 튜플이 가장 정확하다.

## 6. 대안 2: ArrayLike

배열처럼 쓰이지만 배열이 아닌 값(예: `arguments`, DOM `NodeList`)의 입력을 받는다면 `ArrayLike`를 사용한다.

```ts
interface ArrayLike<T> {
  readonly length: number;
  readonly [n: number]: T;
}
```

```ts
function checkedAccess<T>(xs: ArrayLike<T>, i: number): T {
  if (i < 0 || i >= xs.length) {
    throw new Error(`배열의 끝을 지나서 ${i}를 접근하려고 했습니다.`);
  }
  return xs[i];
}

const tupleLike: ArrayLike<string> = {
  0: 'A',
  1: 'B',
  length: 2,
}; // OK

checkedAccess([1, 2, 3], 1); // OK: 배열 가능
checkedAccess(tupleLike, 0); // OK: 유사 배열 가능
```

`ArrayLike`는 `length`와 숫자 인덱스만 제공하므로, `map`, `push` 같은 배열 메서드는 쓸 수 없다.

```ts
function bad<T>(xs: ArrayLike<T>) {
  xs.map((x) => x); // Error: 'map' 속성이 ArrayLike<T>에 없음
}
```

✅ 길이와 읽기 접근만 필요한 함수는 `Array<T>` 대신 `ArrayLike<T>`로 받으면 더 많은 입력을 허용한다.

## 7. 배열 vs ArrayLike vs 튜플 비교

| 상황 | 추천 타입 | 이유 |
| --- | --- | --- |
| 일반적인 순서 있는 컬렉션 | `T[]` / `Array<T>` | 모든 배열 메서드 사용 가능 |
| 길이·위치별 타입이 고정 | `[A, B]` (튜플) | 위치별 정확한 타입 |
| 읽기 전용 접근만 필요 | `readonly T[]` / `ArrayLike<T>` | 변경 방지, 유연한 입력 |
| 배열이 아닌 유사 배열 입력 | `ArrayLike<T>` | `NodeList`, `arguments` 등 수용 |
| 키를 숫자로 쓰는 객체 | `Record<number, T>` 지양 → `Map<number, T>` | 의도 명확 |

✅ 용도에 맞는 타입을 고르면 숫자 인덱스 시그니처를 직접 쓸 일이 거의 없다.

## 핵심 정리

> JavaScript의 키는 실제로는 문자열이며, TypeScript의 숫자 인덱스 시그니처는 타입 체크용 추상화일 뿐이다.
> 배열과 유사한 타입이 필요하면 숫자 인덱스 시그니처를 직접 정의하지 말고 `Array`, 튜플, `ArrayLike`를 사용한다.

- 배열 순회에는 `for-in` 대신 `for-of` 또는 `forEach`를 사용한다. `for-in`의 키는 `string`이다.
- 숫자 인덱스 시그니처는 `length`나 `map` 같은 배열 메서드를 제공하지 않는다.
- 배열처럼 접근만 하는 함수는 `ArrayLike<T>`로 받아 입력의 범위를 넓힌다.
