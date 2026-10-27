# 아이템8. 타입 공간과 값 공간의 심벌 구분하기

> type, interface는 타입 공간에만 존재  
> const, let, function, class, enum등은 값 공간에 존재  
> 일부 심벌은 타입과 값 공간 양쪽에 동시 존재 (class, enum)

## 1. 타입 공간 vs 값 공간
```ts
interface Person {
  name: string;
  age: number;
}

const Person = {
  name: "수찬",
  age: 31,
};
```
여기서 Person은 두 개
```ts
let p: Person; 
// 여기서 Person은 타입 공간의 interface

console.log(Person);
// 여기서 Person은 값 공간의 const
```
✅ 같은 이름이어도 사용 위치에 따라 다르게 해석됨

## 2. 타입으로만 존재하는 것
```ts
type UserId = string;

interface User {
  id: UserId;
  name: string;
}
```
UserId, User는 타입 공간에만 존재함
```ts
console.log(User);
// Error: 'User' only refers to a type, but is being used as a value
```
✅ interface는 컴파일 후 JavaScript에서 사라짐

## 3. 값으로만 존재하는 것
```ts
const userName = "수찬";

function getName() {
  return userName;
}
```
userName, getName은 값 공간의 심벌
```ts
let name: userName;
// Error
```
✅ 값을 타입처럼 바로 사용할 수 없음
값에서 타입을 뽑고 싶으면 typeof를 쓴다

```ts
const user = {
  id: 1,
  name: "수찬",
};

type User = typeof user;
```
결과
```ts
type User = {
  id: number;
  name: string;
}
```

## 4. typeof는 위치에 따라 의미가 다르다
### JavaScript 값 공간의 typeof
```ts
console.log(typeof "hello"); // "string"
console.log(typeof 123);     // "number"
```
런타임에서 값의 타입 문자열을 반환함
### TypeScript 타입 공간의 typeof
```ts
const user = {
  id: 1,
  name: "수찬",
};

type User = typeof user;
```
타입 공간에서 typeof는 값의 타입을 추출한다.

## 5. class는 타입과 값 둘 다 가짐
```ts
class User {
  constructor(public name: string) {}
}

const user = new User("민준");
// 여기서 User는 값, 즉 생성자 함수

let user2: User;
// 여기서 User는 타입, 즉 인스턴스 타입

type UserInstance = User;
type UserConstructor = typeof User;

const createUser = (Ctor: typeof User) => {
  return new Ctor("민준");
};
```
✅ User는 인스턴스 타입, typeof User는 생성자 타입

## 6. enum도 타입과 값 둘 다 가짐
```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

// 타입으로 사용
let dir: Direction = Direction.Up;

// 값으로 사용
console.log(Direction.Up);
console.log(Direction[0]);
```
✅ enum은 JavaScript 코드로도 남기 때문에 값 공간에도 존재

## 7. 타입 단언과 값 비교를 헷갈리지 말기
```ts
interface Square {
  width: number;
}

interface Rectangle {
  width: number;
  height: number;
}

function calculateArea(shape: Square | Rectangle) {
  if (shape instanceof Rectangle) {
    // Error: Rectangle은 interface라 런타임에 존재하지 않음
    return shape.width * shape.height;
  }
}
```
interface Rectangle은 타입 공간에만 있으므로 instanceof에 사용할 수 없음

### 해결
```ts
function calculateArea(shape: Square | Rectangle) {
  if ("height" in shape) {
    return shape.width * shape.height;
  }

  return shape.width * shape.width;
}
```
✅ 런타임에 존재하는 프로퍼티를 기준으로 좁혀야 함 

## 8. 타입 이름과 값 이름을 겹치게 쓰지 않는게 좋음
```ts
interface User {
  name: string;
}

const User = {
  defaultName: "수찬",
};

// 가능은 하지만, 읽는 사람이 헷갈릴 수 있음
// 명확하게 쓰는게 좋음
interface User {
  name: string;
}

const userConfig = {
  defaultName: "수찬",
};
```

## 핵심 정리
> TypeScript 코드를 볼 때 심벌이 타입 위치에 있는지, 값 위치에 있는지 구분해야 함
```ts
let user: User;
// User는 타입

const user = new User();
// User는 값
```
특히 interface, type은 런타임에 사라지므로 instanceof, console.log 조건문 같은 값 공간에서는 사용할 수 없음


