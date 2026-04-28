# Item5. any 타입 지양하기

> any는 TypeScript의 타입 검사를 사실상 꺼버리는 타입임.  
> any를 쓰면 TypeScript가 해당 값에 대해 거의 아무 검사도 하지 않음

```ts
let value: any = 'hello';

value.toUpperCase(); // 가능
value.toFixed(); // 컴파일가능, 런타임 에러 가능
value.foo.bar(); // 컴파일가능, 런타임 에러 가능
```

## any 사용 장점
### 1. 빠르게 개발 가능
```ts
function parseData(data: any) {
  return data.user.name;
}
```
타입을 정확하게 작성하지 않아도 바로 코드 작성 가능

### 2. 외부 라이브러리/레거시 코드 대응 쉬움
```ts
const lecagyResult: any = window.someOldLibrary.getData();
```
타입 정의가 없는 코드와 연결할 때 편함 (현재 흥국 프로젝트가 ts로 가면 그럴듯)

### 3. 타입 에러를 임시로 우회 가능
```ts
const result = unknownValue as any;
```
마이그레이션 초기에는 현실적인 선택일 수 있음

## any 사용 단점
### 1. 타입 안정성 사라짐
```ts
function getUserName(user: any) {
  return user.name.toUpperCase();
}
getUserName(null); // 컴파일 통과, 런타임 에러
```

### 2. 자동완성, 리팩터링 품질 저하
```ts
const user:any = {
  name: 'Kim',
  age: 30,
}
user.naem; // 오타인데도 에러 안남
```

### 3. any 전염
```ts
function getData(): any {
  return {id: 1, name: 'Kim'};
}

const user = getData();
user.notExistMethod(); // 통과
```
any를 반환하면 그 값을 받는 쪽도 타입 검사를 잃음

## any를 지향했을 때 장점
### 1. 컴파일 단계에서 오류 발견 가능
```ts
type User = {
  name: string;
  age: number;
}
function getUserName(user: User){
  return user.name.toUpperCase();
}
getUserName(null); // 에러
```
런타임 전에 문제를 잡을 수 있음

### 2. 자동완성/리팩터링이 좋아짐
```ts
const user:User = {
  name: 'Kim',
  age: 30,
}

user.name; // 자동완성 가능
user.naem; // 오타 에러 발생
```

### 3. 코드 의도가 명확해짐
```ts
function assginSeat(seatId: number, memberId: number) {
  
}
```
