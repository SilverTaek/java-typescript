# 요약

## 5장 any 다루기
## 아이템38 any 타입은 가능한 한 좁은 범위에서만 사용하기

### 함수
```typescript
function processBar(b: Bar) { /* ... */ }
function f1() {
  const x: any = expressionReturningFoo();  // X
  processBar(x);
  return x;
  //반환 값의 타입이 any면 코드 전체에 영향을 줄 수 있음.
}

function f2() {
  const x = expressionReturningFoo();
  processBar(x as any);  // O
  //좁혀서 사용하는 것을 권장
}
```

### 객체

```typescript
const config1: Config {
  a: 1,
  b: 2,
  c: {
    key: value // 에러, Foo 타입에 foo 프로퍼티가 필요하지만 Bar 타입에는 없습니다.
  }
}

const config2: Config {
  a: 1,
  b: 2,
  c: {
    key: value 
  } as any    // 정상, a,b 프로퍼티 마저 타입체크 안됨
  //객체 전체를 any로 단얺ㄴ면 다른 속성들(a와 b) 역시 타입 체크가 되지 않는 부작용이 생깁니다.
}

const config3: Config {
  a: 1,
  b: 2,
  c: {
    key: value as any  // 정상, a,b 프로퍼티 마저 타입체크 가능
  }
}
```

    의도치 않은 타입 안전성의 손실을 피하기 위해서 any의 사용 범위를 최소한으로 좁혀야 합니다.
    함수의 반환 타입이 any인 경우 타입 안전성이 나빠집니다. 따라서 any 타입을 반환하면 절대 안됩니다.
    강제로 타입 오류르르 제거하려면 any 대신 @ts-ignore을 사용하는 것이 좋습니다.


## 아이템39 any를 구체적으로 변형해서 사용하기

```typescript
function getLengthBad(array: any) {
  return array.length;
}

// better than above.
function getLength(array: any[]) {
  return array.length;
}
```
위 코드에서 getLength가 getLengthBad보다 나은 이유는 세 가지 이다.

- 함수 내의 array.length 타입이 체크됨.
- 함수의 반환 타입이 any대신 number로 추론됨.
- 함수 호출 될 때 매개변수가 배열인지 체크됨.

    any를 사용할 때는 정말로 모든 값이 허용되어야만 하는지 면밀히 검토해야 합니다.
    any보다 더 정확하게 모델링할 수 있도록 any[]또는 {[id: string]: any} 또는 () => any처럼 구체적인 형태를 사용해야 합니다.


## 아이템40 함수 안으로 타입 단언문 감추기

함수를 작성하다 보면, 외부로 드러난 타입 정의는 간단하지만 내부 로직이 복잡해서 안전한 타입으로 구현하기 힘든 경우가 많음.
함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만 불필요한 예외 사항까지 고려하여 타입 정보를 힘들게 구현할 필요는 없음.
함수 내부에서는 타입 단언을 사용하고 외부로 드러나는 부분만 타입 정의를 명확히 명시하는 정도로 끝내는 것이 나음.
프로젝트 전반에 타입 단언이 들어있는 것보단, 제대로 타입이 정의된 함수 안으로 감추는 것이 더 좋은 설계임.

//예시 코드 참고

    타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 합니다.
    불가피하게 사용해야 한다면 정확한 정의를 가지는 함수 안으로 숨기도록 합니다.

## 아이템41 any의 진화를 이해하기

타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정됩니다. 그 후에 정제될 수 있지만(예를 들어 null인지 체크해서), 새로운 값이 추가되도록 확장할 수는 없습니다.
그러나 any타입과 관련해서 예외인 경우가 존재합니다.

* 아래의 예시는 any의 진화 과정을 보여줍니다.

```typescript
function range(start: number, limit: number) {
  const out = []; // 타입이 any[]
  for(let i=start; i<limit; i++){
    out.push(i); //  out의 타입이 any
  }
  return out; // 타입이 number[]
}
```
    일반적인 타입들은 정제되기만 하는 반면, 암시적 any와 any[] 타입은 진화할 수 있습니다. 이러한 동작이 발생하는 코드를 인지하고 이해할 수 있어야 합니다.
    any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법입니다.

## 아이템42 모르는 타입의 값에는 any 대신 unknown을 사용하기

* 함수의 반환값과 관련된 형태
```typescript
function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}
```

* 변수 선언과 관련된 형태
```typescript
interface Feature {
id?: string | number;
geometry : Geometry;
property: unknown;
}
```

* 단언문과 관련된 형태
```typescript
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```
    unknown은 any 대신 사용할 수 있는 안전한 타입입니다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우라면 unknown을 사용하면 됩니다.
    사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 unknown을 사용하면 됩니다.
    {}, object, unknown의 차이점을 이해해야 합니다.

## 아이템43 몽키 패치보다는 안전한 타입을 사용하기

* 자바스크립트에서는 임의로 객체와 클래스에 임의의 속성을 추가할 수 있다.
```typescript
window.monkey = 'Tamarin';
document.monkey = 'Howler';
```
// 객체에 임의로 속성을 추가하는 것은 좋은 설계가 아님. 그렇기 때문에 다른 대안이 필요
- any 단언문을 사용할 수 있지만, any를 사용함으로써 타입 안전성을 상실하고 언어 서비스를 사용할 수 없게 된다는 것
- 대안 2가지
1. interface의 특수 기능 중 하나인 보강(augmentation)을 사용

```typescript
interface Document {
  monkey: string;
}
```

2. 더 구체적인 타입 단언문 사용

```typescript
interface MonkeyDocument extends Document {
  monkey: string;
}
```
    전역 변수나 DOM에 데이터를 저장하지 말고, 데이터를 분리하여 사용해야 합니다.
    내장 타입에 데이터를 저장해야 하는 경우, 안전한 타입 접근법 중 하나(보강이나 사용자 정의 인터페이스로 단언)를 사용해야 합니다.
    보강의 모듈 영역 문제를 이해해야 합니다.

## 아이템44 타입 커버리지를 추적하여 타입 안전성 유지하기

any 타입이 여전히 프로그램 내에 존재할 수 있는 두 가지 경우
1. 명시적 any 타입
    * ex) any[], {[key: string]: any}
2. 서드파티 타입 선언
    * @types 선언 파일로부터 any 타입이 전파되는 경우, 가장 극단적인 예 - 전체 모듈에 any 타입을 부여하는 경우, declare module 'my-module'; → my-module 에서 어떤 것이든 오류 없이 임포트할 수 있다
    * 타입에 버그가 있는 경우
    * 선언된 타입과 실제 반환된 타입이 맞지 않는 경우

    noImplicity가 설정되어 있어도, 명시작 any 또는 서드파티 타입 선언(@types)을 통해 any타입은 코드 내에 여전히 존재할 수 있다는 점을 주의해야 합니다.
    작성한 프로그램의 타입이 얼마나 잘 선언되었는지 추적해야 합니다. 추적함으로써 any의 사용을 줄여 나갈 수 있고 타입 안전성을 꾸준히 높일 수 있습니다.



# 개인 독후감

## 이은택
실제로 API 개발을 진행하면서 이미 규격화 된 개발 방법이 있다보니 어느 순간부터 any 와 unknown {} 를 제외하고 개발하고 있습니다.
5장에서는 ANY를 최대한 좁혀서 사용하는 방법과 `unknown` `{}` 의 차이에 대해 강조하고 있습니다. any를 무작정 절대 사용하면 안되는 것으로 생각했었는데, 실제 저자는 숨겨져있는 함수에 any를 사용하는 방법도 안내되어 있었습니다. 추후에 javascript를 마이그레이션하거나 외부 API와 연동하여 개발할 때, any를 사용하게 된다면 해당 장에서 나와 있는 부분들을 잘활용하여 개발헤야겠다고 생각이 들었습니다.

## 김련호

## 강현구

## 김지섭
책을 처음 읽을 때에는 any타입을 지양하여 코드를 작성하는 것이 좋다고만 알았는데, 실제로 any타입을 써야할 때를 알게되어 유용했습니다.
실제 코드를 작성할 때, 이 장의 내용을 바탕으로 코드를 작성해보고 비교 분석해보면 좋을 것 같습니다.


# 아젠다

# 1. 





