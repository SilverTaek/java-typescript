# 요약

## 아이템6 편집기를 사용하여 타입 시스템 탐색하기

타입스크립트를 설치하면 두 가지를 실행할 수 있습니다.

- 타입스크립트 컴파일러(tsc)
- 타입스크립트 서버(tsserver)

### tsserver?

[tsserver](https://github.com/microsoft/TypeScript/wiki/Standalone-Server-%28tsserver%29)는 언어 서비스를 제공하는 Stand alone 서버입니다. JSON 형식으로 요청/응답을 처리하며, vscode에서는 built in extension으로 제공하고 있어서 별다른 셋팅이 없어도 사용 가능합니다.
코드를 확인하고 싶다면 [Github](https://github.com/microsoft/TypeScript)나 node_modules/typescript/lib/tsserver.js 확인해 보면 됩니다.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660318136252/Hi6EO2Z0v.png align="left")

### 언어서비스 이용하기

위의 tsserver에서 제공하는 서비스를 적극적으로 이용하여 타입 시스템이 어떻게 동작하는지, 컴파일러가 타입을 어떻게 추론하고 있는지를 확인할 수 있습니다. 이를 통해서 타입에 익숙해져야 합니다.

## 아이템7 타입이 값들의 집합이라고 생각하기

타입스크립트의 타입을 '할당 가능한 값들의 집합'이라고 가정하면 이해하기가 쉽습니다. 벤다이어그램을 떠올리면서 집합의 개념이라고 생각하면 됩니다.

- 리터럴(유닛) 타입: 한 가지 값만 포함하는 타입
- 유니온 타입: 2개 이상의 값을 포함하는 타입

```typescript
type AB = "A" | "B"; // 유니온 타입
const c: AB = "C"; // 오류; 'C'라는 리터럴 타입은 AB타입(유니온)의 부분집합이 아님
```

집합의 관점에서, 타입 체커의 역할은 하나의 집합이 다른 집합의 부분 집합인지 검사하는 것입니다.

---

값에 대한 `&`과 `keyof`의 동작에 대한 이해를 하기 위한 예시입니다.

```typescript
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}

type PL1 = keyof (Person & Lifespan); // 'name' | 'birth' | 'death'
type PL2 = keyof (Person | Lifespan); // never
type PL3 = keyof Person | keyof Lifespan; // 'name' | 'birth' | 'death'
type PL4 = keyof Person & keyof Lifespan; // never
```

---

타입에서는 extends를 ~의 부분집합으로 이해하면 이해가 쉽고, 이를 통해서 타입을 한정시킨다는 것을 이해하는 것이 중요합니다.

```typescript
function sortBy<K extends keyof T, T>(value: T[], key:K): T[] { ... }
// K를 T타입의 키의 부분집합으로 제한함
```

## 아이템8 타입 공간과 값 공간의 심벌 구분하기

타입스크립트의 심벌은 타입 공간이나 값 공간 중의 한 곳에 존재하며, 심벌의 이름이 같더라도 다른 공간에 존재하는 것을 의미할 수 있습니다.

```typescript
interface Cyl {
  r: number;
  h: number;
}

const Cyl = (r: number, h: number) => ({ r, h });

function cal(s: unknown) {
  if (s instanceof Cyl) {
    s.r = 1; //Property 'r' does not exist on type '{}'.(2339)
  }
}
```

`instanceof`는 자바스크립트 런타임 연산자이고 따라서 값에 대한 연산자입니다. 위 예시에서의 `instanceof Cyl`에서의 `Cyl`은 함수를 참수를 참조했고, 따라서 `r`은 없는 필드입니다.

---

대상이 `값`이냐 `타입`이냐에 따라 `typeof`도 이해가 필요합니다.

```typescript
type T1 = typeof p; // Person
type T2 = typeof email; // (p: Person, subject: string, body: string) => Response

const v1 = typeof p; // object
const v2 = typeof email; // function
```

`타입`의 관점에서 `typeof`는 타입스크립트의 타입을 의미하고, `값`의 관점에서 `typeof`는 자바스크립트 런타임의 `typeof`로 동작하게 되어 6개 타입(string, number, boolean, undefined, obejct, funtion) 중 하나가 결과로 나오게 됩니다. (현재는 8개: symbol, bigint)

이 외의 두 공간 사이에서 다른 의미를 가지는 패턴

- 값으로 쓰이는 `this`는 자바스크립트의 `this` 키워드이고, 타입으로 쓰이는 `this`는 `다형성(polymorphic) this`라고 불리는 `this`의 타입스크립트 타입입니다.
- 값에서 `&`와 `|`는 `AND`, `OR` 비트연산이지만 타입에서는 인터섹션과 유니온입니다.
- `const`는 상수를 선언하고 `as const`는 리터럴의 추론된 타입을 변경합니다.
- `extends`는 sub-class, sub-type, 제네릭 타입의 한정자로 각각 사용할 수 있습니다.
- `in`은 루프(for (key in object)) 또는 `매핑된(mapped) 타입`에 사용할 수 있습니다.

## 아이템9 타입 단언보다는 타입 선언을 사용하기

```typescript
interface Person {
  name: string;
}

const alice: Person = { name: "Alice" }; // 타입 선언
const bob = { name: "Bob" } as Person; // 타입 단언

const alice: Person = {}; // 오류
const bob = {} as Person; // 문제 없음
```

타입 선언은 변수에 타입을 붙여서 선언된 타입임을 명시하지만 타입 단언은 타입을 추론한 것과 무관하게 Person으로 강제합니다. 위의 예에서 확인할 수 있듯이 타입 단언은 오류를 무시하고 강제하여 의도치 않은 결과를 야기할 수 있기 때문에, 타입 선언 방식을 사용하는 것이 좋습니다.
null이 아님을 의미하는 ! 문법 또한 단언문으로 간주할 수 있습니다. 타입스크립트보다 개발자 스스로가 타입정보에 대해서 확신할 수 있는 상황에만 단언문을 사용하고 이외의 경우는 지양할 필요가 있습니다.

## 아이템10 객체 래퍼 타입 피하기

자바스크립트에는 객체 외에 기본 타입(primitive type)에는 string, number, bigint, boolean, undefined, symbol 그리고 null 이 존재하며 모두 불변이고, 메서드를 가지지 않습니다. 하지만 각 타입에 대응되는 래퍼(Wrapper) 타입을 통해서 메소드를 제공합니다.

- string -> String
- number -> Number
- boolean -> Boolean
- symbol -> Symbol
- bigint -> Bigint
  래퍼 타입이 존재한다는 것을 이해하는 수준이면 되며 래퍼 타입을 직접 사용하는 것은 지양해야 합니다. 정상적으로 동작하는 것처럼 보이지만, 문제가 발생하는 경우(string -> String 할당 가능하나, String -> string 할당 시 문제)가 있으며, 결국 런타임에서는 기본 타입으로 변환되어 실행되기 때문입니다.

## 아이템11 잉여 속성 체크의 한계 인지하기

```typescript
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present", // 리터럴 객체를 바로 할당하려고 할때 잉여 속성 체크가 동작함
};

const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present", // 리터럴 객체를 바로 할당하려고 할때 잉여 속성 체크가 동작함
};
const r2: Room = obj; // 정상. 잉여 속성 체크가 동작하지 않음
```

위의 예시와 같이 리터럴 개체를 변수에 바로 할당하거나 함수의 매개변수로 전달할 때 잉여 속성 체크가 동작합니다. 하지만 마지막 부분에서와 같이 임시 변수에 리터럴 객체를 할당한 후에 사용하게 되면 잉여 속성 체크는 동작하지 않고 할당 가능 검사(할당하려는 타입에 속성이 존재하는지 검사)만 수행하게 됩니다.

## 아이템12 함수 표현식에 타입 적용하기

함수 문장(statement)와 함수 표현식(expression)의 차이

```typescript
function dollDice1(sides: number): number { ... } // 문장
const rollDice2 = function(sides: number): number { ... }  // 표현식
const rollDice3 = (sides: number): number => { ... }  // 표현식
```

```typescript
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

함수 표현식은 타입으로 선언하여 다른 함수 표현식에 재사용이 가능하므로 함수 표현식을 사용하는 것이 좋습니다. 타입의 시그니처를 그대로 참조하므로 코드가 간결해진다는 장점도 있습니다.

```typescript
const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error("Request failed:" + response.status);
  }
  return response;
};
```

위의 예와 같이 fetch 함수의 기능을 확장하여 기존의 코드와 호환이 가능한 형태로 재작성을 하고 싶다면 `typeof` 키워드를 응용하면 됩니다. fetch의 시그니처를 그대로 따르게 되므로 문제의 소지를 차단할 수 있습니다.

## 아이템13 타입과 인터페이스의 차이점 알기

타입과 인터페이스는 동일한 점이 많습니다.

- 잉여 속성 체크가 동작합니다.
- 인덱스 시그니처를 사용할 수 있습니다.
- 함수 타입도 정의할 수 있습니다.
- 제네릭을 사용할 수 있습니다.
- 타입과 인터페이스는 서로를 확장할 수 있습니다.

```typescript
interface IA extends TA {// 타입을 인터페이스로 확장 (extends)
  property: string
};

type TB = IB & { property: string } ;  // 인터페이스를 타입으로 확장(&연산자)

class ClassA implements TA // or IA: 클래스를 선언할 때는 타입, 인터페이스 모두 implements로 확장 가능
```

타입과 인터페이스의 차이점과 특성을 명확하게 이해해야 합니다.

- 타입은 `|`연산자로 연결된 유니온 타입이 있지만, 유니온 인터페이스라는 개념은 없습니다.
- 인터페이스를 이용하여 타입을 확장할 수 있지만 유니온 타입을 확장할 수 없습니다.

```typescript
type a = { a: string };
type b = { b: string };
type ab = a | b;

// 불가: An interface can only extend an object type or intersection of object types with statically known members.(2312)
interface i1 extends ab {
  name: string;
}
```

- 타입은 유니온 타입을 확장할 수 있습니다.

```typescript
type nv = (a | b) & { name: string };
const v1: nv = { a: "test", name: "name test" };
```

- 인터페이스는 `선언 병합(declaration merging)`을 사용할 수 있습니다.

```typescript
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const t: IState = {
  name: "KR",
  capital: "seoul",
  population: 5000,
};
```

타입스크립트에서 여러 버전의 자바스크립트를 지원하기 위해서 interface `선언 병합`을 사용합니다.

## 아이템14 타입 연산과 제너릭 사용으로 반복 줄이기

타입의 중복은 코드의 중복에 비해서 덜 민감한 경향이 있지만 발견하거나 수정이 어려운 문제를 발생시킬 가능성이 있습니다. 따라서 타입 확장 기법을 이용하여 타입의 중복을 최소화 하려는 노력이 필요합니다.

- 타입에 이름 붙이기
- `extends`, `&`연산자 사용
- `제네릭` 사용

태그된 유니온 타입에서 인덱싱을 사용하면 간단하게 태그를 정의할 수 있습니다.

```typescript
interface SaveAction {
  type: "save";
}

interface LoadAction {
  type: "load";
}

type Action = SaveAction | LoadAction;
// type ActionType = 'save' | 'laod';  // type 추가에 대응하기 어려움
type ActionType1 = Action["type"]; // 'save' | 'load'
type ActionType2 = Pick<Action, "type">; // { type: 'save' | 'load' }
```

```typescript
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}

type OptionUpdate = { [k in keyof Options]?: Options[k] };
type OptionUpdate2 = Partial<Options>;
```

`제네릭`은 타입을 위한 함수와 동치이지만 타입의 범위를 제한하는 것이 중요합니다. `extends`를 이용해서 `제네릭`의 범위를 제한 할 수 있습니다.

```typescript
// K의 범위를 T의 인덱스로 제한함
type Pick<T, K extends keyof T> = { [k in K]: T[k] };

// lib.es5.d.ts 참조
type Partial<T> = { [P in keyof T]?: T[P] | undefined };
type Required<T> = { [P in keyof T]-?: T[P] };
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

표준 라이브러리에 정의되어 있는 Pick, Partial, ReturnType 같은 제네릭에 익숙해져야 합니다.

## 아이템15 동적 데이터에 인덱스 시그니처 사용하기

타입스크립트의 `인덱스 시그니처`를 이용하면 유연하게 타입을 선언할 수 있습니다.

```typescript
type Rocket = { [property: string]: string };
type Rocket = { [property: string]: string};
const rocket: Rocket = {
    name: 'test,
    Name: 'test',
    namme: 'test
}
```

반면에 사용 시 예제와 같이 단점이 있습니다.

- 잘못된 키를 사용할 수 있고 특정 키를 필요로 하지 않스비나.
- 값은 모두 `string` 타입입니다.
- 언어서비스의 지원을 받을 수 없습니다.
  따라서 본 경우는 `interface`를 사용해야 합니다. 데이터의 키를 미리 알 수 없는 상황(CSV parsing)에서는 사용하지 않을 수 없는 예외적인 경우를 제외하고는, 대부분 필드들은 제한되어 있고 이러한 경우에는 `인덱스 시그니처` 사용을 지양해야 합니다.

키가 `선택(Optional)`적이라면 유니온 타입이나 interface를 통해서 선택적 타입으로 지정하면 됩니다. 혹은 `Record` 제네릭 타입 혹은 매핑된 타입을 사용할 수 있습니다.

```typescript
interface R1 {
  a: number;
  b?: string;
  c?: number;
}
type R2 =
  | { a: number }
  | { a: number; b: string }
  | { a: number; b: string; c: string };

type R3 = Record<"a" | "b", number>;
type R4 = { [k in "a" | "b"]: number };
type R5 = { [k in "a" | "b"]: k extends "b" ? string : number };
```

## 아이템16 number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

자바스크립트는 배열의 인덱스로 `string` 타입을 사용합니다. 이 때문에 발생하는 혼란이 있어서, 타입스크립트 배열의 인덱스는 `number` 타입으로 제한되어 있습니다.
하지만 런타임에서는 `string` 타입이며, 굳이 `number` 타입을 사용할 이유는 없습니다. 배열의 데이터만 필요할 뿐 이외의 다른 함수(`push`, `concat` 등)가 필요하지 않다면 타입스크립트에서 제공하는 `ArrayLike`를 이용하는 것이 좋습니다.

```typescript
// lib.es5.d.ts
interface ArrayLike<T> {
  readonly length: number;
  readonly [n: number]: T;
}
```

## 아이템 17 변경 관련된 오류 방지를 위해 readonly 사용하기

- 어떤 함수가 매개변수로 전달받는 인자의 값을 변경하지 않는 경우에는 `readonly`를 붙여서 전달 받습니다.
- `readonly`가 붙게 되면 `서브 타입`으로 인식되므로 원래의 타입에 할당할 수 없게 됩니다.
- `readonly`와 `const`의 차이를 이해해야합니다.

```typescript
const a = "a";
a = "b"; // Cannot assign to 'a' because it is a constant.(2588)

const b: readonly string[] = ["test"];
b.push("test2"); // Property 'push' does not exist on type 'readonly string[]'.(2339)

let c: readonly string[] = ["test"];
c.push("test2"); // Property 'push' does not exist on type 'readonly string[]'.(2339)
c = ["test2"];
```

- `readonly`는 `얕게` 동작합니다.

```typescript
interface IA {
  readonly x: {
    y: string;
  };
}
const a: IA = { x: { y: "test " } };
a.x = { y: "y" }; // Cannot assign to 'x' because it is a read-only property.(2540)
a.x.y = "y";
```

## 아이템18 매핑된 타입을 사용하여 값을 동기화하기

어떤 타입에 대해 필드가 추가/제외 등의 변경이 있을 때 이 타입을 사용하는 곳(함수)에도 수정이 필요한 경우가 많이 있습니다.

```typescript
interface ScatterProps {
  xs: number;
  ys: number;
  xRange: number;
  color: number;
  onClick: () => void;
}

const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  color: false,
  onClick: false,
};

function shouldUpdate(oldProp: ScatterProps, newProp: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProp) {
    if (oldProp[k] !== newProp[k] && REQUIRES_UPDATE[k]) return true;
  }
  return false;
}
```

위의 예시와 같이 `REQUIRES_UPDATE`에 `ScatterProps`의 필드를 맵핑해둠으로써 이후에 `ScatterProps`의 필드에 변경이 일어나더라도 `REQUIRES_UPDATE`에서 오류가 발생하게 되므로 쉽게 인지할 수 있습니다.~

# 개인 독후감

## 이은택

이번 장은 내용이 특히 많았는데, 실제로 해당 장에서 집중해서 봤던 부분은 interface 와 type 의 장단점, 제네릭을 사용하여 중복 코드를 줄일 수 있는 부분을 위주로 살펴보았다. 해당 내용은 현재 개발을 진행하면서 쉽게 놓칠 수 있었던 부분이였는데, 리팩토링 시 유용하게 사용할 수 있을 것 같다는 생각이 들었다.

## 김련호

중요하고 어려운 내용들이 많이 있었습니다. 특히 `타입`과 `값`의 구분을 하는 내용을 다룬 부분이 익숙하지 않았습니다. 그 외에는 타입스크립트를 사용하면서 팁이 될만한 내용이 많은 도움이 되었고, 특히 `제네릭`에 대해서는 깊은 이해 없이 사용하던 부분을 명확하게 알게 되어 좋았습니다.

## 강현구

가벼운 마음으로 챕터를 시작했는데, 양이 방대해서 놀랐다. 이번 장은 타입스크립트는 어떻게 활용하면 좋은지
실무에도 바로 적용 가능하며, 레거시 코드도 리팩토링을 생각해볼만큼 TS를 사용하며 몰랐던 부분을 많이 깨달았다.
책만 봐서는 내용을 이해가 쉽지않아 예제 코드를 작성하며 하나씩 진행했고 아직도 이해가 가지 않는 부분이 많아 몇번 더 정독 할 생각이다.

## 김지섭

전체적으로 분량이 많아 숙지하는데에 오래 걸렸고, 복습이 많이 필요해보였습니다. 한번에 읽으면서 이해가 가지 않았던 부분은 아이템 7 부분에서 타입의 인터섹션(intersection, 교집합)을 구하는 부분이었으며 이해하는데 시간이 조금 걸렸습니다. 그리고 아이템 13에서 기존에 불명확했던 타입과 인터페이스의 차이에 대해 명확하게 알게되어 도움이 많이 되었습니다.

# 아젠다

1. DTO를 작성할 때 `type`, `interface` 키워드를 사용하시나요? DTO 외에 `type`, `interface` 키워드를 사용할 때가 있나요?
