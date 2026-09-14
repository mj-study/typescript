# 아이템12. 함수 표현식에 타입 적용하기

> JavaScript/TypeScript에서 함수를 작성하는 두 가지 방법: 문장(statement)과 표현식(expression)
> 함수 표현식을 사용하면 함수 타입을 재사용할 수 있어 더 안전하고 간결함
> 매개변수와 반환 타입을 한 번에 선언할 수 있음

## 1. 함수 문장 vs 함수 표현식

### 함수 문장 (Function Statement)

```ts
function rollDice1(sides: number): number {
  return Math.floor(Math.random() * sides) + 1;
}
```

각 매개변수와 반환 타입을 개별적으로 지정해야 함

### 함수 표현식 (Function Expression)

```ts
type DiceRollFn = (sides: number) => number;

const rollDice2: DiceRollFn = (sides) => {
  return Math.floor(Math.random() * sides) + 1;
};
```

✅ 함수 표현식은 함수 전체에 타입을 적용하여 타입 재사용 가능

## 2. 함수 타입 재사용의 장점

여러 함수가 같은 시그니처를 가질 때 함수 타입을 재사용하면 중복을 줄일 수 있습니다.

### Bad: 매번 타입 반복

```ts
function add(a: number, b: number): number {
  return a + b;
}

function subtract(a: number, b: number): number {
  return a - b;
}

function multiply(a: number, b: number): number {
  return a * b;
}
```

### Good: 함수 타입 재사용

```ts
type BinaryFn = (a: number, b: number) => number;

const add: BinaryFn = (a, b) => a + b;
const subtract: BinaryFn = (a, b) => a - b;
const multiply: BinaryFn = (a, b) => a * b;
```

✅ 타입을 한 곳에서 정의하고 재사용하여 일관성 유지

## 3. 라이브러리 함수 타입 활용

라이브러리의 공통 함수 시그니처를 그대로 사용할 수 있습니다.

```ts
// fetch의 타입 시그니처 활용
const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }
  return response;
};
```

`typeof fetch`를 사용하면:
- `input`과 `init` 매개변수 타입 자동 추론
- 반환 타입 `Promise<Response>` 자동 적용
- fetch API 변경 시 자동으로 타입 동기화

✅ 라이브러리 함수의 타입을 재사용하여 타입 안정성 확보

## 4. 다른 함수의 시그니처 참조

```ts
declare function fetch(
  input: RequestInfo,
  init?: RequestInit
): Promise<Response>;

// typeof를 사용한 시그니처 복사
const customFetch: typeof fetch = async (input, init) => {
  // fetch와 동일한 시그니처를 가짐
  console.log(`Fetching ${input}`);
  return fetch(input, init);
};
```

✅ 함수의 시그니처를 정확히 복사하여 타입 오류 방지

## 5. 매개변수 타입 추론

함수 표현식에 타입을 지정하면 매개변수 타입이 자동으로 추론됩니다.

```ts
type MouseEventHandler = (event: MouseEvent) => void;

// event의 타입이 자동으로 MouseEvent로 추론됨
const handleClick: MouseEventHandler = (event) => {
  console.log(event.clientX, event.clientY); // OK
};
```

### 함수 문장에서는 타입 추론 불가

```ts
function handleClick2(event) {
  // Error: Parameter 'event' implicitly has an 'any' type
  console.log(event.clientX, event.clientY);
}
```

✅ 함수 표현식은 매개변수 타입을 자동으로 추론하여 코드 간결화

## 6. 실전 예제: API 핸들러

```ts
type APIHandler = (
  req: Request,
  res: Response
) => Promise<void> | void;

// 여러 핸들러가 같은 시그니처 공유
const getUserHandler: APIHandler = async (req, res) => {
  const user = await fetchUser(req.params.id);
  res.json(user);
};

const createUserHandler: APIHandler = async (req, res) => {
  const user = await createUser(req.body);
  res.status(201).json(user);
};

const deleteUserHandler: APIHandler = async (req, res) => {
  await deleteUser(req.params.id);
  res.status(204).send();
};
```

✅ 모든 핸들러가 일관된 시그니처를 가져 유지보수 용이

## 7. 함수 전체 타입 vs 개별 타입

```ts
// 개별 타입 지정 - 반복적이고 실수 가능
const add1 = (a: number, b: number): number => a + b;
const add2 = (a: number, b: number): number => a + b;
const add3 = (a: number, b: numbre): number => a + b; // 오타!
```

```ts
// 함수 전체 타입 - 일관성 보장
type BinaryOp = (a: number, b: number) => number;
const add1: BinaryOp = (a, b) => a + b;
const add2: BinaryOp = (a, b) => a + b;
const add3: BinaryOp = (a, b) => a + b; // 일관된 타입
```

✅ 타입 재사용으로 오타와 불일치 방지

## 8. 고차 함수에서의 활용

```ts
type UnaryFn<T, R> = (arg: T) => R;

function map<T, R>(
  array: T[],
  fn: UnaryFn<T, R>
): R[] {
  return array.map(fn);
}

// fn의 타입이 명확히 정의됨
const lengths = map(['a', 'bb', 'ccc'], (str) => str.length);
// str은 자동으로 string 타입으로 추론
```

✅ 고차 함수에서 콜백 타입을 명확하게 표현

## 핵심 정리

> **함수 표현식**을 사용하면 함수 타입을 분리하여 재사용 가능
> 매개변수 타입이 자동으로 추론되어 코드가 간결해짐
> 라이브러리 함수의 `typeof`를 활용하면 타입 동기화 자동화

함수 문장보다는 **함수 표현식**을 사용하는 것이 TypeScript에서는 더 안전합니다. 함수 전체에 타입을 적용할 수 있어 재사용성이 높고, 같은 타입 시그니처를 공유하는 여러 함수를 작성할 때 일관성을 보장할 수 있습니다.

특히 **다른 함수의 시그니처를 참조**할 때 (`typeof fetch` 등) 타입을 수동으로 복사하는 대신 자동으로 동기화되어 유지보수가 쉬워집니다.
