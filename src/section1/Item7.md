# 아이템7. 타입을 값의 집합이라고 생각하기

> TypeScript의 타입은 '가능한 값들의 집합(set)'  
> 타입은 이름이 아니라 [이 타입에 들어올 수 있는 값들의 범위


## 기본 개념
```ts
let a: number;
// a는 number라는 값의 집합에 속함
```

## Union 타입 = 합집합
```ts
type A = 'a' | 'b';

// 'a'의 집합 u 'b'의 집합

let x: A; 

x = 'a'; // ok
x = 'b'; // ok
x = 'c'; // x
```

## Intersection 타입 = 교집합
```ts
type A = { name: string };
type B = { age: number };
type C = A & B;

// A n B 둘 다 만족해야함
const obj: C  = {
  name: '김',
  age: 30,
}
```

## extends = 부분집합 관계
```ts
type A = {name: string};
type B = {name: string, age: number} ;

// B는 A의 부분집합
const b:B = {name: 'kim', age: 30};
const a: A = b; // ok
```

## any, unknown, never
```ts
let x: any; // 모든 값 포함 (최상위 집합)
let z: unknown; // 모든 값 포함은 같지만, 사용하려면 타입 좁혀야함 
let y: never; // 빈 집합 (아무값도 없음)
```
