# 아이템19. 다른 타입에는 다른 변수 사용하기

> JavaScript에서는 한 변수에 다른 타입의 값을 재할당할 수 있지만, TypeScript에서 변수의 타입은 보통 바뀌지 않는다.
> 타입을 억지로 넓히기보다, 서로 다른 타입의 값에는 서로 다른 변수를 사용한다.

## 1. 변수의 값은 바뀌어도 타입은 바뀌지 않는다

```ts
// JavaScript에서는 가능
let id = '12-34-56';
id = 123456;
```

```ts
let id = '12-34-56'; // 타입: string
id = 123456; // Error: 'number' 형식은 'string' 형식에 할당할 수 없음
```

✅ TypeScript에서 변수의 타입은 선언 시점에 정해지며, 값의 범위가 좁아지는 것(narrowing)은 가능해도 다른 타입으로 바뀌지는 않는다.

## 2. 유니온 타입으로 해결하면 생기는 문제

```ts
// X: 유니온으로 넓히면 사용할 때마다 타입 체크가 필요
let id: string | number = '12-34-56';
id = 123456; // OK

id.toUpperCase(); // Error: 'number' 형식에 'toUpperCase' 속성이 없음
```

```ts
// 매번 좁혀야 해서 코드가 복잡해짐
if (typeof id === 'string') {
  id.toUpperCase(); // OK
}
```

✅ 유니온 타입은 동작하지만, 쓸 때마다 타입을 좁혀야 해서 오히려 다루기 어렵다.

## 3. 별도의 변수를 사용하기

```ts
// OK: 서로 다른 타입에는 서로 다른 변수
const id = '12-34-56'; // string
const serial = 123456; // number
```

다른 변수를 쓰면 얻는 이점:
- 서로 관련 없는 두 값을 분리할 수 있다
- 변수명을 더 구체적으로 지을 수 있다
- 타입 추론이 좋아지고 타입 구문이 필요 없어진다
- 타입이 좀 더 간결해진다 (`string | number` 대신 `string`, `number`)
- `let` 대신 `const`로 선언할 수 있다

✅ 변수를 재사용하는 대신 값마다 변수를 새로 만들면 더 명확하고 안전하다.

## 4. 변수 재사용의 대표 예: 중간 결과 덮어쓰기

```ts
// X: 같은 변수에 여러 타입의 값을 차례로 대입
function fetchUser(raw: string) {
  let result: string | object | number = raw;
  result = JSON.parse(raw); // 객체
  result = (result as { id: number }).id; // 숫자
  return result;
}
```

```ts
// OK: 단계마다 변수를 분리
function fetchUser2(raw: string) {
  const parsed = JSON.parse(raw) as { id: number };
  const id = parsed.id;
  return id;
}
```

✅ 한 변수가 여러 의미를 가지지 않도록 단계별로 이름을 붙인다.

## 5. 섀도잉(shadowing)과의 차이

다른 스코프에서 같은 이름을 쓰는 섀도잉은 별개의 변수이므로 타입이 달라도 오류가 없지만, 혼동을 일으킬 수 있다.

```ts
const id = '12-34-56'; // string

{
  const id = 123456; // OK: 별개의 변수 (섀도잉)
  console.log(id); // number
}

console.log(id); // string
```

⚠️ 섀도잉은 문법적으로 허용되지만, 같은 이름이 다른 타입을 가리켜 읽는 사람을 혼란스럽게 하므로 피하는 것이 좋다. ESLint의 `no-shadow` 규칙으로 감지할 수 있다.

✅ 섀도잉은 타입 오류가 아니지만, 다른 변수명을 쓰는 편이 더 명확하다.

## 6. 변수를 재사용해도 되는 경우

타입이 바뀌는 것이 아니라 같은 타입 안에서 값만 바뀌는 경우는 문제가 없다.

```ts
let count = 0; // number
count = count + 1; // OK: 같은 타입
count += 1; // OK

let message = 'hello'; // string
message = message.toUpperCase(); // OK: 같은 타입
```

✅ 타입이 같다면 `let`으로 값을 바꿔도 되지만, 가능하면 `const`로 불변성을 유지한다.

## 핵심 정리

> 변수의 값은 바뀔 수 있지만 타입은 보통 바뀌지 않는다.
> 서로 다른 타입의 값에는 유니온으로 넓히는 대신 별도의 변수를 사용한다.

- 한 변수에 여러 타입을 담으려 하면 유니온 타입이 되고, 사용할 때마다 타입을 좁혀야 한다.
- 변수를 분리하면 변수명이 구체적이고 `const`를 쓸 수 있어 타입 추론이 좋아진다.
- 섀도잉은 별개의 변수이지만 혼동을 부르므로 피하고, `no-shadow` 같은 린트 규칙을 활용한다.
