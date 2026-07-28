# 아이템9. 타입 단언 대신 타입 선언 사용하기

> 타입 단언(as)은 타입 체커를 무시하고, 타입 선언(:)은 타입 체커가 검사함
> 타입 단언은 타입이 틀려도 에러를 내지 않아 위험함
> 가능한 타입 선언(:)을 사용하고, 타입 단언(as)은 정말 필요할 때만 사용

## 1. 타입 선언 vs 타입 단언

### 타입 선언 (권장)
```ts
interface Person {
  name: string;
  age: number;
}

const alice: Person = {
  name: "Alice",
  age: 30
}; // ok

const bob: Person = {
  name: "Bob"
}; // Error: Property 'age' is missing
```
✅ 타입 선언은 타입 체커가 값을 검사하여 타입 안정성 보장

### 타입 단언 (위험)
```ts
interface Person {
  name: string;
  age: number;
}

const alice = {
  name: "Alice",
  age: 30
} as Person; // ok

const bob = {
  name: "Bob"
} as Person; // ok (but dangerous!)
```
✅ 타입 단언은 타입 체커를 무시하므로 잘못된 타입도 통과시킴

## 2. 화살표 함수에서의 타입 선언

### 잘못된 방법
```ts
interface Person {
  name: string;
}

const people = ['alice', 'bob', 'charlie'].map(name => ({
  name
} as Person)); // X - 타입 단언 사용
```
문제: name 속성 외에 다른 필수 속성이 추가되어도 에러가 발생하지 않음

### 올바른 방법
```ts
interface Person {
  name: string;
}

const people = ['alice', 'bob', 'charlie'].map((name): Person => ({
  name
})); // ok - 반환 타입 명시

// 또는
const people = ['alice', 'bob', 'charlie'].map((name) => {
  const person: Person = { name };
  return person;
}); // ok
```
✅ 반환 타입을 명시하거나 변수에 타입 선언을 사용하면 타입 체커가 검사

## 3. 타입 단언이 필요한 경우

### DOM 엘리먼트 타입 좁히기
```ts
// TypeScript는 DOM 타입을 HTMLElement로만 인식
const button = document.querySelector('.btn');
// button의 타입: Element | null

// 더 구체적인 타입으로 단언 필요
const button = document.querySelector('.btn') as HTMLButtonElement;
// button의 타입: HTMLButtonElement
```

### 개발자가 타입 체커보다 타입을 더 잘 아는 경우
```ts
interface User {
  id: number;
  name: string;
}

// API 응답을 받았고, 구조를 정확히 알고 있는 경우
const response = await fetch('/api/user');
const user = await response.json() as User;
```
⚠️ 주의: 이 경우에도 런타임 검증을 추가하는 것이 안전함

## 4. null이 아님을 단언하는 접미사 !

```ts
const element = document.getElementById('root');
// element의 타입: HTMLElement | null

// null이 아님을 확신하는 경우
const element = document.getElementById('root')!;
// element의 타입: HTMLElement

// 더 안전한 방법
const element = document.getElementById('root');
if (element) {
  // element의 타입: HTMLElement (타입 좁히기)
  element.textContent = 'Hello';
}
```
✅ ! 단언 대신 null 체크로 타입을 좁히는 것이 더 안전

## 5. 타입 단언의 위험성 예시

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}

// 잘못된 데이터를 타입 단언으로 우회
const product = {
  id: 1,
  name: "Book"
  // price가 없음!
} as Product;

console.log(product.price.toFixed(2));
// 런타임 에러: Cannot read property 'toFixed' of undefined
```

```ts
// 올바른 방법
const product: Product = {
  id: 1,
  name: "Book"
  // Error: Property 'price' is missing
};
```
✅ 타입 선언을 사용하면 컴파일 시점에 에러를 발견

## 6. 타입 단언 체이닝

```ts
interface Person {
  name: string;
}

const data = {} as Person; // X - 너무 관대함

// TypeScript는 서브타입이나 슈퍼타입으로만 단언 가능
const data = {} as unknown as Person; // 가능하지만 최대한 피해야 함
```
⚠️ unknown을 거쳐서 단언하면 모든 타입 체크를 우회할 수 있어 매우 위험

## 핵심 정리

> 타입 선언(: Type)은 값이 타입을 만족하는지 검사하고, 타입 단언(as Type)은 타입 체커에게 "내가 더 잘 알아"라고 말하는 것
> 타입 단언은 타입 체커를 무시하므로 런타임 에러의 원인이 될 수 있음

**실무 가이드:**
- 기본적으로 타입 선언(:)을 사용
- 타입 단언(as)은 DOM 조작 등 정말 필요한 경우에만 제한적으로 사용
- ! 단언 대신 null 체크로 타입 좁히기 활용
- as unknown as Type 같은 이중 단언은 절대 사용하지 말 것
