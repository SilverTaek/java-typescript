# 요약

### INTRO

타입 추론은 수동으로 명시해야하는 타입 구문의 수를 엄청나게 줄여 주기 때문에, 코드의 전체적인 안정성이 향상됩니다.

- 어떻게 타입스크립트가 타입을 추론할까?
- 언제 타입 선언을 작성해야할까?
- 어떤 상황에 타입 추론이 가능하더라도 명시적으로 타입 선언을 작성하는 것이 필요할까?

## 아이템19 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 타입스크립트가 타입을 추론할 수 있다면 타입 구문을 작성하지 않는 게 좋습니다.

```typescript
let x: number = 12;
let x = 12;
```

    만약 타입을 확신하지 못한다면 편집기를 통해 체크하면 됩니다.

- 이상적인 경우 함수/메서드의 시그니처에는 타입 구문이 있지만, 함수 내의 지역 변수에는 타입 구문이 없습니다.

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id; // Bad
  const { id, name, price } = product; // Good
  const { id, name, price }: { id: string; name: string; price: number } =
    product; // Bad
}
```

- 추론될 수 있는 경우라도 객체 리터럴과 함수 반환에는 타입 명시를 고려해야 합니다. 이는 내부 구현의 오류가 사용자 코드 위치에 나타나는 것을 방지해 줍니다.

### 객체 리터럴

```typescript
const elmo: Prouct = {
  name: "Tickle Me elmo",
  id: "0404040",
  price: 29,
};
```

    이런 정의의 타입을 명시하면, 잉여 속성 체크(아이템 11)가 동작합니다.

### 함수 반환

```typescript
function getQuote(ticker: string): Promise<number> {
  if (ticker in cache) {
    return cache[ticker]; // 여기에서 오류 발생
  }
  return fetch("")
    .then((res) => res.json())
    .then((quote) => {
      cache[ticker] = quote;
      return quote;
    });
}
```

    반환타입을 명시하면, 구현상의 오류가 사용자 코드의 오류로 표시되지 않습니다.
    함수에 대해 더욱 명확하게 알 수 있습니다.(함수 구조파악 용이)
    명명된 타입을 사용할 수 있습니다.(반환하는 값이 명확해짐)

## 아이템20 다른 타입에는 다른 변수 사용하기

- 변수의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않습니다.
- 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록 합니다.

```typescript
let id = "test"; // string
testFunction(id);
id = 413; // number
testFunction2(id);
```

    이렇게 사용하면 안됨.

## 아이템21 타입 넓히기

- 타입스크립트가 넓히기를 통해 상수의 타입을 추론하는 법을 이해해야 합니다.
- 동작에 영향을 줄 수 있는 방법인 const, 타입 구문, 문맥, as const에 익숙해져야 합니다.

```typescript
const x = "x"; // 타입은 'x'

const obj1 = { x: 1, y: 2 }; // 타입은 { x: number, y: number }
const obj2 = { x: 1 as const, y: 2 }; // 타입은 { x: 1, y: number }
const obj3 = { x: 1, y: 2 } as const; // 타입은 { readonly x: 1, readonly y: 2 }
```

    const 키워드를 통해서 더 좁은 타입으로 선언할 수 있도록 노력하자.
    그렇지 않으면 타입스크립트는 가능한 많은 타입을 염두해두므로 오류가 발생할 여지가 있다.
    하지만 const 키워드도 만능이 아니다. 배열과 객체 선언에 있어 오류가 발생할 여지가 있다.

### 타입 추론의 강도 직접 제어하기

- 명시적 타입 구문을 제공한다.

```typescript
const v: { x: 1 | 3 | 5 } = {
  x: 1,
};
```

- 타입 체커에 추가적인 문맥을 제공한다.
- const 단언문을 사용한다.

```typescript
const v2 = {
  x: 1 as const,
};
```

## 아이템22 타입 좁히기

- 타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말합니다.
  타입을 좁히는 방법
  - 분기문을 통해서
  - instanceof를 통해서
  - Array.isArray를 통해서
  - 명시적 태그를 붙이는 것을 통해서, 태그된 유니온(tagged union, discriminated union)
  - 사용자 정의 타입 가드(만약 타입스크립트가 타입을 식별하지 못한다면, 커스텀 함수를 도입할 수 있음)
- 분기문 외에도 여러 종류의 제어 프름을 살펴보며 타입스크립트가 타입을 좁히는 과정을 이해해야 합니다.
- 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원활하게 만들 수 있습니다.

```typescript

```

    편집기에서 타입을 조사하는 습관을 가지면 타입 좁히기가 어떻게 동작하는지 자연스레 익힐 수 있습니다.
    타입스크립트에서 타입이 어떻게 좁혀지는지 이해한다면
    타입 추론에 대한 개념을 잡을 수 있고, 오류 발생의 원인을 알 수 있으며, 타입 체커를 더 효율적으로 이용할 수 있습니다.

## 아이템23 한꺼번에 객체 생성하기

- 속성을 제각각 추가하지 말고 한꺼번에 객체로 만들어야 합니다. 안전한 타입으로 속성을 추가하려면 객체 전체({...a, ...b})를 사용하면 됩니다.

```typescript
interface Point = { x: number, y: number }
const pt: Point = {
    x: 3,
    y: 3
}

```
- 객체에 조건부로 속성을 추가하는 방법을 익히도록 합니다.

```typescript
declare let hasMiddle: boolean;
const firstLast = { first: 'Harry', lsat: 'Truman'};
const president = { ... firstLast, ...(hasMiddle ? { middle: 'S' } : {})};

```

## 아이템24 일관성 있는 별칭 사용하기

- 별칭은 타입스크립트가 타입을 좁히는 것을 방해합니다. 따라서 변수에 별칭을 사용할 때는 일관되게 사용해야 합니다.

```typescript
interface Coordinate {
  x: number;
  y: number;
}
interface BoundingBox {
  x: [number, number];
  y: [number, number];
}
interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[];
  bbox?: BoundingBox;
}
function isPointInpolygon(polygon: Polygon) {
  const box = polygon.bbox;
  if (polygon.bbox) {
    polygon.bbox.x; // 정상
    box.x; // Object is possibly 'undefined'.
  }
}

```

## 아이템25 비동기 코드에는 콜백 대신 async 함수 사용하기

- 콜백보다는 프로미스를 사용하는 게 코드 작성과 타입 추론 면에서 유리합니다.
- 가능하면 프로미스를 생성하기 보다는 async와 await를 사용하는 것이 좋습니다. 간결하고 직관적인 코드를 작성할 수 있고 모든 종류의 오류를 제거할 수 있습니다.
- 어떤 함수가 프로미스를 반환한다면 async로 선언하는 것이 좋습니다.

```typescript
// callback
fetchURL(url1, function (res1) {
  fetchURL(url2, function (res2) {
    fetchURL(url3, function (res3) {
      //
    });
  });
});

// promise
const fetchPromise = fetch(url1);
fetchPromise
  .then((res1) => {
    return fetch(url2);
  })
  .then((res2) => {
    return fetch(url3);
  })
  .then((res3) => {
    //
  })
  .catch((error) => {});

// async/await
async function fetchPages() {
  try {
    const res1 = await fetch(url1);
    const res2 = await fetch(url2);
    const res3 = await fetch(url3);
  } catch (e) {}
}

```

## 아이템26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

- 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴봐야 합니다.
- 변수를 뽑아서 별도로 선언했을 때 오류가 발생한다면 타입 선언을 추가해야 합니다.
- 변수가 정말로 상수라면 단언(as const)을 사용해야 합니다. 그러나 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야합니다.

###튜플 사용 시 주의점
```typescript
function panTo(where: [number, number]) {}

panTo([10, 20]); // 정상

const loc = [10, 20];
panTo(loc); // 'number[]' 형식의 인수는 '[number, number]' 형식의 매개변수에 할당될 수 없습니다.

const loc: [number, number] = [10, 20];
const loc = [10, 20] as const;

function panTo(where: readonly [number, number]) {}

const loc = [10, 20] as const;
panTo(loc);
```
###객체 사용 시 주의점
```typescript
type Language = "javascript" | "typescript";
interface GovernedLanguage {
  language: Language;
  organization: string;
}

function complain(language: GovernedLanguage) {}

complain({ language: "typescript", organization: "Microsoft" }); // 정상

const ts = { language: "typescript", organization: "Microsoft" };
complain(ts);
// 'string' 형식은 'Language' 형식에 할당할 수 없습니다.

타입 선언을 추가하거나(const ts: GovernedLanguage = ...) 상수 단언(as const)을 사용해서 해결할 수 있다.
```
###콜백 사용 시 주의점
```typescript
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
  fn(Math.random(), Math.random());
}

callWithRandomNumbers((a, b) => {
  a; // number
  b; // number
});

const fn = (a, b) => {
  // 'a', 'b' 매개변수에는 암시적으로 'any' 형식이 포함됩니다.
};
callWithRandomNumbers(fn);

const fn = (a: number, b: number) => {};
callWithRandomNumbers(fn);

```

## 아이템27 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기 보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는 것이 좋습니다.

```typescript
const csvData = "...";
const rawRows = csvData.split("\n");
const headers = rawRows[0].split(",");

// 절차형
const rows1 = rawRows.slice(1).map((rowStr) => {
  const row = {};
  rowStr.split(",").forEach((val, j) => {
    row[headers[j]] = val;
    // '{}' 형식에서 'string' 형식의 매개변수가 포함된 인덱스 시그니처를 찾을 수 없습니다.
  });
  return row;
});

// 함수형
const rows2 = rawRows.slice(1).map(
  (rowStr) =>
    rowStr
      .split(",")
      .reduce((row, val, i) => ((row[headers[i]] = val), row), {})
  // '{}' 형식에서 'string' 형식의 매개변수가 포함된 인덱스 시그니처를 찾을 수 없습니다.
);

// 서드파티 라이브러리
import _ from "lodash";
const rows3 = rawRows
  .slice(1)
  .map((rowStr) => _.zipObject(headers, rowStr.split(",")));
  // 타입이 _.Dictionary<string>[]
```

# 개인 독후감

## 이은택

평소 타입 좁히기 그 중에서도 타입 가드에 대해서 어떻게 잘사용하는지 궁금했었다. 해당 장에서는 타입 넓히기, 좁히기, 타입 가드에 대해 다루고 있어서 관련 부분에 대한 도움을 받을 수 있었다. 이펙티브 타입스크립트에서 다뤄야하는 내용인지는 모르겠지만, 비동기 동기 관련 async await를 권장하는 내용도 있었다. 처음 업무를 하면서 코드가 로대시로 되어있어서 알아보기 어려웠지만 책에 기술된 것처럼 로대시나 내장 함수를 사용하면 코드를 보다 간결하게 사용할 수 있어서 가독성이 좋아지는 효과가 있는 것 같다.

## 김련호

## 강현구

## 김지섭

# 아젠다

1.
2.
3.
