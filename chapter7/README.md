# 요약

### 아이템53. 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

#### enum 사용을 최소화 하기

- 문자열 `enum`과 숫자형 `enum`이 다르게 동작합니다.
  접근 방식의 차이, JS로 변환되었을 때의 차이가 있습니다.

```typescript
// .ts
enum Flavor {
  VANILLA = "vanilla",
  CHOCOLATE = "chocolate",
  STRAWBERRY = "strawberry",
}
Flavor.VANILLA; // 접근 가능
console.log(Flavor[0]); // 불가

enum Types {
  Circle = 0,
  Ellipse = 1,
  Polygon = 2,
  Line = 3,
}
Types.Circle; // 접근 가능
console.log(Types[0]); // "Circle"

// .js
var Flavor;
(function (Flavor) {
    Flavor["VANILLA"] = "vanilla";
    Flavor["CHOCOLATE"] = "chocolate";
    Flavor["STRAWBERRY"] = "strawberry";
})(Flavor || (Flavor = {}));
Flavor.VANILLA; // 접근 가능
console.log(Flavor[0]); // 불가
var Types;
(function (Types) {
    Types[Types["Circle"] = 0] = "Circle";
    Types[Types["Ellipse"] = 1] = "Ellipse";
    Types[Types["Polygon"] = 2] = "Polygon";
    Types[Types["Line"] = 3] = "Line";
})(Types || (Types = {}));
Types.Circle; // 접근 가능
console.log(Types[0]); // "Circle"

}
```

- JS와 동작이 다릅니다.

```typescript
getFlavor("vanilla"); // js에서는 가능하지만 ts에서는 불가
```

- 유니온 타입 사용을 권장합니다.

```typescript
type Flavor = "vanilla" | "chocolate" | "strawberry";
```

- [TypeScript enum을 사용하지 않는 게 좋은 이유를 Tree-shaking 관점에서 소개합니다.](https://engineering.linecorp.com/ko/blog/typescript-enum-tree-shaking/)

#### 생성자의 매개변수 사용하지 않기

멤버 변수와 생성자의 매개변수 멤버를 혼합해서 사용할 때 설계가 혼란스러울 수 있습니다.

```typescript
class Person {
  first: string;
  last: string;
  constructor(public name: string) {
    [this.first, this.last] = name.split(" ");
  }
}
```

#### 네임스페이스와 트리플 슬래시 임포트

```typescript
namespace foo {
  function bar() {}
}
/// <reference path='other.ts'>
```

#### 데코레이터

데코레이터는 아직 비표준 기능이며, 따라서 향후에 호환성이 깨질 가능성이 있습니다. 따라서 사용을 지양해야 합니다.

[TC39 Proposals](https://github.com/tc39/proposals)

### 아이템54. 객체를 순회하는 노하우

```typescript
const obj = {
  one: "uno",
  two: "dos",
  three: "tres",
};

for (const k in obj) {
  const v = obj[k];
  // k는 string으로 추론되었지만, obj의 키는 3가지로 제한(one, two, three)
}
```

```typescript
// k를 아래와 같이 선언해 주면 키의 범위를 정확하게 선언할 수 있습니다.
let k: keyof typeof obj;
```

하지만 위와 같이 오류 없이 순위하더라도 구조덕 타이핑을 사용하는 특성 때문에 다른 문제가 발생할 수 있습니다.

```typescript
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  let k: keyof ABC;
  for (k in abc) {
    const v = abc[k]; // string | number, Date 타입은 무시되었음
  }
}
const x = { a: "a", b: "b", c: 2, d: new Date() };
foo(x); // 호출 가능
```

골치 아픈 문제를 피하고 싶다면 `Object.entries()`를 사용하면 됩니다.

```typescript
interface ABC {
  a: string;
  b: string;
  c: number;
}
function foo(abc: ABC) {
  for (const [k, v] of Object.entries(abc)) {
    k; // Type is string
    v; // Type is any
  }
}
```

### 아이템55. DOM 계층 구조 이해하기

#### EventTarget

`DOM` 타입 중 최상위 타입인 `EventTarget`은 `addEventListener`, `RemoveEventListener` 등 밖에 할 수 없습니다. 예제에서 currentTarget은 실제로 `HTMLElement` 타입이지만 타입관점에서 `EventTarget`이 될 수 있습니다.

#### Node

다음으로 `Node` 타입은 `Element` 타입을 포함하지만 `Element`가 아닌 `Node` 도 있습니다. 텍스트 노드와 주석이 그 예입니다.

#### Element

그 다음으로는 `Element` 타입이 있습니다. `Element` 타입에는 `HTMLElement`와 `SVGElement` 가 포함됩니다.

#### HTMLElement

마지막으로 `HTMLElement`는 `HTML***Elment`를 포함합니다.

그런데 `getElementById` 등으로 노드를 엘리먼트를 선택하는 경우, 타입스크립트 보다 개발자가 더 정확하게 어떤 `HTML***Element`인지 알 수 있기 때문에 `as` 로 타입 단언문을 사용해도 됩니다.

```typescript
document.getElementById("main-div") as HTMLDivElement;
```

### 아이템56. 정보를 감추는 목적으로 private 사용하지 않기

TS에서 `public`, `protect`, `private` 접근 제어자를 제공하는데, 이것은 JS에서도 접근 제어가 되는 것처럼 오인할 수 있도록 하기 때문에 지양해야합니다. 그리고 TS에서 `private`을 사용하더라도 타입 단언문을 통해 간단히 접근할 수 있습니다.

```typescript
class Diary {
  private secret = "cheated on my English test";
}

const diary = new Diary();
(diary as any).secret; // OK
```

JS에서 데이터 은닉을 위해 `private` 기능을 `해시(#)`를 통해 제공하므로 TS에서 `해시(#)`을 사용하여 `private`를 구현할 수 있습니다. 또는 클로저 기법을 이용해도 동일하게 데이터 은닉을 할 수 있습니다.

### 아이템57. 소스맵을 사용하여 타입스크립트 디버깅하기

TS -> JS로 트랜스파일링 되면서 사실상 디버깅이 거의 불가능합니다. 이에 브라우저 제조사들이 `source map` 이라는 것을 지원하게 되었습니다. `tsconfig`에 아래와 같이 설정하고 컴파일을 하면 \*.js.map 이라는 파일이 생기는데, 실행 시에 브라우저의 디버거에서 `index.ts` 라는 중간 파일이 생성되며 이 파일을 통해서 `break point`를 설정하거나 변수를 조사 할 수 있게 됩니다.

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "sourceMap": true
  }
}
```

`source map`을 적용하여 컴파일 후 `vscode 디버거`를 통해서도 디버깅이 가능합니다.

## 7장

# 개인 독후감

## 이은택

`TS보다는 ECMAScript 기능 쓰기` 장에서는 실제로 개발할 때 js 와 ts 의 경계를 구분하고 개발하고 있지 않아 해당 챕터를 읽으면서 경각심을 가지게 되었다. 특히 enum 을 사용할 때, 무지성으로 사용했는데 조금 더 생각을 해볼필요가 있다고 느껴졌다. 객체 순회가 필요한 경우는 대부분 map이나 filter 를 활용하고 있지만 단순 순회를 할 경우가 생기면 책에서 나오는 예시처럼 of 나 foreach를 사용하는 방법에 대해 생각해봐야겠다. 또, private 같은 경우 다른 곳에서 사용하지 못하도록 숨기는 용도로 사용하곤 했는데, 책에서 나오는 방법에 대해서는 조금 더 생각을 해볼필요가 느껴졌다.

## 김련호

Javascript와 express.js를 사용해서 개발을 할 때 가장 필요한 기능이 enum이였는데, TS에서 지원하고 있어서 가장 유용하게 사용하고 있었습니다. 책에서도 지양해야한다는 언급이 있고, 이외의 관점에서도 명확하게 사용해야한다는 의견이 있어서 실무에서도 유니온 타입으로 대체를 시도 하고 있습니다. 데코레이터에 관한 부분은 꽤 충격적이였습니다. nestjs에서 거의 필수적으로 사용하고 있는 기능이지만, 아직 비표준이고 향후에 표준화 되는 과정에서 호환성 문제가 생길 우려가 있다는 생각을 하지 못했는데 production 레벨에서는 충분히 문제가 될 소지가 있는 부분이라 이 부분은 조금 더 생각해 볼 필요가 있을 것 같습니다.

## 강현구

ECMAScript 에 대해 강조하고 있어서 계속 `ts`를 사용한다면 최신 업데이트 이력은 계속 모니터링하며 공부가 필요해보입니다.
`enum` 열거형에 대해서는 책에서 7장 외에도 여러 번 언급을 했었는데, 지양하는 습관을 가져야 할 것 같습니다.
`private` 사용하지 않기에 대해서는 해시(#) 태그를 붙이는 것에 대해 안내 되어있는데, 확실히 데이터를 감추고 싶다면 `클로저`를 활용하면 좋겠다고 생각했습니다.

## 김지섭

'TS와 ECMAScript에 대해서 구분해서 생각해보자'라는 생각을 한번도 한 적이 없었는데, 7장을 보면서 필요하겠구나라고 생각했습니다.
평소에 enum 타입을 아무런 생각 없이 무분별하게 사용했었는데 주의하여 생각하면서 사용해야겠다고 생각이 들었습니다. 
실제로 production 배포단계에서 어떠한 문제가 있는지 한번 점검해봐야겠습니다.

# 아젠다
