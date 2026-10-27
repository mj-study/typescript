# 아이템13. 타입과 인터페이스의 차이점 이해하기

> TypeScript에서 명명된 타입을 정의하는 두 가지 방법: `type`과 `interface`
> 대부분의 경우 둘 중 무엇을 사용해도 되지만, 차이점을 이해하고 일관되게 사용해야 함
> 복잡한 타입은 `type`, 확장 가능한 객체 타입은 `interface` 사용 권장

## 1. 공통점: 비슷한 용도로 사용 가능

### 객체 타입 정의

```ts
// type 사용
type TState = {
  name: string;
  capital: string;
};

// interface 사용
interface IState {
  name: string;
  capital: string;
}
```

둘 다 동일하게 사용 가능:

```ts
const wyoming: TState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
}; // OK

const newYork: IState = {
  name: 'New York',
  capital: 'Albany',
}; // OK
```

✅ 객체를 정의할 때 `type`과 `interface` 모두 사용 가능

### 인덱스 시그니처 사용

```ts
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}
```

✅ 인덱스 시그니처도 동일하게 지원

### 함수 타입 정의

```ts
type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}

const toStrT: TFn = (x) => String(x); // OK
const toStrI: IFn = (x) => String(x); // OK
```

✅ 함수 타입도 둘 다 정의 가능하지만, `type`이 더 간결함

### 제네릭 사용

```ts
type TPair<T> = {
  first: T;
  second: T;
};

interface IPair<T> {
  first: T;
  second: T;
}
```

✅ 제네릭도 동일하게 지원

### 타입 확장

```ts
// type은 &(intersection) 사용
type TStateWithPop = TState & { population: number };

// interface는 extends 사용
interface IStateWithPop extends IState {
  population: number;
}
```

✅ 둘 다 확장 가능하지만 문법이 다름

### 클래스에서 구현 (implements)

```ts
class StateT implements TState {
  name: string = '';
  capital: string = '';
}

class StateI implements IState {
  name: string = '';
  capital: string = '';
}
```

✅ 클래스는 `type`과 `interface` 모두 구현 가능

## 2. 차이점 1: Union 타입

`type`은 Union 타입을 정의할 수 있지만, `interface`는 불가능합니다.

```ts
type AorB = 'a' | 'b';
type Input = string | number;
type NamedVariable = (Input | AorB) & { name: string };
```

`interface`로는 Union 타입 표현 불가:

```ts
// interface로는 불가능
interface AorB {
  // Error: Union 타입 표현 불가
}
```

✅ **Union 타입**이 필요하면 반드시 `type` 사용

## 3. 차이점 2: 튜플과 배열 타입

튜플은 `type`으로 더 간결하게 표현 가능합니다.

```ts
// type으로 튜플 정의 (간결함)
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];
```

`interface`로도 가능하지만 복잡함:

```ts
interface Tuple {
  0: number;
  1: number;
  length: 2;
}

const t: Tuple = [10, 20]; // OK
```

하지만 `concat` 같은 메서드 사용 불가:

```ts
const t: Tuple = [10, 20];
t.concat([30, 40]); // Error: Property 'concat' does not exist
```

✅ **튜플과 배열 타입**은 `type` 사용 권장

## 4. 차이점 3: interface의 선언 병합 (Declaration Merging)

`interface`는 같은 이름으로 여러 번 선언하면 자동으로 병합됩니다.

```ts
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

// 자동으로 병합됨
const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000, // OK
};
```

`type`은 중복 선언 불가:

```ts
type TState = {
  name: string;
  capital: string;
};

type TState = {
  // Error: Duplicate identifier 'TState'
  population: number;
};
```

### 선언 병합의 활용: 타입 보강 (Augmentation)

라이브러리 타입을 확장할 때 유용합니다:

```ts
// 기존 라이브러리 타입
interface Array<T> {
  length: number;
  // ...
}

// 사용자 정의 메서드 추가
interface Array<T> {
  customMethod(): void;
}

// 이제 모든 배열에서 사용 가능
[1, 2, 3].customMethod(); // OK
```

✅ **선언 병합**이 필요하면 `interface` 사용

## 5. 차이점 4: Computed Property

`type`은 computed property를 사용할 수 있지만, `interface`는 제한적입니다.

```ts
type Keys = 'firstName' | 'lastName';

// type으로 computed property 사용
type Person = {
  [K in Keys]: string;
};

const person: Person = {
  firstName: 'John',
  lastName: 'Doe',
}; // OK
```

`interface`로는 mapped type 불가:

```ts
// interface로는 불가능
interface Person {
  [K in Keys]: string; // Error
}
```

✅ **Mapped type**이 필요하면 `type` 사용

## 6. 언제 무엇을 사용할까?

### `type` 사용이 권장되는 경우

1. **Union 타입**이 필요할 때
   ```ts
   type Status = 'pending' | 'success' | 'error';
   ```

2. **튜플 타입**을 정의할 때
   ```ts
   type Point = [number, number];
   ```

3. **Mapped type**이나 조건부 타입을 사용할 때
   ```ts
   type Readonly<T> = {
     readonly [K in keyof T]: T[K];
   };
   ```

4. **복잡한 타입 조합**이 필요할 때
   ```ts
   type ComplexType = A & B | C;
   ```

### `interface` 사용이 권장되는 경우

1. **API의 공개 타입**을 정의할 때 (향후 확장 가능성)
   ```ts
   interface User {
     id: string;
     name: string;
   }
   ```

2. **라이브러리 타입을 보강**할 때
   ```ts
   interface Window {
     myCustomProperty: string;
   }
   ```

3. **클래스와 함께 사용**할 때 (객체지향 패턴)
   ```ts
   interface Shape {
     area(): number;
   }

   class Circle implements Shape {
     area() {
       return Math.PI * this.radius ** 2;
     }
   }
   ```

✅ 프로젝트에서는 둘 중 하나를 일관되게 선택하여 사용

## 7. 실전 예제: API 응답 타입

### type 사용 (Union과 복잡한 타입)

```ts
type ApiResponse<T> =
  | { status: 'success'; data: T }
  | { status: 'error'; error: string }
  | { status: 'loading' };

type UserResponse = ApiResponse<{ id: string; name: string }>;

const response: UserResponse = {
  status: 'success',
  data: { id: '1', name: 'John' },
}; // OK
```

### interface 사용 (확장 가능한 객체)

```ts
interface User {
  id: string;
  name: string;
}

interface Admin extends User {
  permissions: string[];
}

// 나중에 확장 가능
interface Admin {
  level: number; // 선언 병합
}

const admin: Admin = {
  id: '1',
  name: 'Admin',
  permissions: ['read', 'write'],
  level: 5,
}; // OK
```

✅ 용도에 맞게 `type`과 `interface`를 적절히 선택

## 8. 스타일 가이드 추천

### 일관성이 가장 중요

```ts
// Good: 일관된 스타일
interface User {
  id: string;
  name: string;
}

interface Product {
  id: string;
  title: string;
}

// Bad: 혼용
interface User {
  id: string;
  name: string;
}

type Product = {
  id: string;
  title: string;
};
```

### TypeScript 팀 권장사항

- **기본적으로 `interface` 사용**
- Union, 튜플 등 `interface`로 불가능한 경우 `type` 사용
- 프로젝트 전체에서 일관성 유지

✅ 팀 내에서 컨벤션을 정하고 일관되게 사용

## 핵심 정리

> **`type`과 `interface`**는 대부분 비슷하지만 중요한 차이점이 있음
> `type`은 Union, 튜플, Mapped type 등 더 다양한 타입 표현 가능
> `interface`는 선언 병합과 확장에 유리하여 API 설계에 적합

**실무 권장사항:**
- 복잡한 타입(Union, Intersection, Mapped)은 `type` 사용
- 확장 가능한 객체 타입, API 공개 타입은 `interface` 사용
- 프로젝트 전체에서 일관된 컨벤션 유지
- TypeScript 팀 권장: 기본은 `interface`, 필요시 `type` 사용
