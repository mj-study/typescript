# Item04. 구조적 타이핑에 익숙해지기
| 타입의 이름(name)이 아니라 모양(shape, 구조)이 같으면 같은 타입으로 취급하는 방식

### 예시
```typescript
type User = {
  name: string;
  age: number;
}

const person = {
  name: 'Han',
  age: 30,
};

const user: User = person; // 가능
```
- person 타입 이름이 User가 아니어도 내부구조가 string, number로 같기 때문

## 왜 TypeScript가 이렇게 설계됐는지?
JavaScript는 원래 객체를 자유롭게 다룸
```ts
function printName(obj) {
  console.log(obj.name);
}
```
여기서 중요한 건 클래스 이름이 아니라 name 프로퍼티가 있는지?
=> 그래서 TypeScript도 JS 스타일에 맞춰 구조 중심으로 설계됨

## 실무에서 자주 보이는 예시
### 1. 함수인자
```ts
type Member = {
  id: number;
  name: string;
}

function greet(member: Member) {
  console.log(member.name);
}

greet({
  id: 1,
  name: 'kim'
})
```
구조만 맞으면 전달 가능

### 2. 더 많은 속성이 있어도 가능
```ts
type User = {
  name: string;
}
const obj = {
  name: 'Han',
  age: 30,
}
const user: User = obj; // 가능
```
User의 name만 있으면 됨

## 주의할 점: 객체 리터럴은 더 엄격(Excess Property Check)
아래는 가능
```ts
const obj = {name: 'Han', age: 30};
const user: Uesr = obj;
```

직접 넣는건 불가능
```ts
const user: User = {
  name: 'Han',
  age: 30, // 에러 
}
```

> Tip) TypeScript에서 에러 볼 때
> - 이 타입 이름이 왜 안맞는지 보다 -> [필드 구조]가 맞는지
