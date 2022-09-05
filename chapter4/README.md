# 요약

타입을 잘 설계하면 코드는 직관적으로 작성할 수 있습니다.

그러나 타입 설계가 엉망이라면 어떠한 기억이나 문서도 도움이 되지 못합니다.

## 아이템 28 유효한 상태만 표현하는 타입을 지향하기

### 효과적으로 타입을 설계하려면, 유효한 상태만 표현할 수 있는 타입을 만들어내는 것이 가장 중요합니다.

- 유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 혼란을 초래하기 쉽고 오류를 유발하게 됩니다.
- 유효한 상태만 표현하는 타입을 지향해야 합니다. 코드가 길어지거나 표현하기 어렵지만 결국 시간은 절약하고 고통을 줄일 수 있습니다

### 잘못된 타입설계

```typescript
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}

function renderPage(state: State) {
  if (state.error) {
    return "Error! ~~";
  } else if (state.isLoading) {
    return "Loading... ~~";
  }
  return `<h1>{currentPage}<h1>n${state.pageText}`;
}

async function changePage(state: State, newPage: string) {
  state.isLoading = true;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error("Unable to ~~");
    }
    const text = await response.text();
    state.isLoading = false;
    state.pageText = text;
  } catch (e) {
    state.error = "" + e;
  }
}
```

위 코드의 문제점

- 오류가 발생했을 때 state.isLoading을 false로 설정하는 로직이 빠져있습니다.
- state.error를 초기화하지 않았기 때문에, 페이지 전환중에 로딩 메시지 대신 과거의 오류 메시지를 보여주게 됩니다.
- 페이지 로딩중에 사용자가 페이지를 바꿔버리면 어떤 일이 벌어질지 예상하기 어렵습니다. 새 페이지에 오류가 뜨거나, 응답이 오는 순서에 따라 두 번째 페이지가 아닌 첫 번째 페이지로 전환될 수 있습니다.

문제는 상태 값의 두 가지 속성이 동시에 정보가 부족하거나, 두가지 속성이 충돌할 수 있다는 것입니다.

State 타입은 isLoading이 true 이면서 동시에 error 값이 설정되는 무효한 상태를 허용합니다.

### 개선된 타입 설계

```typescript
interface RequestPending {
  state: "pending";
}

interface RequestError {
  state: "error";
  error: string;
}

interface RequestSuccess {
  state: "ok";
  pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: {
    [page: string]: RequestState;
  };
}

function renderPage(state: State) {
  const { currentPage } = state;
  const requestState = state.request[currentPage];

  switch (requestState.state) {
    case "pending":
      return "Loading ~~";
    case "error":
      return "Error!! ~~";
    case "ok":
      return `<h1>{currentPage}<h1>\n${state.pageText}`;
  }
}

async function changePage(state: State, newPage: string) {
  state.requests[newPage] = { state: "pending" };
  state.currentPage = newPage;

  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error("Unable to ~~");
    }
    const text = await response.text();
    state.requests[newPage] = { state: "ok", pageText };
  } catch (e) {
    state.requests[newPage] = { state: "error", error: "" + e };
  }
}
```

네트워크 요청 과정 각각의 상태를 명시적으로 모델링하는 태그된 유니온이 사용되었습니다. 코드의 길이가 길어지긴 했지만, 유효한 상태만을 나타내도록 크게 개선되었습니다.

renderPage와 changePage의 모호함은 완전히 사라졌습니다. 현재 페이지가 무엇인지 명확하며, 모든 요청은 정확히 하나의 상태로 맞아 떨어집니다.

## 아이템 29 사용할 때는 너그럽게, 생성할 때는 엄격하게

### 함수의 매개변수는 타입의 범위가 넓어도 되지만, 결과를 반환할 때는 일반적으로 타입의 범위가 더 구체적이어야 합니다.

- 보통 매개변수 타입은 반환 타입에 비해 범위가 넓은 경향이 있습니다. 선택적 - 속성과 유니온 타입은 반환타입보다 매개변수 타입에 더 일반적입니다.
- 매개변수와 반환 타입의 재사용을 위해서 기본 형태(반환 타입)와 느슨한 형태(매개변수 타입)을 도입하는 것이 좋습니다.

### 잘못된 타입설계

```typescript
interface CameraOptions {
    center?: LngLat;
    zoom?: number;
    bearing?: number;
    pitch?: number;
}

type LngLat =
    { lng: number, lat: number;} |
    { lon: number, lat: number;} |
    [number, number];

type LngLatBounds =
    { northest: LngLat, southest: LngLat} |
    [LngLat, LngLat] |
    [number, number, number, number]

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;

function focusOnFeature(f: Feature) {

    const bounds = calculateBoundingBox(f);
    const camera = viewportForBounds(bounds);
    setCamera(camera);

    const {center: {lat, lng}, zoom} = camera;
                 // ~~        ... 형식에 'lat' 속성이 없습니다.
                 // ~~        ... 형식에 'lng' 속성이 없습니다.
    zoom; // 타입이 number | undefined
    ....
}
```

위 코드의 문제점

- lat 과 lng 속성이 없고 zoom 속성만 존재
- zoom의 타입이 number | undefined로 추론된다.

문제는 viewportForBounds의 타입 선언이 사용될 때 뿐만 아니라 만들어질 때에도 너무 자유롭다는 것 입니다.

camera 값을 안전한 타입으로 사용하는 유일한 방법은 유니온 타입의 각 요소별로 코드를 분기하는 것 입니다.

수많은 선택적 속성을 가지는 반환 타입과 유니온 타입은 viewportForBounds함수를 사용하기 어렵게 만듭니다.

### 개선된 타입설계

```typescript
interface LngLat { lng: number; lat: number;};
type LngLatLike =
    LngLat |
    {lon: number, lat: number} |
    [number, number]

interface Camera{
    center: LngLat;
    zoom: number;
    bearing: number;
    pitch: number;
}
interface CameraOptions extends Omit<Partial<Camera>, center>{
    center?: LngLatLike;
}

interface CameraOptioins {
    center?: LngLatLike;
    zoom?: number;
    bearing?: number;
    pitch?: number;
}

type LngLatBounds =
    { northeast: LngLatLike, southwest: LngLatLike } |
    [LngLatLike, LngLatLike] |
    [number, number, number, number]

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LatLngBounds): Camera;


function focusOnFeature(f: Feature) {

    const bounds = calculateBoundingBox(f);
    const camera = viewportForBounds(bounds);
    setCamera(camera);

    const {center: {lat, lng}, zoom} = camera;  // 정상
    zoom;                                       // 타입이 number
    ....
}
```

Camera가 너무 엄격하므로 조건을 완화하여 느슨한 CameraOptions 타입으로 만들었습니다.

이제 viewportForBounds 함수를 사용하기 훨씬 쉬워졌습니다.

## 아이템 30 문서에 타입정보를 쓰지 않기

### 함수의 입력과 출력의 타입을 코드로 표현하는 것이 주석보다 더 나은 방법이라는 것은 자명합니다.

- 주석과 변수명에 타입 정보를 적는 것은 피해야 합니다. 타입 선언이 중복되는 것으로 끝나면 다행이지만 최악의 경우는 타입 정보에 모순이 발생하게 됩니다.
- 타입이 명확하지 않은 경우는 변수명에 단위 정보를 포함하는 것을 고려하는 것이 좋습니다.

### 잘못된 타입설계

```typescript
/**
 * 전경색(foreground) 문자열을 반환합니다.
 * 0개 또는 1개의 매개변수를 받습니다.
 * 매개변수가 없을 때는 표준 전경색을 반환합니다.
 * 매개변수가 있을 때는 특정 페이지의 전경색을 반환합니다.
 */
function getForegroundColor(page?: string) {
  return page === "login" ? { r: 127, g: 127, b: 127 } : { r: 0, g: 0, b: 0 };
}

/** nums를 변경하지 않습니다 */
function sort(nums: number[]) {
  /* ... */
}
```

위 코드의 문제점

- 코드의 주석 정보가 맞지 않습니다

  함수가 string 형태의 색깔을 반환한다고 적혀있지만, 실제로는 {r, g, b} 객체를 반환합니다.
  주석에는 함수가 0개 혹은 1개의 매개변수를 받는다고 설명하고 있지만, 타입 시그니처만 보아도 명확하게 알 수 있는 정보입니다.
  불필요하게 장황합니다. 함수 선언과 구현체보다 주석이 더 깁니다.

- 값을 변경하지 않는다고 설명하는 주석도 좋지 않습니다.

누군가 강제하지 않는 이상 주석은 코드와 동기화되지 않습니다. 그러나 타입 구문은 타입스크립트의 타입 체커가 타입 정보를 동기화하도록 강제합니다.
주석 대신 타입 정보를 작성한다면 코드가 변경된다 하더라도 정보가 정확히 동기화됩니다.

### 개선된 타입설계

```typescript
/** 애플리케이션 또는 특정 페이지의 전경색을 가져옵니다 */
function getForegroundColor(page?: string): Color {
  // ...
}

function sort(nums: readonly number[]) {
  /* ... */
}
```

특정 매개변수를 설명하고 싶다면 JSDoc의 @param 구문을 사용하면 됩니다.

값을 변경하지 않는다는 주석 대신에 readonly로 선언하여 타입스크립트가 규칙을 강제할 수 있게합니다.

주석에 적용한 규칙은 변수명에도 그대로 적용할 수 있습니다. 변수명에 타입 정보를 넣지 않도록 합니다.

예를 들어 변수명을 ageNum으로 하는 것 보다 age를 변수명으로 하고, 그 타입이 number임을 명시하는게 좋습니다.

그러나 단위가 있는 숫자들은 예외입니다. 단위가 무엇인지 확실하지 않다면 변수명 또는 속성 이름에 단위를 포함할 수 있습니다.

예를 들어 timeMS는 time보다 훨씬 명확하고 temparatureC는 temparature보다 훨씬 명확합니다.

## 아이템 31 타입 주변에 null 값 배치하기

### 값이 전부 null이거나 전부 null이 아닌 경우로 분명히 구분된다면 값이 섞여 있을 때보다 다루기 쉽습니다.

- 한 값의 null 여부가 다른 값의 null 여부에 암시적으로 관련되도록 설계하면 안 됩니다.
- API 작성 시에는 반환 타입을 큰 객체로 만들고 반환 타입 전체가 null이거나 null이 아니게 만들어야 합니다. 사람과 타입 체커 모두에게 명료한 코드가 될 것 입니다.
- 클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하여 null이 존재하지 않도록 하는 것이 좋습니다.
- strictNullChecks를 설정하면 코드에 많은 오류가 표시되겠지만, null 값과 관련된 문제점을 찾아낼 수 있기 때문에 반드시 필요합니다.

### 잘못된 타입설계

```typescript
function extent(nums: number[]) {
  let min, max;

  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
      // ~~ 'number | undefined' 형식의 인수는 'number' 형식의 매개변수에 할당될 수 없습니다.
    }
  }

  return [min, max];
}

const [min, max] = extent([0, 1, 2]);
const span = max - min;
// ~~    ~~ 개체가 'undefined'인 것 같습니다.
```

위 코드의 문제점

- 최솟값이나 최댓값이 0인 경우, 값이 덧씌워져 버립니다. 예를 들어 extent([0, 1, 2])의 결과는 [0, 2]가 아니라 [1, 2]가 됩니다.
- nums 배열이 비어 있다면, 함수는 [undefined, undefined]를 반환합니다. undefined를 포함하는 객체는 다루기 어렵고 절대 권장하지 않습니다.

이 코드는 타입 체커를 통과하고(strictNullCheck 없이), 반환타입은 number[]로 추론됩니다.

strictNullChecks 설정을 켜면 앞의 두 가지 문제점이 드러납니다.

extent의 반환 타입이 (number | undefined)[]로 추론되어서 호출하는 곳마다 타입 오류의 형태로 나타납니다.

min과 max를 한 객체 안에 넣고 null 이거나 null이 아니게 하면 됩니다.

### 개선된 타입설계

```typescript
function extent(nums: number[]) {
  let result: [number, number] | null = null;

  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, min), Math.max(num, max)];
    }
  }

  return result;
}

const [min, max] = extent([0, 1, 2])!;
const span = max - min;

const range = extent([0, 1, 2]);
if (range) {
  const [min, max] = range;
  const span = max - min;
}
```

extent의 결과값으로 단일 객체를 사용함으로써 설계를 개선했고, 타입스크립트가 null 값 사이의 관계를 이해할 수 있도록 했으며 버그도 제거했습니다.

if(!result)는 이제 제대로 동작합니다.

null과 null이 아닌 값을 섞어서 사용하면 클래스에서도 문제가 생깁니다.

### 잘못된 타입설계

```typescript
class UserPosts {
  user: UserInfo | null;
  posts: Post[] | null;

  constructor() {
    this.user = null;
    this.posts = null;
  }

  async init(userId: string) {
    return Promise.all([
      async () => (this.user = await fetchUser(userId)),
      async () => (this.posts = await fetchPostsForUser(userId)),
    ]);
  }

  getUserName() {
    // ...
  }
}
```

위 코드의 문제점

- 두 번의 네트워크 요청이 로드되는 동안 user와 posts 속성은 null 상태입니다. 어떤 시점에는 둘 다 null 이거나, 둘 중 하나만 null 이거나, 둘 다 null이 아닐 것 입니다.

속성값의 불확실성이 클래스의 모든 메서드에 나쁜 영향을 미칩니다.

결국 null 체크가 난무하고 버그를 양산하게 됩니다.

필요한 데이터가 모두 준비된 후에 클래스를 만들도록 해야합니다.

### 개선된 타입설계

```typescript
class UserPosts {
  user: UserInfo;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user;
    this.posts = posts;
  }

  static async init(userId: string) {
    const [user, posts] = await Promise.all([
      fetchUser(userId),
      fetchPostsForUser(userId),
    ]);

    return new User(user, posts);
  }

  getUserName() {
    return this.user.name;
  }
}
```

UserPosts 클래스는 완전히 null이 아니게 되었고, 메서드를 작성하기 쉬워졌습니다.

물론 이 경우에도 데이터가 부분적으로 준비되었을 때 작업을 시작해야 한다면, null과 null이 아닌 경우의 상태를 다루어야 합니다.

## 아이템 32 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

### 유니온 타입의 속성을 가지는 인터페이스를 작성중이라면, 혹시 인터페이스의 유니온 타입을 사용하는 개 더 알맞지는 않을지 검토해 봐야 합니다.

- 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하므로 주의해야 합니다.
- 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋습니다.
- 타입스크립트가 제어흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야합니다. 태그된 유니온은 타입스크립트와 매우 잘 맞기 떄문에 자주 볼 수 있는 패턴입니다.

### 잘못된 타입설계

```typescript
interface Layer {
  type: "fill" | "line" | "point";
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

위 코드의 문제점

- type이 'fill' 타입이면서 paint 속성이 LinePaint 타입인 조합이 나올 수 있다.

더 나은 방법으로 모델링 하기 위해서는 각각 타입의 계층을 분리된 인터페이스로 둬야합니다.

### 개선된 타입설계

```typescript
interface FillLayer {
  type: "fill";
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayout {
  type: "line";
  layout: LineLayout;
  paint: LineLayout;
}
interface PointLayer {
  type: "point";
  layout: PointLayer;
  paint: PointPaint;
}
```

이런 형태로 Layer를 정의하면 layout과 paint 속성이 잘못된 조합으로 섞이는 경우를 방지할 수 있습니다.

type속성은 '태그'이며 런타임에 어떤 타입의 Layer가 사용되는지 판단하는데 쓰입니다.

태그된 유니온은 타입스크립트 타입 체커와 잘 맞기 때문에 타입스크립트 코드 어디에서나 찾을 수 있습니다.

어떤 데이터 타입을 태그된 유니온으로 표현할 수 있다면 보통은 그렇게 하는 것이 좋습니다.

또는 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 undefined인 경우도 태그된 유니온 패턴과 잘 맞습니다.

### 잘못된 타입설계

```typescript
interface Person {
  name: string;
  // 다음은 둘 다 동시에 있거나 동시에 없습니다.
  placeOfBirth?: string;
  dateOfBirth?: Date;
}
```

위 코드의 문제점

- 타입 정보를 담고 있는 주석은 문제가 될 소지가 매우 높습니다. placeOfBirth와 dateOfBirth 필드는 실제로 관련되어 있지만, 타입 정보에는 어떠한 관계도 표현되지 않았습니다.

### 개선된 타입설계

```typescript
/** 두 개의 속성을 하나의 객체로 모으는 것이 더 나은 설계입니다.*/
interface Person {
  name: string;
  birth?: {
    place: string;
    date: Date;
  };
}

/** 타입의 구조를 손 댈 수 없는 상황이면, 앞서 다룬 인터페이스의 유니온을 사용해서 속성 사이의 관계를 모델링할 수 있습니다. */
interface Name {
  name: string;
}

interface PersonWithBirth extends Name {
  placeOfBirth: string;
  dateOfBirth: Date;
}

type Person = Name | PersonWithBirth;
```

앞의 두 경우 모두 타입 정의를 통해 속성간의 관계를 더 명확하게 만들 수 있습니다.

## 아이템 33 string 타입보다 더 구체적인 타입 사용하기

### string 타입으로 변수를 선언하려 한다면, 혹시 그보다 더 좁은 타입이 적절하지는 않을지 검토해 보아야 합니다.

- 문자열을 남발하여 선언된 코드를 피합시다. 모든 문자열을 할당할 수 있는 string 타입보다는 더 구체적인 타입을 사용하는 것이 좋습니다.
- 변수의 범위를 보다 정확하게 표현하고 싶다면 string 타입보다는 문자열 리터럴 타입의 유니온을 사용하면 됩니다. 타입 체크를 더 엄격히 할 수 있고 생산성을 향상시킬 수 있습니다.
- 객체의 속성 이름을 함수 매개변수로 받을 때는 string 보다 keyof T를 사용하는 것이 좋습니다.

### 잘못된 타입설계

```typescript
interface Album {
  artist: string;
  title: string;
  releaseDate: string; // YYYY-MM-DD
  recordingType: string; // 예를들어, "live" 혹은 "studio"
}

const kindOfBlue: Album = {
  artist: "Miles Davis",
  title: "kind of blue",
  releaseDate: "August 17th, 1959", // 날짜 형식이 다릅니다.
  recordingType: "Studio", // 오타 (대문자 S)
}; // 정상
```

위 코드의 문제점

- releaseDate 필드의 값은 주석에 설명된 형식과 다르다.
- recordType 필드의 값 'Studio'는 소문자대신 대문자가 쓰였다.
- recordRelease 함수의 호출에서 매개변수들의 순서가 바뀌었지만, 둘 다 문자열이기 때문에 타입체커가 정상으로 인식합니다.

releaseDate 필드는 Date 객체를 사용해서 날짜 형식으로만 제한하는 것이 좋습니다.

recordType 필드는 'live'와 'studio' 단 두 개의 값으로 유니온 타입을 정의할 수 있습니다.

### 개선된 타입설계

```typescript
type RecordingType = "studio" | "live";

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}

const kindOfBlue: Album = {
  artist: "Miles Davis",
  title: "kind of blue",
  releaseDate: new Date("1959-08-17"),
  recordingType: "Studio",
  //  ~~~~~~~~~~~~~ 'Studio' 형식은 'RecordingType'형식에 할당할 수 없습니다.
};
```

`이러한 방식에는 세가지 장점이 더 있습니다.`

- 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지됩니다.
- 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있습니다.
- keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해집니다.

### keyof 연산자

```typescript
/** 어떤 배열에서 한 필드의 값만 추출하는 함수 */
function pluck(records, key) {
  return records.map((r) => r[key]);
}

/** 타입 체크가 되긴하지만, any 타입이 있어서 정밀하지 못함 */
function pluck(records: any[], key: string): any[] {
  return records.map((r) => r[key]);
}

/**
 * 제너릭 타입을 도입해서 any를 없앰
 * key 타입이 string이지만 범위가 너무 넓어서 오류가 발생
 */
function pluck<T>(records: T[], key: string): any[] {
  return records.map((r) => r[key]);
}

/**
 * keyof 연산자를 이용해 string의 범위를 좁힘
 * 타입 체커를 통과할 뿐 아니라, 반환 타입을 추론할 수 있음
 * key의 값으로 하나의 문자열을 넣게 되면, 범위가 너무 넓음
 */
function pluck<T>(records: T[], key: keyof T): T[keyof T][] {
  return records.map((r) => r[key]);
}
const releaseDates = pluck(albums, "releaseDate"); // 타입이 (string | Date)[]

/**
 * 범위를 더 좁히기 위해서, keyof T의 부분집합으로 두번째 제너릭 매개변수를 도입
 */
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map((r) => r[key]);
}

pluck(albums, "releaseDate"); // 타입이 Date[]
pluck(albums, "artist"); // 타입이 string[]
pluck(albums, "recordingType"); // 타입이 RecordType[]
pluck(albums, "recordingDate"); // ~~~~~~~~~~~~~ "recordingDate" 형식의 인수는 ... 형식의 매개변수에 할당될 수 없습니다.
```

## 아이템 34 부정확한 타입보다는 미완성 타입을 사용하기

### 타입이 구체적으로 정제된다고 해서 정확도가 무조건 올라가지는 않습니다.

- 타입 안정성에서 불쾌한 골짜기는 피해야합니다. 타입이 없는 것보다 잘못된 게 더 나쁩니다.
- 정확하게 타입을 모델링할 수 없다면, 부정확하게 모델링하지 말아야합니다. 또한 any와 unknown를 구별해야합니다.
- 타입 정보를 구체적으로 만들 수록 오류 메시지와 자동 완성 기능에 주의를 기울여야 합니다. 정확도뿐만 아니라 개발 경험과도 관련됩니다.

### 잘못된 타입설계

```typescript
/**
 * 책 184p
 * 1. 모두 허용
 * 2. 문자열, 숫자, 배열 허용
 * 3. 문자열, 숫자, 알려진 함수 이름으로 시작하는 배열 허용
 * 4. 각 함수가 받는 매개변수의 개수가 정확한지 확인
 * 5. 각 함수가 받는 매개변수의 타입이 정확한지 확인
 */

type Expression1 = any;
type Expression2 = number | string | any[];

const tests: Expression2[] = [
  10,
  "red",
  true, // (오류) 'true' 형식은 'Expression2' 형식에 할당할 수 없습니다.
  ["case", [">", 20, 10], "red", "blue", "green"], // 값이 너무 많습니다.
  ["**", 2, 31], // **은 함수가 아닙니다.
  ["rgb", 255, 238, 64],
  ["rgb", 255, 0, 127, 0], // 값이 너무 많습니다.
];

type FnName = "+" | "-" | "*" | "/" | ">" | "<" | "case" | "rgb";
type CallExpression = [FnName, ...any[]];
type Expression3 = number | string | CallExpression;

const tests: Expression3[] = [
  10,
  "red",
  true, // (오류) 'true' 형식은 'Expression3' 형식에 할당할 수 없습니다.
  ["case", [">", 20, 10], "red", "blue", "green"], // 값이 너무 많습니다.
  ["**", 2, 31], // (오류) '**' 형식은 'FnName'형식에 할당할 수 없습니다.
  ["rgb", 255, 238, 64],
  ["rgb", 255, 0, 127, 0], // 값이 너무 많습니다.
];

type CallExpression = MathCall | CaseCall | RGBCall;
type Expression4 = number | string | CallExpression;
type MathCall = {
  0: "+" | "-" | "*" | "/" | ">" | "<";
  1: Expression4;
  2: Expression4;
  length: 3;
};
type CaseCall = {
  0: "case";
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4 | 6 | 8 | 10 | 12; // 등등
};
type RGBCall = {
  0: "rgb";
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4;
};

/** 오류가 발생하면 엉뚱한 에러 메시지를 발생 */
const tests: Expression4[] = [
  10,
  "red",
  true, // (오류) 'true' 형식은 'Expression4' 형식에 할당할 수 없습니다.
  ["case", [">", 20, 10], "red", "blue", "green"], // (오류) '['case',['>',...],...]' 형식은 string 형식에 할당할 수 없습니다.
  ["**", 2, 31], // (오류) 'number' 형식은 'string'형식에 할당할 수 없습니다.
  ["rgb", 255, 238, 64],
  ["rgb", 255, 0, 127, 0], // (오류) 'number'형식은 'string' 형식에 할당할 수 업습니다.
];
```

## 아이템 35 데이터가 아닌, API와 명세를 보고 타입만들기

### 명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있도록 도와줍니다. 반면에 예시 데이터를 참고해 타입을 생성하면 눈앞에 있는 데이터들만 고려하게 되므로 예기치 않은 곳에서 오류가 발생할 수 있습니다.

- 코드 구석 구석까지 타입 안정성을 얻기 위해 API 또는 데이터 형식에 대한 타입 생성을 고려해야 합니다.
- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세로부터 코드를 생성하는 것이 좋습니다.

## 아이템 36 해당 분야의 용어로 타입 이름 짓기

엄선된 타입, 속성, 변수의 이름은 의도를 명확히 하고 코드의 추상화 수준을 높혀줍니다.

- 가독성을 높이고, 추상화 수준을 올리기 위해서 해당 분야의 용어를 사용해야합니다.
- 같은 의미에 다른 이름을 붙이면 안 됩니다. 특별한 의미가 있을 때만 용어를 구분해야합니다.

타입, 속성, 변수에 이름을 붙일 때 명심해야 할 세 가지 규칙이 있습니다.

- 동일한 의미를 나타낼 때는 같은 용어를 사용해야합니다. 동의어를 사용하면 글을 읽을 때는 좋지만, 코드에서는 좋지 않습니다.
- data, info, thing, item 같은 모호하고 의미없는 이름은 피해야합니다.
- 이름을 지을 때는 포함된 내용이나 계산방식이 아니라 데이터가 무엇인지 고려해야합니다.

## 아이템 37 공식 명칭에는 상표를 붙이기

타입스크립트의 구조적 타이핑(덕 타이핑) 특성에 따른 예측 불가능한 결과 도출 방지 기법

Nominal Typing: https://michalzalecki.com/nominal-typing-in-typescript/

```typescript
interface Person {
  name: string
  age: number
}

const printPerson = (person: Person) => {
  console.log(person)
}

const jack = { name: 'Jack', age:  30 }
const jaden = { name: 'Jaden', age: 35, rich: true }

printPerson(jack)   // OK
printPerson(jaden)  // OK

---

interface Person {
  _brand: 'PERSON'
  name: string
  age: number
}

const printPerson = (person: Person) => {
  console.log(person)
}

const createPerson = (name: string, age: number): Person => {
  return { name, age, _brand: 'PERSON' }
}

const jack = createPerson('Jack', 30)
const jaden = { name: 'Jaden', age: 35, rich: true }

printPerson(jack)   // OK
printPerson(jaden)  // ERROR
```

# 개인 독후감

## 이은택


## 김련호

전체적으로 가벼은 팁에 관한 내용이였던 것 같습니다. 그 중에 아이템28, 33은 바로 실무에 적용해볼 수 있는 좋은 내용이였습니다. 특히 enum을 선언하여 많이 사용하고 있었는데, 굳이 enum이 반드시 필요한 사항이 아니라면 제한된 타입의 유니온 타입으로 사용하는 것이 더 간결하고 가벼워서, enum 타입의 사용을 많이 줄일 수 있을 것 같습니다.

## 강현구
타입을 어떤식으로 설계하면 좋을지 다양한 예제를 통해 꿀팁을 얻어가는 장 이었습니다. 특히 'string' 타입을 광범위하게 사용하고 있었는데
요번 장을 보면서 좀 더 구체적인 타입으로 설계 및 사용하며 불 필요한 타입 추론 등의 비용을 줄일 수 있겠다 생각했습니다.

## 김지섭
함수를 작성함에 있어 매개변수의 타입은 범위가 넓어질 수 있으나 함수가 반환하는 값은 매개변수의 타입에 비해서는 최대한 자세하고 명료하게 적어주는 것이 더 좋은 코드임을 알게되었다. 반환타입을 자세하고 명료하게 쓰기 위해 Omit, Partial 등의 유틸리티 타입을 이용할 수 있다. 또한 string 타입보다는 더 구체적인 타입을 사용하는 것이 더 좋은 코드임을 알게되었습니다. 데이터를 보지 않고 API 명세를 보고 타입을 생성하면 좋다는 것이 인상적이었습니다.

# 아젠다

* enum 이야기 조금 ..
* Item 29에 엄격하게 작성한 타입이 오히려 가독성이 안좋지 않을까

enum 관련 이야기

김련호 : typescript 에만 있는 기능들 중(e.g. enum, interface)에 해당 문법들이 자바스크립트 변환 과정 후
로직을 확인해보면 즉시 실행 함수로 되어있는데 enum 은 꼭 필요하지 않으면 안 써도 되겠다고 생각했다.

- 추가 공부 자료

```typescript
export enum MOBILE_OS {
  IOS,
  ANDROID
}

// 문자열을 할당한 경우
export enum MOBILE_OS {
  IOS = 'iOS',
  ANDROID = 'Android'
}
```

위 코드를 트랜스파일 시 아래와 같은 Javascript 코드가 됩니다.

```typescript
export var MOBILE_OS;
(function (MOBILE_OS) {
    MOBILE_OS[MOBILE_OS["IOS"] = 0] = "IOS";
    MOBILE_OS[MOBILE_OS["ANDROID"] = 1] = "ANDROID";
})(MOBILE_OS || (MOBILE_OS = {}));

// 문자열을 할당한 경우
export var MOBILE_OS;
(function (MOBILE_OS) {
    MOBILE_OS["IOS"] = "iOS";
    MOBILE_OS["ANDROID"] = "Android";
})(MOBILE_OS || (MOBILE_OS = {}));
```
JavaScript에 존재하지 않는 것을 구현하기 위해 TypeScript 컴파일러는 IIFE(즉시 실행 함수)를 포함한 코드를 생성합니다. 
그런데 Rollup과 같은 번들러는 IIFE를 '사용하지 않는 코드'라고 판단할 수 없어서 Tree-shaking이 되지 않습니다. 결국 MOBILE_OS를 import하고 실제로는 사용하지 않더라도 최종 번들에는 포함되는 것입니다.

##Tree-shaking은 무엇인가요?

Tree-shaking이란 간단하게 말해 사용하지 않는 코드를 삭제하는 기능을 말합니다. 나무를 흔들면 죽은 잎사귀들이 떨어지는 모습에 착안해 Tree-shaking이라고 부릅니다. Tree-shaking을 통해 export했지만 아무 데서도 import하지 않은 모듈이나 사용하지 않는 코드를 삭제해서 번들 크기를 줄여 페이지가 표시되는 시간을 단축할 수 있습니다. 
