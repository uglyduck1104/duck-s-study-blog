---
title: 타입 추론 - Item 24 ~ 26
date: '2022-10-23'
tags: ['typescript']
draft: false
summary: 일관성 있는 별칭 사용하기, 비동기 코드에는 콜백 대신 async 함수 사용하기, 타입 추론에 문맥이 어떻게 사용되는지 이해하기
layout: PostSimple
authors: ['default']
---

# 타입 추론 - Item 24 ~ 26

## Item 24) 일관성 있는 별칭 사용하기

```tsx
const borough = { name: 'Brooklyn', location: [40.688, -73.979] }
const loc = borough.location
```

- `borough.location` 배열에 `loc`이라는 별칭을 만들고 별칭의 값을 변경하면 원래 속성값에서도 변경됨

  ```
  loc[0] = 0;
  borough.location
  // [0, -73.979]
  ```

- 이와 같이 별칭을 남발해서 사용하면 제어 흐름 분석이 어려울 수 있으므로, 무분별한 별칭 사용을 자제해야 함

```tsx
// 다각형을 표현하는 자료구조
interface coordinate {
  x: number
  y: number
}

interface BoundingBox {
  x: [number, number]
  y: [number, number]
}

interface Polygon {
  exterior: Coordinate[]
  holes: Coordinate[][]
  bbox?: BoundingBox
}
```

- 다각형의 기하학적 구조는 exterior와 holes 속성으로 정의되며, bbox는 필수가 아닌 최적화 속성임
- bbox 속성을 통해 어떤 점이 다각형에 포함되는지 빠르게 체크할 수 있음

### 문제 파악

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  if (polygon.bbox) {
    if (
      pt.x < polygon.bbox.x[0] ||
      pt.x > polygon.bbox.x[1] ||
      pt.y < polygon.bbox.y[0] ||
      pt.y > polygon.bbox.y[1]
    ) {
      return false
    }
  }

  // ...
}
```

- 해당 코드는 잘 동작하지만 반복되는 부분이 존재하므로 중복을 줄이고 임시 변수를 뽑아낼 수 있음

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox
  if (polygon.bbox) {
    if (
      pt.x < box.x[0] ||
      pt.x > box.x[1] ||
      // 객체가 'undefined'일 수 있습니다.
      pt.y < box.y[0] ||
      pt.y > box.y[1]
    ) {
      // 객체가 'undefined'일 수 있습니다.
      return false
    }
  }
  //...
}
```

- `polygon.bbox`를 별도의 `box`라는 별칭을 만들고 첫 번째 예제에서 잘 동작했던 제어 흐름 분석을 방해함

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  polygon.bbox // 타입이 BoundingBox | undefined
  const box = polygon.bbox
  box // 타입이 BoundingBox | undefined
  if (polygon.bbox) {
    polygon.bbox // 타입이 BoundingBox
    box // 타입이 BoundingBox | undefined
  }
}
```

- 속성 체크는 `polygon.bbox`의 타입을 정제했으나 `box`는 그렇지 않았기 때문에 오류가 발생함
- 이러한 오류는 `별칭은 일관성 있게 사용한다`는 기본 원칙을 지키면 방지할 수 있음

### 해결

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox
  if (box) {
    if (pt.x < box.x[0] || pt.x > box.x[1] || pt.y < box.y[0] || pt.y > box.y[1]) {
      // 정상
      return false
    }
  }
  //...
}
```

- 속성 체크에 `box`를 사용하여 타입 체커의 문제는 해결되었지만, `box`와 `bbox`가 같은 값인데도 불구하고 다른 이름을 사용하는 문제점이 있음

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const { bbox } = polygon
  if (bbox) {
    const { x, y } = bbox
    if (pt.x < box.x[0] || pt.x > box.x[1] || pt.y < box.y[0] || pt.y > box.y[1]) {
      return false
    }
  }
  //...
}
```

- 객체 비구조화를 통해 간결한 문법으로 일관된 이름을 사용할 수 있음
  - 전체 `bbox` 속성이 아니라 `x, y`가 선택적 속성일 경우에 속성체크가 더 필요하므로 타입의 경계에 `null` 값을 추가하는게 좋음

### 런타임에서의 혼동

```tsx
const { bbox } = polygon
if (!bbox) {
  calculatePolygondBbox(polygon) // polygon.bbox가 채워짐
  // polygon.bbox와 bbox는 다른 값을 참조함
}
```

- 별칭은 타입 체커뿐만 아니라 런타임에도 혼동을 야기함
- 타입스크립트의 제어 흐름 분석은 지역 변수에서는 꽤 잘 동작하지만 객체 속성에서는 주의해야 함

```tsx
function fn(p: Polygon) {
  /*...*/
}

polygon.bbox // 타입이 BoundingBox | undefined
if (polygon.bbox) {
  polygon.bbox // 타입이 BoundingBox
  fn(polygon)
  polygon.bbox // 타입이 BoundingBox
}
```

- `fn(polygon)` 호출은 `polygon.bbox`를 제거할 가능성이 있으므로 타입을 `BoundingBox | undefined`로 되돌리는 것이 안전함
  - 하지만 함수를 호출할때마다 속성 체크를 반복해야 하므로 타입스크립트는 함수가 타입 정제를 무효화하지 않는다고 가정함

> 실제로는 `polygon.bbox`로 사용하는 대신 `bbox` 지역 변수로 뽑아내서 사용하면 `bbox`의 타입은 정확히 유지되만, `polygon.bbox`의 값과 같게 유지되지 않을 수 있음을 주의해야 함

## Item 25) 비동기 코드에는 콜백 대신 async 함수 사용하기

- 비동기 동작 모델링을 위해 ES2015는 콜백 지옥 대신 프로미스(`Promise`) 개념을 도입해, 코드를 직관적으로 이해하거나 오류 처리의 동작을 명확하게 함
- ES2017에서는 async와 await 키워드를 도입하여 더욱 간단하게 처리할 수 있게 됐는데, await 키워드는 각각의 프로미스가 처리(resolve)될 때까지 fetchPages 함수의 실행을 멈추고 async 함수 내에서 await 중인 프로미스가 거절(reject)되면 예외를 던질 수 있음

  ```tsx
  async function fetchPages() {
    try {
      const response1 = await fetch(url1)
      const response2 = await fetch(url2)
      const response3 = await fetch(url3)
      // ...
    } catch (e) {
      // ...
    }
  }
  ```

- `ES5` 또는 더 이전 버전을 대상으로 할 때, 타입스크립트 컴파일러는 `async`와 `await`가 동작하도록 정교한 변환을 수행함
  - 즉, 런타임에 관계없이 `async/await`를 사용할 수 있음

### Promise나 async/await를 사용해야 하는 이유

- 콜백보다는 프로미스가 코드 작성하기 쉬움
- 콜백보다는 프로미스가 타입 추론하기 쉬움
- 예를 들어 병렬로 페이지를 로드하고 싶다면 Promise.all을 사용해 프로미스를 조합할 수 있음

  ```tsx
  async function fetchPages() {
    const [response1, response2, response3] = await Proimse.all([
      fetch(url1),
      fetch(url2),
      fetch(url3),
    ])
    //...
  }
  ```

### await와 구조 분해할당

```tsx
function fetchPagesCB() {
  let numDone = 0
  const responses: string[] = []
  const done = () => {
    const [response1, response2, response3] = responses
    // ...
  }
  const urls = [url1, url2, url3]
  urls.forEach((url, i) => {
    fetchURL(url, (r) => {
      responses[i] = url
      numDone++
      if (numDone === urls.length) done()
    })
  })
}
```

- 타입스크립트는 세 가지 response 변수 각각의 타입을 Response로 추론하지만 콜백 스타일로 동일한 코드를 작성하려면 더 많은 코드와 타입 구문이 필요함
- 이 코드에 오류 처리를 포함하거나 Promise.all 같은 일반적인 코드로 확장하는 것은 어려움

### Promise.race

- 입력된 프로미스들 중 첫 번째가 처리될 때 완료되는 `Promise` 기능은 타입 추론과 잘 맞음
- `Promise.race`를 사용해 프로미스에 타임아웃을 추가하는 방법은 흔하게 사용되는 패턴임

```tsx
function timeout(millis: number): Promise<never> {
  return new Promise((resolve, reject) => {
    setTimeout(() => reject('timeout'), millis)
  })
}

async function fetchWithTimeout(url: string, ms: number) {
  return Promise.race([fetch(url), timeout(ms)])
}
```

- 타입 구문이 없어도 `fetchWithTimeout`의 반환 타입은 `Promise<Response>`로 추론됨
  - `Promise.race`의 반환 타입은 `Promise<Response | never>`가 되지만 `never`와의 유니온은 아무런 효과가 없으므로 결과가 `Promise<Response>`로 간단해짐
- 가끔 프로미스를 직접 생성해야 할 때, `setTimeout`과 같은 콜백 API를 래핑할 경우가 있는데 선택의 여지가 있다면 프로미스를 생성하기보다는 `async/await`를 사용해야 함

  1. 일반적으로 더 간결하고 직관적인 코드가 됨
  2. `async` 함수는 항상 프로미스를 반환하도록 강제됨

     ```tsx
     // function getNumber(): Promise<number>
     async function getNumber() {
       return 42
     }
     ```

  - async 화살표 함수를 만들 수도 있음

    ```tsx
    const getNumber = async () => 42 // 타입이 () => Promise<number>
    ```

  - 프로미스를 직접 생성하면 다음과 같음

    ```tsx
    const getNumber = () => Promise.resolve(42) // 타입이 () => Promise<number>
    ```

  - 즉시 사용 가능한 값에도 프로미스를 반환하는 것은 `비동기 함수로 통일하도록 강제`하는 데 도움이 됨
  - 함수는 항상 동기 또는 비동기로 실행되어야 하며 절대 혼용해서는 안됨

### 이점

- 타입스크립트를 사용하면 타입 정보가 명확히 드러나기 때문에 비동기 코드의 개념을 잡는데 도움이 됨

  ```tsx
  // function getJSON(url: string): Promise<any>
  async function getJSON(url: string) {
    const response = await fetch(url)
    const jsonPromise = response.json() // 타입이 Promise<any>
    return jsonPromise
  }
  ```

## Item 26) 타입 추론에 문맥이 어떻게 사용되는지 이해하기

- 타입스크립트는 타입을 추론할 때 값뿐만 아니라, 값이 존재하는 문맥까지도 살핌
- 타입 추론이 문맥에 어떻게 사용되는지 제대로 이해하면 의도치 않은 타입 추론의 결과에 대해 명확히 대처할 수 있음

```tsx
type Language = 'Javascript' | 'TypeScript' | 'Python'
function setLanguate(language: Language) {
  /*...*/
}

setLanguate('Javascript') // 정상

let language = 'JavaScript'
setLanguate(language)
// 'string' 형식의 인수는 'Language' 형식의 매개변수에 할당될 수 없습니다.
```

- 인라인 형태에서 타입스크립트는 함수 선언을 통해 매개변수가 `Language` 타입이어야 한다는 것을 알고 있음
- 해당 타입에 문자열 리터럴 `JavaScript`는 할당 가능하므로 정상이지만, 이 값을 변수로 분리해내면, 타입스크립트는 할당 시점에 타입을 추론함
  - 이번 경우는 `string`으로 추론했고, `Language` 타입으로 할당이 불가능하므로 오류가 발생함

### 오류 해결 방법

1. 타입 선언에서 `language`의 가능한 값을 제한함

   ```tsx
   let language: Language = 'JavaScript'
   setLanguage(language) // 정상
   ```

- 만약 language의 값에 `Typescript` 같은 오타가 있었다면 오류를 표시해 주는 장점도 있음

2. `language`를 상수로 만듦

   ```tsx
   const language = 'JavaScript'
   setLanguage(language) // 정상
   ```

- `const`를 사용하여 타입 체커에게 `language`는 변경할 수 없다고 알려줌
- 따라서 타입스크립트는 `language`에 대해서 더 정확한 타입인 문자열 리터럴 `JavaScript`로 추론할 수 있음

### 튜플 사용 시 주의점

- 문자열 리터럴 타입과 마찬가지로 튜플 타입에서도 문제가 발생함

```tsx
// 매개변수는 (latitude, logitude) 쌍
function panTo(where: [number, number]) {
  /*...*/
}

panTo([10, 20]) // 정상

const loc = [10, 20]
panTo(loc)
// 'number[]' 형식의 인수는
// '[number, number]' 형식의 매개변수에 할당될 수 없음
```

- 첫 번째 경우는 `[10, 20]`이 튜플 타입 `[number, number]`에 할당 가능함
- 두 번째 경우는 타입스크립트가 `loc`의 타입을 `number[]`로 추론함
- `any`를 사용하지 않고 오류를 고칠 수 있는 방법은 `상수 문맥`을 제공

  - `const`는 단지 값이 가리키는 참조가 변하지 않는 얕은 상수인 반면, `as const`는 그 값이 내부까지 상수라는 사실을 타입스크립트에게 알려줌

    ```tsx
    const loc = [10, 20] as const
    panTo(loc)
    // 'readonly [10, 20]' 형식은 'readonly'이며
    // 변경 가능한 형식은 '[number, number]'에 할당할 수 없습니다.
    ```

    - `panTo`의 타입 시그니처는 `where`의 내용이 불변이라고 보장하지 않음
      - `loc` 매개변수가 `readonly` 타입으므로 동작하지 않음

  - panTo 함수에 readonly 구문을 추가

    ```tsx
    function panTo(where: readonly [number, number]) {
      /*...*/
    }
    const loc = [10, 20] as const
    panTo(loc) // 정상
    ```

- as const는 타입 정의에 실수가 있다면 오류는 타입 정의가 아니라 호출되는 곳에서 발생함

  - 여러 겹 중된 객체에서 오류가 발생한다면 근본적인 원인을 파악하기 어려움

    ```tsx
    const loc = [10, 20, 30] as const // 실제 오류는 여기서 발생합니다.
    panTo(loc)
    // 'readonly [10, 20, 30]' 형식의 인수는
    // 'readonly [number, number]' 형식의 매개변수에 할당될 수 없습니다.
    // 'length' 속성의 형식이 호환되지 않습니다.
    // '3' 형식은 '2' 형식에 할당할 수 없습니다.
    ```

### 객체 사용 시 주의점

- 문맥에서 값을 분리하는 문제는 문자열 리터럴이나 튜플을 포함하는 큰 객체에서 상수를 뽑아낼 때도 발생함

```tsx
type Language = 'JavaScript' | 'TypeScript' | 'Python'
interface GovernedLanguage {
  language: Language
  organization: string
}

function complain(language: GovernedLanguage) {
  /*...*/
}

complain({ language: 'TypeScript', organization: 'Microsoft' }) // 정상

const ts = {
  language: 'TypeScript',
  organization: 'Microsoft',
}

complain(ts)
// '{ language: string; organization: string; }' 형식의 인수는
// 'GovernedLanguage' 형식의 매개변수에 할당될 수 없습니다.
// 'language' 속성의 형식이 호환되지 않습니다.
// 'string' 형식은 'Language' 형식에 할당할 수 없습니다.
```

- ts 객체에서 `language`의 타입은 `string`으로 추론됨
- 타입 선언을 추가하거나 상수 단언(`as const`)을 사용해 해결해야 함

### 콜백 사용 시 주의점

- 콜백을 다른 함수로 전달할 때, 타입스크립트는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용함

```tsx
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
  fn(Math.random(), Math.random())
}

callWithRandomNumbers((a, b) => {
  a // 타입이 number
  b // 타입이 number
  console.log(a + b)
})
```

- `callWithRandom`의 타입 선언으로 인해 `a`와 `b`의 타입이 `number`로 추론됨
- 콜백을 상수로 뽑아내면 문맥이 소실되고 `noImplicitAny` 오류가 발생함

```tsx
const fn = (a, b) => {
  // 'a' 매개변수에는 암시적으로 'any' 형식이 포함됨
  // 'b' 매개변수에는 암시적으로 'any' 형식이 포함됨
  console.log(a + b)
}
callWithRandomNumbers(fn)
```

- 이런 경우는 매개변수에 타입 구문을 추가해서 해결할 수 있음

```tsx
const fn = (a: number, b: number) => {
  console.log(a + b)
}
callWithRandomNumbers(fn)
```

> 가능할 경우 전체 함수 표현식에 타입 선언을 적용하는 것임

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 131 ~ 147p
