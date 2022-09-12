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

# 개인 독후감

## 이은택
실제로 API 개발을 진행하면서 이미 규격화 된 개발 방법이 있다보니 어느 순간부터 any 와 unknown {} 를 제외하고 개발하고 있습니다.
5장에서는 ANY를 최대한 좁혀서 사용하는 방법과 `unknown` `{}` 의 차이에 대해 강조하고 있습니다. any를 무작정 절대 사용하면 안되는 것으로 생각했었는데, 실제 저자는 숨겨져있는 함수에 any를 사용하는 방법도 안내되어 있었습니다. 추후에 javascript를 마이그레이션하거나 외부 API와 연동하여 개발할 때, any를 사용하게 된다면 해당 장에서 나와 있는 부분들을 잘활용하여 개발헤야겠다고 생각이 들었습니다.

## 김련호


## 강현구


# 아젠다

# 1. 





