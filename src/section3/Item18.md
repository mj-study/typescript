# 아이템18. 추론 가능한 타입을 사용해 장황한 코드 방지하기

> TypeScript는 변수와 함수의 타입을 스스로 추론(type inference)할 수 있으므로, 추론 가능한 곳에는 타입 구문을 생략한다.
> 다만 객체 리터럴, 함수 반환 타입, API 경계에서는 타입을 명시하는 것이 오류를 더 잘 잡아준다.

## 1. 불필요한 타입 구문은 코드를 어지럽힌다

```ts
// X: 추론 가능한 타입을 중복해서 작성
let x: number = 12;
let name: string = 'kim';
let isDone: boolean = false;
```

```ts
// OK: 타입 추론에 맡김
let x = 12; // number
let name = 'kim'; // string
let isDone = false; // boolean
```

`const`로 선언하면 더 좁은 리터럴 타입으로 추론된다.

```ts
const y = 12; // 타입: 12
let z = 12; // 타입: number
```

✅ 원시값 변수에 타입 구문을 달면 오히려 노이즈만 늘어난다.

## 2. 추론은 더 정확한 타입을 만들어준다

```ts
// X: 타입 구문이 오히려 정보를 잃음
const axis1: string = 'x'; // 타입: string

// OK: 추론하면 리터럴 타입 유지
const axis2 = 'x'; // 타입: 'x'
```

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id;
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

```ts
// OK: 구조 분해 할당 + 타입 추론
function logProduct(product: Product) {
  const { id, name, price } = product;
  console.log(id, name, price);
}
```

✅ 타입 추론은 `Product`가 변경되면 자동으로 따라가므로 유지보수에 유리하다.

## 3. 비구조화 할당과 매개변수 기본값

비구조화 할당문은 모든 지역 변수의 타입이 추론되도록 한다.

```ts
// X: 기본값이 있으면 타입 구문을 생략할 수 있음
function parseNumber(str: string, base: number = 10) {
  return parseInt(str, base);
}

// OK: 기본값에서 number로 추론
function parseNumber2(str: string, base = 10) {
  return parseInt(str, base);
}
```

✅ 매개변수에 기본값이 있으면 타입 구문을 생략한다.

## 4. 콜백 함수의 매개변수 타입

문맥(contextual typing)에서 콜백 매개변수의 타입이 추론되므로 생략한다.

```ts
// X: 불필요한 타입 구문
const nums = [1, 2, 3];
nums.map((n: number): number => n * 2);

// OK: 문맥에서 추론
nums.map((n) => n * 2);
```

```ts
// 라이브러리의 콜백 타입도 마찬가지
app.get('/health', (request, response) => {
  response.send('OK'); // request, response 타입이 자동 추론됨
});
```

✅ 콜백에는 타입을 달지 않고 문맥 추론에 의존한다.

## 5. 타입 구문을 명시해야 하는 경우

### 객체 리터럴: 잉여 속성 체크

```ts
interface Person {
  name: string;
  born: { where: string; when: string };
  died?: { where: string; when: string };
}

// X: 타입 구문이 없으면 오류 위치가 사용처로 밀려남
const alanTuring = {
  name: 'Alan Turing',
  born: { where: 'London', when: 'June 23, 1912' },
  died: { where: 'Wilmslow', when: 'June 7, 1954' },
  site: 'wikipedia', // 오류 없음
};

// OK: 타입 구문을 달면 정의 지점에서 오류 발생
const ada: Person = {
  name: 'Ada Lovelace',
  born: { where: 'London', when: 'December 10, 1815' },
  site: 'wikipedia', // Error: 'site'는 Person 형식에 없음
};
```

✅ 객체 리터럴에 타입 구문을 달면 잉여 속성 체크와 자동완성이 정의 시점에 동작한다.

### 함수 반환 타입

```ts
const cache: { [ticker: string]: number } = {};

// X: 반환 타입이 없으면 실수가 호출 쪽에서 드러남
function getQuote(ticker: string) {
  if (ticker in cache) {
    return cache[ticker]; // number
  }
  return fetch(`https://quotes.example.com/?q=${ticker}`)
    .then((response) => response.json())
    .then((quote) => {
      cache[ticker] = quote;
      return quote;
    }); // Promise<any> — 반환 타입이 number | Promise<any>로 추론됨
}
```

```ts
// OK: 반환 타입을 명시해 구현 오류를 함수 안에서 잡는다
function getQuote2(ticker: string): Promise<number> {
  if (ticker in cache) {
    return cache[ticker]; // Error: 'number'를 'Promise<number>'에 할당할 수 없음
  }
  return Promise.resolve(cache[ticker]); // OK
}
```

반환 타입을 명시해야 하는 이유:
- 구현과 호출의 오류를 구분해서 보여준다
- 함수에 대한 명세(contract)를 구현보다 먼저 작성하게 된다 (테스트 주도 개발과 유사)
- 반환 타입이 `any`나 유니온으로 흘러가는 것을 막는다
- 명명된 타입을 사용하면 가독성이 좋아진다

```ts
interface Vector2D {
  x: number;
  y: number;
}

// OK: 반환 타입을 명명된 타입으로 지정
function add(a: Vector2D, b: Vector2D): Vector2D {
  return { x: a.x + b.x, y: a.y + b.y };
}
```

✅ 함수 반환 타입은 명시하면 구현 오류와 호출 오류를 분리할 수 있다.

## 6. 정리: 타입 구문 사용 가이드

| 상황 | 타입 구문 | 이유 |
| --- | --- | --- |
| 원시값 변수 (`let x = 12`) | 생략 | 추론으로 충분 |
| 기본값이 있는 매개변수 | 생략 | 기본값에서 추론 |
| 콜백의 매개변수 | 생략 | 문맥 추론 |
| 비구조화 할당 | 생략 | 원본 타입에서 추론 |
| 객체 리터럴 (정의 시점 검증이 필요할 때) | 명시 | 잉여 속성 체크 |
| 함수의 반환 타입 | 명시 | 명세 작성, 오류 위치 개선 |
| 공개 API / 라이브러리 경계 | 명시 | 계약 안정성 |

⚠️ ESLint의 `no-inferrable-types` 규칙은 원시값의 불필요한 타입 구문을 잡아준다.

## 핵심 정리

> 코드의 모든 곳에 타입을 쓰지 말고, 추론 가능한 곳에서는 생략하여 코드를 간결하게 유지한다.
> 객체 리터럴과 함수 반환 타입은 명시해서 오류를 정의 지점에서 잡는다.

- 원시값, 기본값 매개변수, 콜백 매개변수, 비구조화 할당에는 타입 구문을 생략한다.
- 함수 반환 타입을 명시하면 구현 오류를 함수 내부에서 잡고, `any` 전파를 막을 수 있다.
- 추론이 가능한 곳과 필요한 곳을 구분하는 것이 핵심이다.
