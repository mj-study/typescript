# 아이템15. 타입 연산과 제네릭 사용으로 반복 줄이기

> DRY(Don't Repeat Yourself) 원칙은 코드뿐 아니라 타입에도 적용된다.
> 타입에 이름을 붙이고, 확장·매핑·인덱싱·제네릭으로 중복을 제거하면 타입 간 일관성이 유지된다.

## 1. 반복을 줄이는 기본: 타입에 이름 붙이기

```ts
// X: 같은 시그니처가 반복됨
function distance(a: { x: number; y: number }, b: { x: number; y: number }) {
  return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}
```

```ts
// OK: 타입에 이름을 붙여 재사용
interface Point2D {
  x: number;
  y: number;
}

function distance(a: Point2D, b: Point2D) {
  return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}
```

함수 시그니처가 반복될 때는 함수 타입에 이름을 붙인다.

```ts
type HTTPFunction = (url: string, opts: Options) => Promise<Response>;

const get: HTTPFunction = (url, opts) => { /* ... */ };
const post: HTTPFunction = (url, opts) => { /* ... */ };
```

✅ 상수를 변수로 추출하듯, 반복되는 타입은 이름을 붙여 추출한다.

## 2. 인터페이스 확장으로 중복 제거

```ts
// X: 공통 필드가 반복됨
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate {
  firstName: string;
  lastName: string;
  birth: Date;
}
```

```ts
// OK: extends로 확장
interface PersonWithBirthDate extends Person {
  birth: Date;
}

// OK: 교차 타입(&)으로 확장
type PersonWithBirthDate2 = Person & { birth: Date };
```

✅ 한 인터페이스가 다른 인터페이스를 확장하면 공통 필드를 한 곳에서만 관리할 수 있다.

## 3. 인덱싱으로 부분 타입 추출

전체 상태 타입의 일부만 필요할 때, 새로 정의하지 말고 인덱싱한다.

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

// X: State의 일부를 다시 정의 (State가 바뀌면 불일치 발생)
interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
```

```ts
// OK: Pick으로 필요한 속성만 추출
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

### 인덱싱으로 하위 타입 접근

```ts
type PageTitle = State['pageTitle']; // string
```

✅ `Pick`은 `State`를 단일 원천(single source of truth)으로 삼아 동기화를 보장한다.

## 4. 매핑된 타입(Mapped Type)으로 필드 반복 제거

```ts
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}

// X: 모든 필드를 옵셔널로 다시 정의
interface OptionsUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}
```

```ts
// OK: 매핑된 타입으로 배열 for 루프처럼 필드를 순회
type OptionsUpdate = { [K in keyof Options]?: Options[K] };

// OK: 표준 라이브러리 Partial 사용
type OptionsUpdate2 = Partial<Options>;
```

`keyof`는 타입의 키를 유니온으로 반환한다.

```ts
type OptionsKeys = keyof Options; // 'width' | 'height' | 'color' | 'label'
```

### Pick 직접 구현해보기

```ts
type MyPick<T, K extends keyof T> = { [P in K]: T[P] };
```

✅ 매핑된 타입은 배열의 필드 순회를 타입 공간에서 수행한다.

## 5. typeof로 값에서 타입 추출

값의 형태에 맞춰 타입을 정의해야 한다면 `typeof`로 추출한다.

```ts
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: '#00FF00',
  label: 'VGA',
};

// X: 값과 타입이 따로 정의됨
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
```

```ts
// OK: 값으로부터 타입 추출
type Options = typeof INIT_OPTIONS;
```

⚠️ 타입 정의를 먼저 하고 값이 그 타입에 할당 가능하게 선언하는 쪽이 일반적으로 더 좋은 설계다. `typeof`는 값이 이미 있고 그것이 단일 원천일 때만 쓴다.

### ReturnType으로 함수 반환 타입 추출

```ts
function getUserInfo(userId: string) {
  return { userId, name: 'kim', age: 20 };
}

type UserInfo = ReturnType<typeof getUserInfo>;
// { userId: string; name: string; age: number }
```

✅ `ReturnType`은 값 공간의 함수 `getUserInfo`에 `typeof`를 적용해 타입 공간으로 가져와야 한다.

## 6. 태그된 유니온에서 Union 추출

```ts
interface SaveAction {
  type: 'save';
}
interface LoadAction {
  type: 'load';
}
type Action = SaveAction | LoadAction;

// X: 유니온을 직접 반복
type ActionType = 'save' | 'load';

// OK: 인덱싱으로 추출
type ActionType2 = Action['type']; // 'save' | 'load'
```

✅ 새로운 Action이 추가되면 `ActionType`도 자동으로 갱신된다.

## 7. 제네릭 타입은 타입을 위한 함수

제네릭은 타입 공간에서의 함수다. 타입을 매개변수로 받아 새로운 타입을 만든다.

```ts
// 값 공간의 함수: 중복 제거
function logAndReturn(value: number) { return value; }

// 타입 공간의 함수: 제네릭
interface Name {
  first: string;
  last: string;
}
type DancingDuo<T extends Name> = [T, T];

const couple1: DancingDuo<Name> = [
  { first: 'Fred', last: 'Astaire' },
  { first: 'Ginger', last: 'Rogers' },
]; // OK

const couple2: DancingDuo<{ first: string }> = [
  // Error: Type '{ first: string }' does not satisfy the constraint 'Name'
  { first: 'Sonny' },
  { first: 'Cher' },
];
```

### extends로 제네릭 매개변수 제한

`extends`는 "~의 부분 집합"을 의미한다 (아이템7 참고).

```ts
// OK: K는 T의 키 중 하나여야 함
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'kim', age: 20 };
getValue(user, 'name'); // OK: string
getValue(user, 'email'); // Error: 'email'은 'name' | 'age'에 할당할 수 없음
```

✅ 제네릭 타입에서 `extends`는 매개변수를 제한해 잘못된 사용을 컴파일 타임에 막는다.

## 8. 실전 예제: 표준 라이브러리 유틸리티 타입

| 유틸리티 | 설명 | 예시 |
| --- | --- | --- |
| `Pick<T, K>` | 지정한 속성만 선택 | `Pick<State, 'userId'>` |
| `Omit<T, K>` | 지정한 속성 제외 | `Omit<State, 'pageContents'>` |
| `Partial<T>` | 모든 속성을 옵셔널로 | `Partial<Options>` |
| `Required<T>` | 모든 속성을 필수로 | `Required<Options>` |
| `Readonly<T>` | 모든 속성을 읽기 전용으로 | `Readonly<State>` |
| `ReturnType<F>` | 함수 반환 타입 | `ReturnType<typeof fn>` |

```ts
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Omit<User, 'password'>; // OK
type UserPatch = Partial<Omit<User, 'id'>>; // OK: id를 제외한 수정용 타입
```

✅ 직접 구현하기 전에 표준 라이브러리에 같은 기능이 있는지 먼저 확인한다.

## 핵심 정리

> 타입에도 DRY 원칙을 적용해 반복을 제거하라.
> `extends`, `Pick`, `Partial`, `keyof`, `typeof`, `ReturnType` 등으로 타입 간 매핑을 만들고, 제네릭은 `extends`로 제한하라.

- 반복된 타입은 이름을 붙이거나, 확장·인덱싱·매핑된 타입으로 줄인다.
- 제네릭은 타입을 위한 함수이므로, `extends`로 매개변수를 제한해 안전하게 사용한다.
- 단일 원천(single source of truth)을 정하고 나머지 타입은 거기서 파생시킨다.
- 다만 DRY를 과도하게 적용하면 우연히 같은 모양인 타입까지 묶여 오히려 결합도가 높아질 수 있으니, 의미상 같은 것만 합친다.
