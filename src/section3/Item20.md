# 아이템20. 변수의 타입이 결정되는 원리 이해하기

> TypeScript는 변수를 선언할 때 초기값을 보고 "가능한 값들의 집합" 중 적절한 타입을 선택한다. 이 과정을 타입 넓히기(widening)라고 한다.
> `const`, 타입 구문, 문맥, `as const`로 추론 범위를 직접 조절할 수 있다.

## 1. 타입 넓히기(Widening)

런타임에 변수는 하나의 값을 가지지만, 정적 분석 시점에는 "가능한 값들의 집합", 즉 타입을 가진다.

```ts
interface Vector3 {
  x: number;
  y: number;
  z: number;
}

function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}

let x = 'x'; // 타입: string (넓혀짐)
let vec = { x: 10, y: 20, z: 30 };

getComponent(vec, x); // Error: 'string' 형식은 '"x" | "y" | "z"' 형식에 할당할 수 없음
```

`let x = 'x'`는 `'x'`라는 리터럴 대신 `string`으로 추론되었기 때문에 오류가 발생한다.

```ts
const x2 = 'x'; // 타입: 'x'
getComponent(vec, x2); // OK
```

✅ 타입스크립트는 오류를 최소화하기 위해 리터럴을 넓히되, 너무 엄격하지 않도록 균형을 맞춘다.

## 2. const vs let 의 추론 차이

```ts
const a = 1; // 타입: 1
let b = 1; // 타입: number

const c = 'hello'; // 타입: 'hello'
let d = 'hello'; // 타입: string
```

`const`는 재할당이 불가능하므로 리터럴 타입으로, `let`은 재할당될 수 있어 넓은 타입으로 추론된다.

✅ 값이 바뀌지 않는다면 `const`를 사용하면 더 정확한 타입이 추론된다.

## 3. 객체와 배열의 넓히기

`const`로 선언해도 객체의 속성은 변경될 수 있으므로 넓혀진다.

```ts
const v = {
  x: 1,
}; // 타입: { x: number }

v.x = 3; // OK
v.x = '3'; // Error: 'string' 형식은 'number' 형식에 할당할 수 없음
v.y = 4; // Error: 'y' 속성이 없음
```

```ts
const arr = [1, 2, 3]; // 타입: number[]
const mixed = [1, 'a']; // 타입: (string | number)[]
```

넓히기의 규칙(속성 단위):

| 값 | 추론 타입 |
| --- | --- |
| `const x = 'x'` | `'x'` |
| `let x = 'x'` | `string` |
| `{ x: 'x' }`의 `x` | `string` |
| `['x']`의 요소 | `string` |

✅ 객체·배열은 `const`로 선언해도 내부 값이 변경될 수 있어서 속성 타입이 넓혀진다.

## 4. 타입 추론의 강도를 제어하는 방법

### 4-1. 명시적 타입 구문

```ts
const v1: { x: 1 | 3 | 5 } = {
  x: 1,
}; // OK: 타입이 { x: 1 | 3 | 5 }
```

### 4-2. 타입 체커에 추가 문맥 제공

함수의 매개변수로 값을 전달하면 문맥에서 추론된다.

```ts
function getComponent2(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}

getComponent2(vec, 'x'); // OK: 문맥에서 'x'가 리터럴로 추론됨
```

### 4-3. `as const`

`as const`는 값에 최대한 좁은 타입을 부여한다. 이 구문은 `const`와 다르며, 타입 공간에서의 `const` 단언이다.

```ts
const v2 = {
  x: 1,
  y: 2,
} as const;
// 타입: { readonly x: 1; readonly y: 2 }

const v3 = [1, 2, 3] as const;
// 타입: readonly [1, 2, 3]
```

```ts
const ax = { x: 'x' } as const;
getComponent(vec, ax.x); // OK: ax.x의 타입이 'x'
```

✅ 타입을 좁히고 싶으면 타입 구문, 문맥, `as const` 세 가지 도구를 활용한다.

## 5. 실전 예제: 과도하게 좁아진 타입의 문제

`as const`를 쓰면 `readonly`가 되어 나중에 값을 바꾸는 코드에서 오류가 날 수 있다.

```ts
const config = {
  retries: 3,
  mode: 'fast',
} as const;

config.retries = 5; // Error: 읽기 전용 속성이므로 'retries'에 할당할 수 없음
```

```ts
// OK: 변경이 필요한 값은 타입 구문으로 필요한 만큼만 좁힘
interface Config {
  retries: number;
  mode: 'fast' | 'slow';
}

const config2: Config = {
  retries: 3,
  mode: 'fast',
};

config2.retries = 5; // OK
```

✅ `as const`는 불변 값에만 쓰고, 변경되는 값에는 타입 구문으로 허용 범위를 직접 정한다.

## 6. 넓히기가 일어나지 않는 경우

`null`과 `undefined`는 `strictNullChecks` 설정에 따라 달라진다.

```ts
// strictNullChecks 켜짐
const x = null; // 타입: null
let y = null; // 타입: null (strict에서는 넓혀지지 않음)

// strictNullChecks 꺼짐
// let y = null → any 로 넓혀짐
```

✅ `strictNullChecks`를 켜면 `null`/`undefined`가 `any`로 넓혀지는 일이 없어 추론이 예측 가능해진다.

## 핵심 정리

> 변수는 초기값을 보고 타입이 정해지며, `let`·객체 속성은 넓혀지고 `const` 변수는 리터럴 타입을 유지한다.
> 타입 구문, 문맥, `as const`로 추론 범위를 직접 제어한다.

- 타입스크립트가 어떻게 넓히는지 이해하면 "string은 '"x" | "y"'에 할당할 수 없다" 같은 오류의 원인을 바로 알 수 있다.
- 불변 값에는 `as const`, 변경되는 값에는 정확한 타입 구문을 사용한다.
- `strictNullChecks`를 켜 두면 `null`과 `undefined`의 추론이 안정적이다.
