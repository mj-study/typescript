# 아이템11. 타입 체크와 잉여 속성 체크 구분해서 사용하기

> TypeScript는 두 가지 다른 체크 방식을 사용함
> **구조적 타이핑**: 할당 가능성을 느슨하게 검사
> **잉여 속성 체크**: 객체 리터럴을 엄격하게 검사

## 1. 구조적 타이핑 (Structural Typing)

TypeScript는 기본적으로 구조적 타이핑을 사용합니다.

```ts
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // Error: 잉여 속성 체크 발동
};
```

하지만 중간 변수를 사용하면 통과합니다:

```ts
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present',
};

const r: Room = obj; // OK
```

✅ 구조적 타이핑은 타입에 필요한 속성만 있으면 할당 가능

## 2. 잉여 속성 체크 (Excess Property Check)

객체 리터럴을 **직접** 할당할 때만 잉여 속성 체크가 발동합니다.

```ts
interface Options {
  title: string;
  darkMode?: boolean;
}

const o1: Options = {
  title: 'Ski Free',
  darkmode: true, // Error: 'darkmode'는 Options에 없음
};
```

### 잉여 속성 체크를 피하는 방법들

#### 방법 1: 중간 변수 사용

```ts
const intermediate = {
  title: 'Ski Free',
  darkmode: true,
};

const o2: Options = intermediate; // OK
```

#### 방법 2: 타입 단언 사용 (권장하지 않음)

```ts
const o3 = {
  title: 'Ski Free',
  darkmode: true,
} as Options; // OK, 하지만 오타 검출 못함
```

#### 방법 3: 인덱스 시그니처 사용

```ts
interface Options {
  title: string;
  darkMode?: boolean;
  [otherOptions: string]: unknown;
}

const o4: Options = {
  title: 'Ski Free',
  darkmode: true, // OK
};
```

✅ 잉여 속성 체크는 오타나 불필요한 속성을 잡아내는 유용한 기능

## 3. 함수 매개변수에서의 차이

### 직접 전달 (잉여 속성 체크 O)

```ts
function setDarkMode(options: Options) {
  // ...
}

setDarkMode({
  title: 'Ski Free',
  darkmode: true, // Error
});
```

### 변수로 전달 (구조적 타이핑만)

```ts
const opts = {
  title: 'Ski Free',
  darkmode: true,
};

setDarkMode(opts); // OK
```

✅ 함수에 객체 리터럴을 직접 전달하면 잉여 속성 체크가 작동

## 4. 약한 타입 (Weak Type)

모든 속성이 선택적인 타입을 "약한 타입"이라고 합니다.

```ts
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}

const opts = { logScale: true }; // 'logscale'이 아닌 'logScale'
const o: LineChartOptions = opts; // Error
```

약한 타입은 중간 변수를 사용해도 최소 하나의 공통 속성이 있어야 합니다.

```ts
const validOpts = { logscale: true };
const o2: LineChartOptions = validOpts; // OK
```

✅ 약한 타입은 최소한의 타입 안정성을 제공하기 위해 더 엄격함

## 5. 실전 예제: API 응답 처리

```ts
interface UserResponse {
  id: number;
  name: string;
  email?: string;
}

// Bad: 잉여 속성 체크를 우회하여 타입 안정성 상실
const response = {
  id: 1,
  name: 'John',
  emal: 'john@example.com', // 오타
};
const user: UserResponse = response; // OK (구조적 타이핑)
```

```ts
// Good: 객체 리터럴로 직접 할당하여 오타 발견
const user: UserResponse = {
  id: 1,
  name: 'John',
  emal: 'john@example.com', // Error: 잉여 속성 체크가 오타 발견
};
```

✅ 가능하면 객체 리터럴을 직접 할당하여 잉여 속성 체크의 이점 활용

## 핵심 정리

> **구조적 타이핑**은 할당 시 필요한 속성만 있으면 허용
> **잉여 속성 체크**는 객체 리터럴 직접 할당 시 추가 속성을 오류로 감지
> 두 체크 방식의 차이를 이해하고 각각의 장점을 활용해야 함

잉여 속성 체크는 타입 오류를 잡는 효과적인 방법이지만, 구조적 타이핑의 일반 규칙과는 다릅니다. 중간 변수를 사용하거나 타입 단언을 쓰면 잉여 속성 체크를 우회할 수 있지만, 이는 오타나 불필요한 속성을 놓칠 수 있습니다.

**선택적 속성만 있는 약한 타입**은 구조적으로 관련이 없는 타입 간의 할당을 방지하기 위해 더 엄격한 체크를 수행합니다.
