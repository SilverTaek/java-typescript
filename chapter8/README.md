# 요약

# 제8장 타입스크립트로 마이그레이션하기

타입스크립트는 자바스크립트보다 개선된 언어로 프로젝트를 새로 시작 시 타입스크립트를 적극 사용

2017년 한 조사에 따르면 깃헙에 있는 js 프로젝트에서 발견된 버그의 15%는, 타입 스크립트 사용 시 컴파일 시점에 미리 방지가 가능했을것으로 설명한다.

## 아이템 58 모던 자바스크립트로 작성하기

타입스크립트는 타입 체크 기능 외 ts 코드를 특정 버전의 js 로 컴파일하는 기능을 가지고 있다.

1999년에 나온 ES3 버전의 자바스크립트 코드까지 컴파일 되며, 타입스크립트 컴파일러를 자바스크립트 ‘트랜스파일러’로 사용할 수 있습니다.

### ECMAScript 모듈 사용하기

ES2015 이전에는 코드를 개별 모듈로 분할하는 표준이 없었지만, 이후부터 `import` 와 `export` 를 사용하는

ECMAScript 모듈(ES Module) 이 표준이 되었습니다.

마이그레이션 대상인 자바스크립트 코드가 단일 파일이거나 비표준 모듈 시스템을 사용 중이라면 ES모듈로 전환하는 것이 좋다.

**모던 자바스크립트는 최신 버전의 자바스크립트를 의미** 

```jsx
// CommonJS 
// a.js
const b = require(./b');
console.log(b.name);

// b.js
const name = 'Module B';
module.exports = {name};

// ECMAScript Module
// a.ts
import * as b from './b';
console.log(b.name);

// b.ts
export const name = 'Module B';
```

### 프로토타입 대신 클래스 사용하기

과거에 자바스크립트는 프로토타입 기반의 객체 모델을 사용했으나

프로토타입 모델보다는 견고하게 설계된 클래스 기반 모델을 사용하는것이 좋음.

```jsx
// Prototype 기반
function Person(first, last) {
	this.first = first;
	this.last = last;
}

Person.prototype.getName = function() {
	return this.first + ' ' + this.last;
}

const marie = new Person('Marie', 'Curie');
const personName = marie.getName();

// Class 기반
class Person {
  first: string
  last: string

  constructor(first: string, last: string) {
    this.first = first
    this.last = last
  }

  getName() {
    return this.first + ' ' + this.last
  }
}

const marie = new Person('Marie', 'Curie')
const personName = marie.getName()
```

`prototype` 기반 Person 객체보다 `class` 로 구현한 Person 객체가 문법이 간결하고 직관적

### var 대신 let/const 사용하기

자바스크립트 `var` 키워드의 `스코프`(scope) 규칙 문제로 다양한 버그의 발생 소지가 있었음.

모던 자바스크립트에서는 `var` 키워드를 `let`이나 `const`로 변경 사용을 권장

```jsx
function foo() {
	bar();
	function bar() {
		console.log('hello');
	}	
}
```

foo 함수 호출 시 bar 함수의 정의가 호이스팅 되어 가장 먼저 수행되기 때문에

호출에는 문제가 없으나, 호이스팅은 실행 순서를 예상하기 어렵게 만들고 직관적이지 않기 때문에

함수 표현식(const bar = () ⇒ { … }) 을 사용 권장

### for(;;) 대신 for-of 또는 배열 메서드 사용하기

```jsx
// C 스타일의 for 루프
for (var i = 0; i < array.length; i++) {
  const el = array[i]
  // ...
}

// for-of 루프
for (const el of array) {
  // ...
}

// 인덱스 변수가 필요한 경우엔 forEach 메서드 사용
array.forEach((el, i) => {
  // ...
})
```

### 함수 표현식보다 화살표 함수 사용하기

`this` 키워드는 일반적인 변수들과는 다른 스코프 규칙을 가지기 때문에, 자바스크립트에서 가장 어려운 개념 중 하나이다. 일반적으로 `this`가 클래스 인스턴스를 참고하는 것을 기대하지만 다음 예제처럼 예상치 못한 결과가 나오는 경우도 있다.

```jsx
class Foo {
  method() {
    console.log(this)
    ;[1, 2].forEach(function (i) {
      console.log(this)
    })
  }
}
const f = new Foo()
f.method()
// Prints Foo, undefined, undefined in strict mode
// Prints Foo, window, window (!) in non-strict mode

class Foo {
  method() {
    console.log(this)
    ;[1, 2].forEach(i => {
      console.log(this)
    })
  }
}
const f = new Foo()
f.method()
// Always prints Foo, Foo, Foo
```

인라인에서는 일반 함수보다 화살표 함수가 더 직관적이며 코드가 간결하기 때문에 가급적 화살표 함수 사용 권장

추가로 컴파일러 옵션에 `noImplicitThis` (또는 strict) 설정 시 타입스크립트가 this 바인딩 관련 오류를 

표시하므로 설정하는것이 좋습니다.

### 단축 객체 표현과 구조 분해 할당 사용하기

**단축 객체 표현**

```jsx
// AS-IS
const x = 1,
  y = 2,
  z = 3
const pt = {
  x: x,
  y: y,
  z: z,
}

// TO-BE

const x = 1,
  y = 2,
  z = 3
const pt = { x, y, z }

['A', 'B', 'C'].map((char, idx) => ({ char, idx }))
// [ { char: 'A', idx: 0 },  { char: 'B', idx: 1 }, { char: 'C', idx: 2 } ]
```

**객체 구조 분해(object de-structuring)**

```jsx
const props = obj.props
const a = props.a
const b = props.b

// 다음과 같이 줄여서 작성 할 수 있다.

const {props} = obj;
const {a, b} = props;

// 극단적으로는 다음처럼 줄일 수도 있다.
//참고로 a와 b는 변수로 선언되었지만 props는 변수 선언이 아니라는 것을 주의
const {props: {a, b}} = obj;

//구조 분해 문법에서는 기본값 지정 가능

//if 문을 사용한 단순한 방법
let {a} = obj.props;
if (a === undefined) a = 'default';

//개선된 기본값 지정
const {a = 'default'} = obj.props; 
```

**배열 구조 분해 문법** (튜플처럼 사용하는 배열에서 특히 유용)

```jsx
const point = [1, 2, 3]
const [x, y, z] = point
const [, a, b] = point // Ignore the first one

//함수 매개변수에도 구조 분해 문법 사용
const points = [
  [1, 2, 3],
  [4, 5, 6],
]
points.forEach(([x, y, z]) => console.log(x + y + z))
// Logs 6, 15
```

단순 객체 표현과 마찬가지로 객체 구조 분해를 사용하면 문법이 간결해지고 변수 사용 시 실수를 줄일 수 있기 때문에 적극적으로 사용하는 것이 좋다.

### 함수 매개변수 기본값 사용하기

자바스크립트에서 함수의 모든 매개변수는 선택적(생략 가능)이며, 매개변수 미지정 시 `undefined`로 간주

```jsx
function log2(a, b) {
  console.log(a, b)
}
log2() //logs undefined undefined

//모던 자바스크립트에서 매개변수 기본값 직접 지정

function parseNum(str, base=10) {
	return parseInt(str, base);
}
```

### 저수준 프로미스나 콜백 대신 async/await 사용하기

```jsx
//콜백 방식
function getJSON(url: string) {
  return fetch(url).then(response => response.json())
}
function getJSONCallback(url: string, cb: (result: unknown) => void) {
  // ...
}

// async/await 
async function getJSON(url: string) {
  const response = await fetch(url)
  return response.json()
}
```

콜백과 프로미스를 사용한 코드보다는 `async`와 `await` 작성한 코드가 훨신 깔끔하고 직관적입니다.

### 연관 배열에 객체 대신 Map과 Set 사용하기

```jsx
function countWords(text: string) {
	const counts: {[word: string]: number} = {};
	for (const word of text.split(/[\s,.]+/)) {
		counts[word] = 1 + (counts[word] || 0);
	}
	return counts;
}

console.log(countWords('Objects have a constructor'));

// logs
{
  Objects: 1,
  have: 1,
  a: 1,
  constructor: "1function Object() { [native code] }"
}
```

`constructor`의 초기값은 `undefined`가 아니라 `Object.prototype`에 있는 생성자 함수입니다.

원치 않는 값일 뿐만 아니라, 타입도 `number`가 아닌 `string`으로 이런 문제를 방지하려면  `Map`을 사용

```jsx
function countWrodsMap(text: string) {
  const counts = new Map<string, number>();
  for (const word of text.split(/[\s,.]+/)) {
    counts.set(word, 1 + (counts.get(word) || 0));
  }
  return counts;
}

// logs Map(4) {'Objects' => 1, 'have' => 1, 'a' => 1, 'constructor' => 1}
```

### 타입스크립트에 use strict 넣지 않기

ES5에서는 버그가 될 수 있는 코드 패턴에 오류를 표시해 주는 ‘엄격 모드(strict mode)’가 도입

코드의 제일 처음에 ‘use strict’를 넣으면 엄격 모드가 활성화

```jsx
'use strtict';
function foo() {
		x = 10;// strict : error , non-strict : Global Variable
}
```

타입스크립트에서 수행되는 안정성 검사(sanity check)가 엄격 모드보다 훨씬 더 엄격한 체크를 하기 때문에

타입스크립트 코드에서 ‘use strict’ 는 실제로 무의미

실제로는 타입스크립트 컴파일러가 생성하는 자바스크립트 코드에서 'use strict'가 추가된다. `alwaysStrict` 또는 `strict` 컴파일러 옵션을 설정하면, 타입스크립트는 업격모드로 코드를 파싱하고 생성되는 자바스크립트에 'use stict'를 추가한다.

즉 타입스크립트 코드에 'use strict'를 쓰지 않고, 대신 `alwaysStrict` 설정을 사용

**아이템58 요약**

- 타입스크립트 개발 환경은 모던 자바스크립트도 실행 할 수 있으므로 모던 자바스크립트의 최신 기능들을 적극적으로 사용하길 바란다. 코드 품질, 타입추론도 더 나아진다.
- 타입스크립트 개발 환경에서는 컴파일러와 언어 서비스를 통해 클래스, 구조 분해, async/await 같은 기능을 쉽게 배울 수 있다.
- 'use strict'는 타입스크립트 컴파일러 수준에서 사용되므로 코드에서 제가해야한다.
- TC39의 깃험 저장소와 타입스크립트 릴리즈 노트를 통해 최신 기능을 확인하자.

## 타입스크립트 도입 전에 @ts-check와 JSDoc으로 시험해보기

본격적으로 타입스크립트 전환 전 `@ts-check` 지시자 사용 시 타입 체커가 파일을 분석하고, 발견된 오류를 보고하도록 지시 그러나 `ts-check` 지시자는 매우 느슨한 수준으로 타입 체크를 수행하는데, 심지어 `noImplicitAny` 설정을 해제한 것보다 헐거운 체크를 수행한다는 점에 주의

```jsx
// @ts-check
const person = {first: 'Grace', last: 'Hopper'};
2 * person.first
 // ~~~~~~~~~~~~ The right-hand side of an arithmetic operation must be of type
 //              'any', 'number', 'bigint', or an enum type
```

### 선언되지 않은 전역 변수

```jsx
// @ts-check
console.log(user.firstName);
         // ~~~~ Cannot find name 'user'
```

user 선언을 위한 types.d.ts 파일 생성

```jsx
//types.d.ts

interface UserData {
  firstName: string;
  lastName: string;
}
declare let user: UserData;
```

타입 선언 파일을 만들면 오류가 해결되며, 파일을 찾지 못하는 경우 ‘트리플 슬래시’ 참조를 사용하여

명시적 임포트 가능

```jsx
// @ts-check
/// <reference path="./types.d.ts" />
console.log(user.firstName); // success
```

### 알 수 없는 라이브러리

서드파티 라이브러리 사용 시 해당 라이브러리의 타입 정보를 알아야 함.

```jsx
// @ts-check
$('#graph').style({ width: '100px', height: '100px' });
// Cannot find name '$'

$ npm install --save-dev @types/jquery

// @ts-check
$('#graph').style({'width': '100px', 'height': '100px'});
// ~~~~~ Property 'style' does not exist on type 'JQuery<HTMLElement>'

// .style 메서드를 .css 로 변경 시 통과
$('#graph').css({ width: '100px', height: '100px' });
```

### DOM 문제

HTMLInputElement 타입에는 value 속성이 있지만, document.getElementByID는 더 상위 개념인

 HTMLElement 타입을 반환하기 때문에 오류 발생

만약 #age 엘리먼트가 확실히 input 엘리먼트라는 것을 알고 있다면 타입 단언문 사용

그러나 js 파일이므로 ts 단언문을 사용 할 수 없어 jsdoc 를 사용하여 타입 단언문을 사용 가능하며

JSDOC 의 @type 구문 사용 시 타입을 감싸는 중괄호가 필요

### 부정확한 JSDoc

이미 JSDoc 스타일의 주석 사용 시 @ts-check 설정 순간부터 많은 오류가 발생 할 수 있으며

타입 정보를 추가해 나가면 된다.

**아이템59 요약**

- 파일 상단에 // @ts-check를 추가하면 자바스크립트에서도 타입 체크를 수행할 수 있다.
- 전역 선언과 서드파티 라이브러리 타입선언을 추가하는 방법을 익히자.
- JSDoc 주석을 잘 활용하면 자바스크립트 상태에서도 타입 단언과 타입 추론을 할 수 있다.
- JSDoc 주석은 중간 단계이기 때문에 너무 공들일 필요는 없습니다. 최종목표는 .ts로 된 타입스크립트 코드임을 명심하자.

## 아이템60 allowJs로 타입스크립트와 자바스크립트 같이 사용하기

```jsx
// tsconfig.json

"allowJS" : true // 해당 옵션 추가 시 js 파일과 ts 파일을 서로 임포트 할 수 있게 해줌.
```

번들러에 타입스크립트가 통합되어 있거나, 플러그인 방식으로 통합이 가능하다면 allowJs를 간단히 적용 가능

예를 들어, `npm install --save-dev tsify`를 실행하고 browserify를 사용하여 플러그인을 추가한다면 다음과 같다.

```jsx
$ browserify index.ts -p [ tsify --allowJs ] > bundle.js
```

유닛 테스트 도구에도 동일한 역할의 옵션 있음.

e.g. jest 사용 시 ts-jest 설치 후 jest.config.js 에 전달할 타입스크립트 소스 지정

```jsx
module.export = {
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
};
```

프레임워크 없이 빌드 체인 직접 구성했다면 outDir 옵션 사용

타입스크립트가 outDir에 지정된 디렉터리에 소스 디렉터리와 비슷한 구조로 자바스크립트 코드를 생성하게 되고, outDir로 지정된 디렉터리를 대상으로 기존 빌드 체인(import)를 실행하면 된다.

**아이템60 요약**

- 점진적 마이그레이션을 위해 자바스크립트와 타입스크립트를 동시에 사용 할 수 있게 `allowJs` 컴파일러 옵션을 사용합시다.
- 대규모 마이그레이션 작업을 시작하기 전에, 테스트와 빌드 체인에 타입스크립트를 적용한다.

## 아이템61 의존성 관계에 따라 모듈 단위로 전환하기

본격적으로 자바스크립트 코드를 타입스크립트로 전환하는 단계 알아보기

점진적 마이그레이션은 모듈 단위로 처리하는 것이 이상적이나

한 모듈을 선택 후 타입 정보 추가 시 해당 모듈이 의존하는 모듈에서 비롯되는 타입 오류가 발생하므로

작업 순서를 다른 모듈에 의존하지 않는 최하단 모듈부터 → 최상단에 있는 모듈을 마지막으로 완성

프로젝트 내 존재하는 모듈은 서드파티 라이브러리에 의존하지만 서드파티 라이브러리는 해당 모듈에

의존하지 않기 때문에 타입 정보를 가장 먼저 해결한다.

e.g. lodash 라이브러리 사용 시

`npm install —save-dev @types/lodash` 실행

마이그레이션시 타입 정보 추가만 하고, 리팩터링은 하면 안 된다.

당장의 목표는 코드 개선이 아니라 타입스크립트로 전환하는 것임을 명심하여

개선이 필요한 부분은 목록을 만들어서 나중에 리팩터링 할 수 있도록 관리

타입스크립트로 전환하며 발견하게 되는 일반적인 오류를 놓치지 않아야 하며,

타입 정보를 유지하기 위해 필요에 따라 JSDoc 주석을 활용 할 수 있음.

## 아이템62 마이그레이션의 완성을 위해 noImplicitAny 설정하기

프로젝트 전체를 타입스크립트로 전환했다면 `noImplicitAny` 를 설정해보자.

설정되지 않은 상태는 타입 선언에서 비롯되는 실제 오류가 숨어 있기 때문에 마이그레이션이

완료되었다고 할 수 없다.

타입 체크의 강도를 높이는 설정에는 여러가지가 있으며

`noImplicitAny`는 상당히 엄격한 설정이며, 최종적으로 가장 강력한 설정은 **strict: “true”** 이다.

- `noImplicitAny` 설정을 활성화하여 마이그레이션의 마지막 단계를 진행하자. `noIplicitAny` 설정이 없다면 타입 선언과 관련된 실제 오류가 드러나지 않습니다.
- `noIplicitAny`를 전면 적용하기 전에 로컬에서부터 타입 오류를 점진적으로 수정해야 합니다
    
    엄격한 타입 체크를 적용하기 전에 팀원들이 타입스크립트에 익숙해질 수 있도록 시간을 줍시다.

## 8장

# 개인 독후감

## 이은택

현재는 코드가 TS로 되어 있어서 실제로 JS 에서 TS로 마이그레이션 하는 경우를 크게 생각해보진 않았지만, 점진적으로 어떻게 마이그레이션을 하는지에 대해 나와있어서 추후 이러한 상황이 발생하면 단계별로 체크 리스트를 두고 진행해보며 좋을 것 같다고 느껴졌다.

## 김련호

마이그레이션을 계획하거나 경험해본적은 없지만, JS에 익숙한 개발자가 참고할 만한 좋은 내용이 많이 있었습니다. 특히 `JSDoc`, `@ts-check`을 통해서 미리 테스트해볼 수 있는 부분이 흥미로웠습니다. `JSDoc`을 이용해서 JS를 TS와 유사하게 타입 체크할 수 있는 것은 미리 마이그레이션을 대비해 볼 수 있는 좋은 방법인 것 같습니다. 그리고 마지막 아이템으로 언급되었던 `noImplicitAny` 옵션은 TS에서 필수이고 TS 그 자체임을 다시 한번 복기 할 수 있었습니다.

## 강현구

평소에 사용하던 문법에 대한 명칭(ex : 단축 객체 표현, 객체 구조 분해 ,..)들을 알게 되었고 자세하진 않지만 자바스크립트가 어떻게 발전되어 왔는지 알 수 있었고
당장 진행 할 일은 없지만 8장의 내용을 토대로 마이그레이션의 기회가 주어지면 주도적으로 해볼 수 있을 것 같다.
추후 ts 컴파일러로 변환한 es3(1999) ~ 최신 ES2021 까지의 js 코드를 분석해보면 재미있을 것 같다는 생각이 들었다.

## 김지섭

# 아젠다
