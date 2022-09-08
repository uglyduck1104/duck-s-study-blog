---
title: 타입스크립트의 타입 시스템 - Item 12 ~ 14
date: '2022-09-08'
tags: ['typescript']
draft: false
summary: 함수 표현식에 타입 적용하기, 타입과 인터페이스의 차이점 알기, 타입 연산과 제너릭 사용으로 반복 줄이기
layout: PostSimple
authors: ['default']
---

# 타입스크립트의 타입 시스템 - Item 12 ~ 14

## Item 12) 함수 표현식에 타입 적용하기

- 자바스크립트에서는 `함수 문장`과 `함수 표현식`을 다르게 인식함
- 타입스크립트에서는 `함수 표현식`을 사용하는 것이 좋음

  - 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있음

    ```tsx
    function rollDice1(sides: number): number {
      /* ... */
    } // 문장
    const rollDice2 = function (sides: number): number {
      /* ... */
    } // 표현식
    const rollDice3 = (sides: number): number => {
      /* ... */
    } // 표현식
    ```

- 함수 타입의 선언은 불필요한 코드의 반복을 줄임

  - 함수 선언문

    ```tsx
    function add(a: number, b: number) {
      return a + b
    }
    function minus(a: number, b: number) {
      return a - b
    }
    function mul(a: number, b: number) {
      return a * b
    }
    function div(a: number, b: number) {
      return a / b
    }
    ```

  - 함수 시그니처를 하나의 함수 타입으로 통합

    ```tsx
    type BinaryFn = (a: number, b: number) => number
    const add: BinaryFc = (a, b) => a + b
    const minus: BinaryFc = (a, b) => a - b
    const mul: BinaryFc = (a, b) => a * b
    const div: BinaryFc = (a, b) => a / b
    ```

    - 함수 구현부도 분리되어 있어 로직이 보다 분명해짐
    - 함수 타입 선언을 이용했던 예제보다 타입 구문이 적음

### fetch 함수 예제

```tsx
async function getQuote() {
  const reponse = await fetch('/quote?by=Mark+Twain') // 타입이 Promise<Response>
  const quote = await response.json()
  return quote
}
// {
//   "quote": "If you tell the truth, you don't have to remember anything.",
//   "source": "notebook",
//   "date": "1894",
// }
```

- `/quote`가 존재하지 않는 API라면 ‘`404 Not Found`’가 포함된 내용을 응답함
- 하지만, 응답은 JSON 형식이 아닐 수 있으므로 `response.json()`은 JSON 형식이 아니라는 새로운 오류 메시지를 담아 거절(`rejected`)된 프로미스를 반환
  - 호출한 곳에서는 새로운 오류 메시지가 전달되어 실제 오류인 `404`가 감추어짐
  - fetch가 실패하면 거절된 프로미스를 응답하지 않는다는 걸 간과하기 쉬움

```tsx
// lib.dom.d.ts에 있는 fetch의 타입 선언을 확인
declare function fetch(input: RequestInfo, init?: RequestInit): Promise<Response>
```

```tsx
// checkedFetch 함수
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init)
  if (!reponse.ok) {
    // 비동기 함수 내에서 거절된 프로미스로 변환
    throw new Error('Request failed: ' + response.status)
  }
  return response
}
```

```tsx
// checkedFetch 함수의 간결한 버전
const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init)
  if (!reponse.ok) {
    throw new Error('Request failed: ' + response.status)
  }
  return response
}
```

- 함수 문장을 함수 표현식으로 바꿨고 함수 전체에 타입(`typeof fetch`)을 적용
- 타입스크립트가 `input`과 `init`의 타입을 추론할 수 있게 해 줌
- 타입 구문 또한 `checkedFetch`의 반환 타입을 보장하며, `fetch`와 동일함

> 함수 표현식 `전체 타입을 정의`하는 것이 코드도 간결하고 안전하며, 다른 함수의 시그니처와 동일한 타입을 가지는 새 함수를 작성하거나, 동일한 타입 시그니처를 가지는 여러 개의 함수를 작성할 때는 매개변수의 타입과 반환 타입을 반복해서 작성하지 말고 `함수 전체 타입 선언을 적용`해야 함

## Item 13) 타입과 인터페이스의 차이점 알기

- 명명된 타입(named type)을 정의하는 방법

```tsx
// 타입 선언
type TState = {
  name: string;
  capital: string;
}

// 인터페이스 선언
interface IState = {
  name: string;
  capital: string;
}
```

### 비슷한 점

- 명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없음
- 인덱스 시그니처는 인터페이스와 타입에서 모두 사용가능

  ```tsx
  type TDict = { [key: string]: string }
  interface IDict {
    [key: string]: string
  }
  ```

- 함수 타입도 인터페이스나 타입으로 정의할 수 있음

  ```tsx
  type TFn = (x: number) => string
  interface IFn {
    (x: number): string
  }

  const toStrT: TFn = (x) => '' + x // 정상
  const toStrI: IFn = (x) => '' + x // 정상
  ```

- 타입 별칭과 인터페이스는 모두 제너릭이 가능함

  ```tsx
  type TPair<T> = {
    first: T
    second: T
  }
  interface IPair<T> {
    first: T
    second: T
  }
  ```

- 인터페이스는 타입을 확장할 수 있으며, 타입은 인터페이스를 확장할 수 있음

  ```tsx
  interface IStateWithPop extends TState {
    population: number
  }
  type TStateWithPop = IState & { population: number }
  ```

  - `IStateWithPop`과 `TStateWithPop`은 동일함
    - 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지는 못하고, 확장하려면 `타입`과 `&`를 사용해야 함

### 다른 점

- 인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없음

  ```tsx
  type Input = {
    /* ... */
  }
  type Output = {
    /* ... */
  }
  interface VariableMap {
    [name: string]: Input | Output
  }
  ```

  - `Input`과 `Output`은 별도의 타입이며 이 둘을 하나의 변수명으로 매핑하는 `VariableMap` 인터페이스를 만들 수 있음

- `type` 키워드는 일반적으로 `interface`보다 쓰임새가 많음

  - 유니온이 될 수도 있고, 매핑된 타입 또는 조건부 타입 같은 고급 기능에 활용됨
  - 튜플과 배열 타입도 `type` 키워드를 이용해 간결하게 표현할 수 있음

    ```tsx
    type Pair = [number, number]
    type StringList = string[]
    type NamedNums = [string, ...number[]]
    ```

    - 인터페이스로 튜플을 비슷하게 구현하면 튜플에서 사용할 수 있는 `concat` 같은 메서드들을 사용할 수 없음(`type` 키워드 권장)

- `Interface` 키워드는 보강(`augment`)이 가능함

  ```tsx
  interface IState {
    name: string
    capital: string
  }
  interface IState {
    population: number
  }
  const wyoming: IState = {
    name: 'Wyoming',
    capital: 'Cheyenne',
    population: 500_000,
  } // 정상
  ```

  - 속성을 확장하는 것을 선언 병합(`declaration merging`)이라 하고, 타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 `반드시 인터페이스를 사용`해야 하며 표준을 따라야 함

> 복잡한 타입이라면 고민할 것도 없이 타입 별칭을 사용하고, 어떤 `API`에 대한 타입 선언을 작성해야 한다면 인터페이스를 사용하는 것이 좋음
> (`API`가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합할 수 있어 유용함)

## Item 14) 타입 연산과 제너릭 사용으로 반복 줄이기

- 원기둥(`cylinder`)의 반지름과 높이, 표면적, 부피를 출력하는 코드
- 반복 제거 전

  ```tsx
  console.log(
    'Cylinder 1 x 1 ',
    'Surface area:', 6.283185 * 1 * 1 + 6.283185 * 1 * 1,
    'Volume:', 3.14159 * 1 * 1 * 1
  )
  console.log(
    'Cylinder 1 x 2 ',
    'Surface area:', 6.283185 * 1 * 1 + 6.283185 * 2 * 1,
    'Volume:', 3.14159 * 1 * 2 * 1
  )
  console.log(
    'Cylinder 2 x 1 ',
    'Surface area:', 6.283185 * 2 * 1 + 6.283185 * 2 * 1,
    'Volume:', 3.14159 * 2 * 2 * 1
  )
  ```

- 반복 제거 후

  ```tsx
  const surfaceArea = (r, h) => 2 * Math.PI * r * (r + h)
  const volume = (r, h) => Math.PI * r * r * h
  for (const [r, h] of [
    [1, 1],
    [1, 2],
    [2, 1],
  ]) {
    console.log(
      `Cylinder ${r} x ${h}`,
      `Surface area: ${surfaceArea(r, h)}`,
      `Volume: ${volume(r, h)}`
    )
  }
  ```

- 같은 코드를 반복하지 말라는 DRY(`don’t repeat yourself`) 원칙

  ```tsx
  interface Person {
    firstName: string
    lastName: string
  }

  interface PersonWithBirthDate {
    firstName: string
    lastName: string
    birth: date
  }
  ```

  - 타입 중복은 코드 중복만큼 많은 문제를 발생시키는데, 선택적 필드인 `middleName`을 `Person`에 추가한다고 가정했을 때 `Person`과 `PerosnWithBirthDate`는 다른 타입이 됨
  - 타입 중복이 더 흔한 이유 중 하나는 공유된 패턴을 제거하는 메거니즘이 기존 코드에서 하던 것과 비교해 덜 익숙하기 때문임

### 예제

- 반복을 줄이는 가장 간단한 방법은 타입에 이름을 붙임

  ```tsx
  function distance(a: { x: number; y: number }, b: { x: number; y: number }) {
    return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2))
  }
  ```

  - 코드를 수정해 타입에 이름을 붙이면 아래와 같음

    ```tsx
    interface Point2D {
      x: number
      y: number
    }
    function distance(a: Point2D, b: Point2D) {
      /* ... */
    }
    ```

    - 상수를 사용해서 반복을 줄이는 기법을 동일하게 타입 시스템에 적용

- 명명된 타입으로 시그니처 분리하기

  - 변경 전

    ```tsx
    function get(url: string, opts: Options): Promise<Response> {
      /* ... */
    }
    function post(url: string, opts: Options): Promise<Response> {
      /* ... */
    }
    ```

  - 변경 후

    ```tsx
    type HTTPFunction = (url: string, opts: Options) => Promise<Response>
    const get: HTTPFunction = (url, opts) => {
      /* ... */
    }
    const post: HTTPFunction = (url, opts) => {
      /* ... */
    }
    ```

- `Person/PersonWithBirthDate` 예제에서는 한 인터페이스가 다른 인터페이스를 확장하게 해서 반복을 제거할 수 있음

  ```tsx
  interface Person {
    firstName: string
    lastName: string
  }

  interface PersonWithBirthDate extends Person {
    birth: Date
  }
  ```

  - 두 인터페이스가 필드의 부분 집합을 공유한다면, 공통 필드만 골라서 기반 클래스로 분리해 낼 수 있음

- 전체 애플리케이션의 상태를 표현하는 `State` 타입과 부분만 표현하는 `TopNavState`의 경우

  ```tsx
  interface State {
    userId: string
    pageTitle: string
    recentFiles: string[]
    pageContents: string
  }

  interface TopNavState {
    userId: string
    pageTitle: string
    recentFiles: string[]
  }
  ```

  - `TopNavState`를 확장하여 `State`를 구성하기보다, `State`의 부분 집합으로 `TopNavState`를 정의하는 것이 바람직함
  - `State`를 인덱싱하여 속성의 타입 중복 제거

    ```tsx
    type TopNavState = {
      userId: State['userId']
      pageTitle: State['pageTitle']
      recentFiles: State['recentFiles']
    }
    ```

  - 매핑된 타입으로 개선

    ```tsx
    type TopNavState = {
      [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k]
    }
    ```

    - 배열의 필드를 루프 도는 것과 같은 방식
    - 표준 라이브러리에서도 사용되며 Pick이라고 함

      ```tsx
      type Pick<T, K> = { [k in K]: T[k] }
      ```

    - 다음과 같이 사용할 수 있음

      ```tsx
      type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>
      ```

    - Pick은 제너릭 타입으로써, 함수를 호출하는 것과 마찬가지임

- 태그된 유니온의 중복

  ```tsx
  interface SaveAction {
    type: 'save'
    // ...
  }
  interface LoadAction {
    type: 'load'
    // ...
  }
  type Action = SaveAction | LoadAction
  type ActionType = 'save' | 'load' // 타입의 반복!
  ```

  - Action 유니온을 인덱싱하여 ActionType을 정의

    ```tsx
    type ActionType = Action['type'] // 타입은 "save" | "load"
    ```

  - `Action` 유니온에 타입을 더 추가하면 `ActionType`은 자동적으로 그 타입을 포함함

    ```tsx
    type ActionRec = Pick<Action, 'type'> // {type: "save" | "load" }
    ```

- 값의 형태 타입 정의

  ```tsx
  const INIT_OPTIONS = {
    width: 640,
    height: 480,
    color: '#00FF00',
    label: 'VGA',
  }
  interface Options {
    width: number
    height: number
    color: string
    label: stirng
  }
  ```

  - `typeof`를 사용해서 값의 형태에 해당하는 타입을 정의

    ```tsx
    type Options = typeof INIT_OPTIONS
    ```

- 함수나 메서드의 반환 값에 명명된 타입 생성

  ```tsx
  function getUserInfo(userId: string) {
    // ...
    return {
      userId,
      name,
      age,
      height,
      weight,
      favoriteColor,
    }
  }

  // 추론한 반환 타입은 { userId: string; name: string; age: number, ... }
  ```

  - 조건부 타입이 필요하므로 `ReturnType` 제너릭을 사용

    ```tsx
    type UserInfo = ReturnType<typeof getuserInfo>
    ```

    - `ReturnType`은 함수의 값인 `getUserInfo`가 아니라 함수의 타입인 `typeof getUserInfo`에 적용됨
    - `typeof`와 마찬가지로 적용 대상이 값인지 타입인지 정확히 알고 구분해서 처리해야 함

- `extends` 키워드 사용

  - 제너릭 타입에서 매개변수를 제한할 수 있는 방법이 필요한데 `extends` 키워드를 사용해 제너릭 매개변수가 특정 타입을 확장한다고 선언

    ```tsx
    type Pick<T, K extends keyof T> = {
      [K in K]: T[k]
    } // 정상
    ```

    - `K`는 인덱스로 사용될 수 있는 `string | number | symbol`이 되어야 하며 실제로는 범위를 조금 더 좁힐 수 있는 데, `keyof T`를 사용해 `부분 집합`의 개념으로 접근해야 함
    - 타입이 값의 집합이라는 관점에서 생각하면 `extends`를 확장이 아니라 부분 집합이라는 걸 이해하는 데 도움이 됨

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 65 ~ 84p
