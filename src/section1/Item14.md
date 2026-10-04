# 아이템14. 변경 관련 오류 방지를 위해 readonly 사용하기

> `readonly` 키워드를 사용하면 의도치 않은 객체 변경을 방지할 수 있음
> 함수가 매개변수를 변경하지 않는다는 것을 명시적으로 표현하여 타입 안정성 향상
> `readonly`는 얕은(shallow) 불변성만 보장하므로 깊은 불변성이 필요하면 추가 조치 필요

## 1. readonly의 기본 개념

### 변경 가능한 배열 vs 읽기 전용 배열

```ts
function arraySum(arr: number[]): number {
  let sum = 0;
  for (const num of arr) {
    sum += num;
  }
  arr.push(999); // 배열 변경 - 의도치 않은 부작용!
  return sum;
}

const numbers = [1, 2, 3, 4, 5];
const total = arraySum(numbers);
console.log(numbers); // [1, 2, 3, 4, 5, 999] - 원본 배열이 변경됨!
```

### readonly로 보호

```ts
function arraySum(arr: readonly number[]): number {
  let sum = 0;
  for (const num of arr) {
    sum += num;
  }
  arr.push(999); // Error: Property 'push' does not exist on type 'readonly number[]'
  return sum;
}
```

✅ `readonly`를 사용하면 배열이나 객체를 변경하는 메서드 호출을 컴파일 타임에 차단

## 2. readonly 배열의 특징

### 읽기 전용 배열은 변경 메서드 사용 불가

```ts
const arr: readonly number[] = [1, 2, 3];

arr[0] = 10; // Error: Index signature in type 'readonly number[]' only permits reading
arr.push(4); // Error: Property 'push' does not exist
arr.pop(); // Error: Property 'pop' does not exist
arr.splice(1, 1); // Error: Property 'splice' does not exist
```

### 읽기 메서드는 정상 작동

```ts
const arr: readonly number[] = [1, 2, 3];

console.log(arr.length); // OK: 3
console.log(arr[0]); // OK: 1
const doubled = arr.map((x) => x * 2); // OK: [2, 4, 6]
const filtered = arr.filter((x) => x > 1); // OK: [2, 3]
```

✅ `readonly` 배열은 읽기 작업만 허용하고 변경 작업은 차단

## 3. 일반 배열과 readonly 배열의 호환성

### readonly 배열은 일반 배열에 할당 가능

```ts
const readonlyArr: readonly number[] = [1, 2, 3];
const normalArr: number[] = readonlyArr; // Error: 'readonly number[]'는 'number[]'에 할당 불가
```

TypeScript는 일반 배열로 변환을 허용하지 않습니다. 이는 안전성을 보장하기 위함입니다.

### 일반 배열은 readonly 배열에 할당 가능

```ts
const normalArr: number[] = [1, 2, 3];
const readonlyArr: readonly number[] = normalArr; // OK
```

✅ 일반 배열 → `readonly` 배열은 안전하므로 허용됨

## 4. readonly를 사용한 함수 매개변수

### Bad: 매개변수 변경 가능

```ts
function printTriangles(n: number, points: [number, number][]) {
  for (let i = 0; i < n; i++) {
    const point = points.pop(); // 원본 배열 변경!
    console.log(point);
  }
}

const coords: [number, number][] = [
  [0, 0],
  [1, 1],
  [2, 2],
];
printTriangles(2, coords);
console.log(coords); // [[0, 0]] - 원본이 변경됨!
```

### Good: readonly로 보호

```ts
function printTriangles(n: number, points: readonly [number, number][]) {
  for (let i = 0; i < n; i++) {
    const point = points[i]; // 읽기만 수행
    console.log(point);
  }
  // points.pop(); // Error: Property 'pop' does not exist
}

const coords: [number, number][] = [
  [0, 0],
  [1, 1],
  [2, 2],
];
printTriangles(2, coords); // OK
console.log(coords); // [[0, 0], [1, 1], [2, 2]] - 원본 유지
```

✅ 함수가 매개변수를 변경하지 않는다면 `readonly`로 명시

## 5. readonly 객체 속성

### 객체의 특정 속성을 읽기 전용으로

```ts
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
  retries: number; // 변경 가능
}

const config: Config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
};

config.apiUrl = 'https://new-api.com'; // Error: Cannot assign to 'apiUrl' because it is a read-only property
config.timeout = 10000; // Error: Cannot assign to 'timeout' because it is a read-only property
config.retries = 5; // OK
```

✅ 객체의 특정 속성만 `readonly`로 지정 가능

## 6. Readonly 유틸리티 타입

### Readonly<T>로 모든 속성을 읽기 전용으로

```ts
interface User {
  id: string;
  name: string;
  email: string;
}

type ReadonlyUser = Readonly<User>;
// 동일:
// type ReadonlyUser = {
//   readonly id: string;
//   readonly name: string;
//   readonly email: string;
// }

const user: ReadonlyUser = {
  id: '1',
  name: 'John',
  email: 'john@example.com',
};

user.name = 'Jane'; // Error: Cannot assign to 'name' because it is a read-only property
```

✅ `Readonly<T>` 유틸리티 타입으로 모든 속성을 한 번에 읽기 전용으로 변환

## 7. 얕은(Shallow) 불변성의 한계

### readonly는 1단계 깊이만 보호

```ts
interface Outer {
  readonly inner: {
    x: number;
  };
}

const obj: Outer = {
  inner: { x: 0 },
};

obj.inner = { x: 1 }; // Error: Cannot assign to 'inner'
obj.inner.x = 1; // OK - 중첩된 속성은 변경 가능!
```

### 깊은 불변성이 필요하면 중첩 적용

```ts
interface Outer {
  readonly inner: {
    readonly x: number;
  };
}

const obj: Outer = {
  inner: { x: 0 },
};

obj.inner = { x: 1 }; // Error: Cannot assign to 'inner'
obj.inner.x = 1; // Error: Cannot assign to 'x'
```

✅ `readonly`는 얕은 불변성만 제공하므로 중첩 객체는 각 단계마다 `readonly` 필요

## 8. DeepReadonly 타입 구현

### 재귀적으로 모든 속성을 읽기 전용으로

```ts
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object
    ? DeepReadonly<T[K]>
    : T[K];
};

interface NestedData {
  level1: {
    level2: {
      value: number;
    };
  };
}

type DeepReadonlyData = DeepReadonly<NestedData>;

const data: DeepReadonlyData = {
  level1: {
    level2: {
      value: 42,
    },
  },
};

data.level1.level2.value = 100; // Error: Cannot assign to 'value'
```

✅ 깊은 불변성이 필요하면 `DeepReadonly` 같은 재귀적 타입 활용

## 9. readonly와 const의 차이

### const: 변수 재할당 방지

```ts
const arr = [1, 2, 3];
arr = [4, 5, 6]; // Error: Cannot assign to 'arr' because it is a constant
arr.push(4); // OK - 배열 내용은 변경 가능
```

### readonly: 객체/배열 내용 변경 방지

```ts
const arr: readonly number[] = [1, 2, 3];
arr = [4, 5, 6]; // Error: Cannot assign to 'arr' because it is a constant
arr.push(4); // Error: Property 'push' does not exist
```

✅ `const`는 재할당 방지, `readonly`는 내용 변경 방지

## 10. 실전 예제: 불변 상태 관리

### Bad: 상태를 직접 변경

```ts
interface State {
  user: {
    name: string;
    age: number;
  };
  settings: {
    theme: string;
  };
}

function updateUserAge(state: State, newAge: number): State {
  state.user.age = newAge; // 원본 변경!
  return state;
}
```

### Good: readonly로 불변성 보장

```ts
interface State {
  readonly user: {
    readonly name: string;
    readonly age: number;
  };
  readonly settings: {
    readonly theme: string;
  };
}

function updateUserAge(state: State, newAge: number): State {
  // state.user.age = newAge; // Error: Cannot assign to 'age'

  // 새로운 객체 반환 (불변 업데이트)
  return {
    ...state,
    user: {
      ...state.user,
      age: newAge,
    },
  };
}
```

✅ `readonly`를 사용하면 실수로 원본을 변경하는 것을 방지

## 11. 라이브러리에서의 readonly 활용

### React에서의 Props

```ts
interface ButtonProps {
  readonly label: string;
  readonly onClick: () => void;
  readonly disabled?: boolean;
}

function Button(props: ButtonProps) {
  // props.label = 'New Label'; // Error: Cannot assign to 'label'
  return <button onClick={props.onClick}>{props.label}</button>;
}
```

### Immutable 업데이트 패턴

```ts
interface TodoItem {
  readonly id: string;
  readonly text: string;
  readonly completed: boolean;
}

function toggleTodo(todo: TodoItem): TodoItem {
  // todo.completed = !todo.completed; // Error

  return {
    ...todo,
    completed: !todo.completed,
  };
}
```

✅ 라이브러리와 프레임워크에서 불변성 패턴을 강제하는 데 유용

## 12. readonly와 타입 추론

### 타입 추론 시 readonly 손실

```ts
function getConfig() {
  return {
    apiUrl: 'https://api.example.com',
    timeout: 5000,
  };
}

const config = getConfig();
config.apiUrl = 'new-url'; // OK - readonly가 아님
```

### 명시적으로 readonly 반환

```ts
function getConfig(): {
  readonly apiUrl: string;
  readonly timeout: number;
} {
  return {
    apiUrl: 'https://api.example.com',
    timeout: 5000,
  };
}

const config = getConfig();
config.apiUrl = 'new-url'; // Error: Cannot assign to 'apiUrl'
```

### as const 활용

```ts
function getConfig() {
  return {
    apiUrl: 'https://api.example.com',
    timeout: 5000,
  } as const;
}

const config = getConfig();
config.apiUrl = 'new-url'; // Error: Cannot assign to 'apiUrl'
```

✅ 반환 타입에 `readonly`를 명시하거나 `as const` 사용

## 13. 성능 고려사항

`readonly`는 컴파일 타임에만 존재하며, 런타임 성능에는 영향을 주지 않습니다.

```ts
// TypeScript (컴파일 전)
function sum(arr: readonly number[]): number {
  return arr.reduce((a, b) => a + b, 0);
}

// JavaScript (컴파일 후)
function sum(arr) {
  return arr.reduce((a, b) => a + b, 0);
}
```

✅ `readonly`는 타입 체크만 수행하고 런타임 코드에는 영향 없음

## 핵심 정리

> **`readonly`**를 사용하면 의도치 않은 변경을 컴파일 타임에 방지
> 함수 매개변수, 객체 속성, 배열 등에 적용하여 불변성 보장
> 얕은 불변성만 제공하므로 중첩 구조에서는 각 단계마다 `readonly` 필요

**실무 권장사항:**
- 함수가 매개변수를 변경하지 않는다면 `readonly` 사용
- 상태 관리 시 불변성을 강제하기 위해 `readonly` 활용
- `Readonly<T>` 유틸리티 타입으로 간편하게 모든 속성을 읽기 전용으로 변환
- 깊은 불변성이 필요하면 `DeepReadonly` 같은 재귀 타입 구현
- `const`는 재할당 방지, `readonly`는 내용 변경 방지라는 차이점 이해
- `as const`를 활용하여 리터럴 타입과 함께 불변성 보장
