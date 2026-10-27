# 아이템16. 인덱스 시그니처보다 정확한 타입 사용하기

> 인덱스 시그니처(index signature)는 키의 이름을 모를 때만 사용하고, 가능하면 더 정확한 타입을 사용한다.
> 인터페이스, `Record`, 매핑된 타입, `Map` 등이 인덱스 시그니처보다 안전하고 자동완성도 지원한다.

## 1. 인덱스 시그니처란?

```ts
type Rocket = { [property: string]: string };

const rocket: Rocket = {
  name: 'Falcon 9',
  variant: 'v1.0',
  thrust: '4,940 kN',
}; // OK
```

세 가지 타입 정보가 들어 있다: 키의 타입(`string`), 값의 타입(`string`), 키의 이름(`property`, 문서화 용도).

### 인덱스 시그니처의 단점

```ts
// 1. 잘못된 키도 허용됨 (name 대신 Name)
const bad: Rocket = {
  Name: 'Falcon 9', // OK: 오타를 잡지 못함
};

// 2. 특정 키가 반드시 필요하다는 것을 표현할 수 없음
const empty: Rocket = {}; // OK: 빈 객체도 가능

// 3. 키마다 다른 값 타입을 가질 수 없음
const mixed: Rocket = {
  name: 'Falcon 9',
  thrust: 4940, // Error: 'number' 형식은 'string' 형식에 할당할 수 없음
};

// 4. 자동완성 불가
rocket.na; // 'name'이 자동완성되지 않음
```

✅ 인덱스 시그니처는 너무 광범위하여 오타, 누락, 자동완성 모두 지원하지 못한다.

## 2. 대안 1: 인터페이스 (키가 고정된 경우)

```ts
interface Rocket {
  name: string;
  variant: string;
  thrust_kN: number;
}

const falconHeavy: Rocket = {
  name: 'Falcon Heavy',
  variant: 'v1',
  thrust_kN: 15200,
}; // OK

const wrong: Rocket = {
  Name: 'Falcon', // Error: 'Name'은 Rocket 형식에 없음
  variant: 'v1',
  thrust_kN: 1,
};
```

✅ 필드가 정해져 있다면 인터페이스로 키마다 다른 타입을 정확히 표현한다.

## 3. 대안 2: 데이터가 동적이라 일부 필드가 없을 수 있는 경우

CSV 파싱처럼 열 이름을 런타임에야 알 수 있는 경우에는 인덱스 시그니처가 적절하다.

```ts
function parseCSV(input: string): { [columnName: string]: string }[] {
  const lines = input.split('\n');
  const [header, ...rows] = lines;
  const headerColumns = header.split(',');
  return rows.map((rowStr) => {
    const row: { [columnName: string]: string } = {};
    rowStr.split(',').forEach((cell, i) => {
      row[headerColumns[i]] = cell;
    });
    return row;
  });
}
```

열 이름을 알고 있는 경우에는 호출하는 쪽에서 타입을 단언하거나 제네릭으로 좁힌다.

```ts
interface ProductRow {
  productId: string;
  name: string;
  price: string;
}

declare let csvData: string;
const products = parseCSV(csvData) as unknown as ProductRow[]; // OK
```

⚠️ 단언은 런타임에 열이 실제로 존재하는지 보장하지 않으므로, 안전하게 하려면 값이 `undefined`일 수 있다고 가정한다.

```ts
declare let csvData: string;
const rows = parseCSV(csvData);
const columnB = rows[0]?.['B']; // 타입: string (noUncheckedIndexedAccess를 켜면 string | undefined)
```

✅ 키를 런타임에야 알 수 있을 때만 인덱스 시그니처를 쓴다.

## 4. 대안 3: Record와 매핑된 타입

인덱스 시그니처보다 키의 범위를 좁게 지정한다.

### Record

```ts
type Vec3D = Record<'x' | 'y' | 'z', number>;

const v: Vec3D = { x: 1, y: 2, z: 3 }; // OK
const w: Vec3D = { x: 1, y: 2 }; // Error: 'z' 속성이 없음
```

### 매핑된 타입 (Mapped Type)

```ts
type ABC = { [k in 'a' | 'b' | 'c']: k extends 'b' ? string : number };
// { a: number; b: string; c: number }
```

✅ 키가 유니온으로 한정될 때는 `Record`나 매핑된 타입으로 키를 정확히 제한한다.

## 5. 대안 4: 선택적 필드와 유니온

```ts
// 일부 키만 허용: 선택적 필드
interface Row1 {
  a: number;
  b?: number;
  c?: number;
  d?: number;
}

// 더 정확한: 가능한 조합을 유니온으로 표현
type Row2 =
  | { a: number }
  | { a: number; b: number }
  | { a: number; b: number; c: number }
  | { a: number; b: number; c: number; d: number };
```

✅ 값이 존재하는 조합이 제한적이라면 유니온이 가장 정확하다.

## 6. 대안 5: Map

인덱스 시그니처는 객체이므로 프로토타입의 속성과 충돌할 수 있다. 키-값 쌍을 다룬다면 `Map`이 더 안전하다.

```ts
const counts: { [key: string]: number } = {};
counts['constructor']; // 타입은 number지만 실제로는 Object의 생성자 함수
```

```ts
const counts2 = new Map<string, number>();
counts2.set('apple', 3); // OK
counts2.get('constructor'); // OK: undefined
```

✅ 임의의 문자열 키 사전에는 `Map`이 프로토타입 오염 없이 더 안전하다.

## 7. 인덱스 시그니처와 `noUncheckedIndexedAccess`

```ts
const dict: { [key: string]: string } = { a: 'A' };
const value = dict['b']; // 타입: string (실제로는 undefined)
value.toUpperCase(); // 런타임 에러 위험
```

`../../tsconfig.json`에서 `noUncheckedIndexedAccess`를 켜면 인덱스 접근 결과가 `string | undefined`가 되어 안전해진다.

```ts
const value2 = dict['b']; // string | undefined
value2.toUpperCase(); // Error: 'value2'가 undefined일 수 있음
value2?.toUpperCase(); // OK
```

✅ 인덱스 시그니처를 써야 한다면 `noUncheckedIndexedAccess`로 undefined 가능성을 타입에 반영한다.

## 핵심 정리

> 인덱스 시그니처는 동적 데이터(키를 런타임에야 아는 경우)에만 사용한다.
> 키가 정해져 있다면 인터페이스, 유니온 키라면 `Record`·매핑된 타입, 키-값 사전이라면 `Map`을 사용한다.

- 인덱스 시그니처는 오타·누락·자동완성을 지원하지 못한다.
- 정확한 타입(인터페이스, `Record`, 유니온)이 오류를 컴파일 시점에 잡아준다.
- 불가피하게 쓴다면 `noUncheckedIndexedAccess`로 `undefined` 가능성을 표현한다.
