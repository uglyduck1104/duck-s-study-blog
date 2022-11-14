---
title: 타입 추론 - Item 21 ~ 23
date: '2022-10-11'
tags: ['typescript']
draft: false
summary: 타입 넓히기, 타입 좁히기, 한꺼번에 객체 생성하기
layout: PostSimple
authors: ['default']
---

# 타입 추론 - Item 21 ~ 23

## Item 21) 타입 넓히기

- 런타임에 모든 변수는 유일한 값을 가지지만, 타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에 변수는 가능한 값들의 집합인 `타입`을 가짐
- 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추하는 과정을 `타입 넓히기`라고 부름
  - 타입 넓히기를 이해하면 오류의 원인을 파악하고 타입 구문을 효과적으로 사용할 수 있음

```tsx
interface Vector3 {
  x: number
  y: number
  z: number
}
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis]
}
```

- Vector3 함수를 사용한 다음 코드는 런타임에 오류 없이 실행되지만, 편집기에서는 오류가 표시됨

```tsx
let x = 'x'
let vec = { x: 10, y: 20, z: 30 }
getComponent(vec, x)
// ~ 'string' 형식의 인수는 "x" | "y" | "Z"
// 형식의 매개변수에 할당될 수 없습니다.
```

- `getComponent` 함수는 두 번 쟤 매개변수에 `"x" | "y" | "z"` 타입을 기대했지만, x의 타입은 할당 시점에 넓히가 동작하여 `string`으로 추론되었음
- 타입 넓히기가 진행될 때, 주어진 값으로 추론 가능한 타입이 여러 개이므로 타입 추론 과정이 상당히 모호해짐

```tsx
const mixed = ['x', 1]
```

- mixed의 타입은 다음과 같이 추론될 수 있음
  - `('x' | 1)[]`
  - `['x', 1]`
  - `[string, number]`
  - `readonly [string, number]`
  - `(string|number)[]`
  - `readonly (string|number)[]`
  - `[any, any]`
  - `any[]`
- 정보가 충분하지 않으면 `mixed`가 어떤 타입으로 추론되어야 하는지 알 수 없으므로, 타입스크립트는 작성자의 의도를 추측함(`string|number[]`로 추론됨)

### 타입 스크립트의 넓히기 과정

- const 사용하기

  - let 대신 const로 변수를 선언하면 더 좁은 타입이 되므로 앞에서 발생한 오류가 해결됨

    ```tsx
    const x = 'x' // 타입이 "x"
    let vec = { x: 10, y: 20, z: 30 }
    getComponent(vec, x) // 정상
    ```

  - x는 재할당될 수 없으므로 타입스크립트는 더 좁은 타입으로 추론할 수 있음
  - 문자 리터럴 타입 `"x"`는 `"x"|"y"|"z"` 에 할당 가능하므로 코드가 타입 체커를 통과함
  - 객체와 배열의 경우에는 여전히 문제가 발생함

- 타입 추론의 강도를 직접 제어하기

  1. 명시적 타입 구문 제공

     ```tsx
     const v: { x: 1 | 3 | 5 } = {
       x: 1,
     } // 타입이 { x: 1|3|5; }
     ```

  2. 타입 체커에 추가적인 문맥을 제공

  - ex) 함수의 매개변수로 값을 전달

  3. const 단언문 사용

  - `const` 단언문은 변수 선언에 쓰이는 `const`와 달리 온전히 타입 공간의 기법임

    ```tsx
    const v1 = {
      x: 1,
      y: 2,
    } // 타입은 { x: number, y: number; }

    const v2 = {
      x: 1 as const,
      y: 2,
    } // 타입은 { x: 1: y: number; }

    const v3 = {
      x: 1,
      y: 2,
    } as const // 타입은 { readonly x: 1; readonly y: 2; }
    ```

  - 값 뒤에 `as const`를 작성하면, 타입스크립트는 최대한 좁은 타입으로 추론함
  - 배열을 튜플 타입으로 추론할 때에도 `as const`를 사용할 수 있음

- 넓히기로 인해 오류가 발생한다고 생각되면, 명시적 타입 구문 또는 `const` 단언문을 추가하는 것을 고려해야 함

## Item 22) 타입 좁히기

- 타입 넓히기의 반대 개념인 타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말함

### 조건문 사용하기

- 가장 일반적인 예시는 `null` 체크

  ```tsx
  const el = document.getElementById('foo') // 타입이 HTMLElement | null
  if (el) {
    el // 타입이 HTMLElement
    el.innerHTML = 'Party Time'.blink()
  } else {
    el // 타입이 null
    alert('No element #foo')
  }
  ```

  - 타입 체커는 일반적으로 이런 조건문에서 타입 좁히기를 잘하지만 타입 별칭이 존재하면 그러지 못할 수 있음

- 분기문에서 예외를 던지거나 함수를 반환하여 블록의 나머지 부분에서 변수의 타입을 좁힐 수 있음

  ```tsx
  const el = document.getElementById('foo') // 타입이 HTMLElement | null
  if (!el) throw new Error('Unable to find #foo')
  el // 이제 타입은 HTMLElement
  el.innerHTML = 'Party Time'.blink()
  ```

### `intanceof` 사용하기

```tsx
function contains(text: string, search: string | RegExp) {
  if (search instanceof RegExp) {
    search // 타입이 RegExp
    return !!search.exec(text)
  }
  search // 타입이 string
  return text.includes(search)
}
```

### 속성 체크 사용하기

```tsx
interface A {
  a: number
}
interface B {
  b: number
}
function pickAB(ab: A | B) {
  if ('a' in ab) {
    ab // 타입이 A
  } else {
    ab // 타입이 B
  }
  ab // 타입이 A | B
}
```

### 내장 함수(`Array.isArray`) 사용하기

```tsx
function contains(text: string, terms: string | string[]) {
  const termList = Array.isArray(term) ? terms : [terms]
  termList // 타입이 string[]
  // ...
}
```

### 잘못된 사용

```tsx
const el = document.getElementById('foo') // 타입이 HTMLElement | null
if (typeof el === 'object') {
  el // 타입이 HTMLElement | null
}
```

- 유니온 타입에서 `null`을 제외하고 싶었지만 자바스크립트에서는 `typeof null`이 `object` 이므로, if 구문에서 `null`이 제외되지 않음

```tsx
function foo(x?: number | string | null) {
  if (!x) {
    x // 타입이 string | number | null | undefined
  }
}
```

- 빈 문자열 `‘’`과 `0` 모두 `false`가 되므로 타입은 전혀 좁혀지지 않고 `x`는 여전히 블록 내에서 `string` 또는 `number`가 됨

### 명시적 태그 붙이기

```tsx
interface UploadEvent {
  type: 'upload'
  filename: string
  contents: string
}
interface DownloadEvent {
  type: 'download'
  filename: string
}
type AppEvent = UploadEvent | DownloadEvent
function handleEvent(e: AppEvent) {
  switch (e.type) {
    case 'download':
      e // 타입이 DownloadEvent
      break
    case 'upload':
      e // 타입이 UploadEvent
      break
  }
}
```

- 이 패턴은 태그된 유니온 또는 구별된 유니온이라고 불리며, 타입스크립트 어디에서나 찾아볼 수 있음

### 사용자 정의 타입 가드 사용

```tsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el // 타입이 HTMLInputElement
    return el.value
  }
  el // 타입이 HTMLElement
  return el.textContent
}
```

- 반환 타입의 `el is HTMLInputElement`는 함수의 반환이 `true`인 경우, 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알려줌

### 배열에서 사용자 정의 타입 가드 사용

```tsx
function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined
}
const members = ['Janet', 'Michael'].map((who) => jackson5.find((n) => n === who)).filter(isDefined) // 타입이 string[]
```

- 배열에서 어떤 탐색을 수행할 때 `undefined`가 될 수 있는 타입 사용하는 경우
- 타입 가드 및 `filter` 함수를 사용해 타입을 좁힐 수 있음

## Item 23) 한꺼번에 객체 생성하기

- 변수의 값은 변경될 수 있지만, 타입스크립트의 타입은 일반적으로 변경되지 않음
- 따라서, 객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입추론에 유리함

```tsx
const pt = {}
pt.x = 3
// ~ '{}' 형식에 'x' 속성이 없습니다.
pt.y = 4
// ~ '{}' 형식에 'y' 속성이 없습니다.
```

- 자바스크립트에서 2차원의 점을 표현하는 객체를 생성할 때, 타입스크립트에서는 오류가 발생함
- 첫 번째 줄의 `pt` 타입은 `{}` 값을 기준으로 추론되기 때문에 존재하지 않는 속성을 추가할 수 없음
- Point 인터페이스를 정의하면 오류가 다음처럼 바뀜

  ```tsx
  interface Point {
    x: number
    y: number
  }
  const pt: Point = {}
  // ~~ '{}' 형식에 'Point' 형식의 x, y 속성이 없습니다.
  pt.x = 3
  pt.y = 4
  ```

### 객체 한번에 정의하기

- 이 문제들은 객체를 한번에 정의하면 해결할 수 있음

  ```tsx
  const pt = {
    x: 3,
    y: 4,
  } // 정상
  ```

  - 객체를 반드시 제각각 나눠서 만들어야 한다면, 타입 단언문(`as`)을 사용해 타입 체커를 통과하게 할 수 있음

    ```tsx
    // 분리하기
    const dividedPoint = {} as Point
    dividedPT.x = 3
    dividedPT.y = 4 // 정상

    // 한꺼번에 만들기
    const mergedPoint: Point = {
      x: 3,
      y: 4,
    }
    ```

- 객체 전개 연산자(`…`) 사용하여 큰 객체를 한꺼번에 만들 수 있음

  ```tsx
  const namedPoint = { ...pt, ...id }
  namedPoint.name // 정상, 타입이 string
  ```

  - 타입 걱정 없이 필드 단위로 객체를 생성할 수도 있음

    ```tsx
    const pt0 = {}
    const pt1 = { ...pt0, x: 3 }
    const pt: Point = { ...pt1, y: 4 } // 정상
    ```

  - 간단한 객체를 만들기 위해 우회하기는 했으나, 객체에 속성을 추가하고 타입스크립트가 새로운 타입을 추론할 수 있게 해 유용함

### 전개 연산자로 조건부 속성 추가하기

```tsx
declare let hasMiddle: boolean
const firstLast = { first: 'Harry', last: 'Truman' }
const president = { ...firstLast, ...(hasMiddle ? { middle: 'S' } : {}) }
```

- 타입에 안전한 방식으로 조건부 속성을 추가하려면, 속성을 추가하지 않는 `null` 또는 `{}`으로 객체 전개를 사용
- president 심벌은 다음과 같이 타입이 추론됨

  ```tsx
  const president: {
    middle?: string
    first: string
    last: string
  }
  ```

### 전개 연산자로 여러 속성 추가하기

```tsx
declare let hasDates: boolean
const nameTitle = { name: 'Khufu', title: 'Pharaoh' }
const pharaoh = {
  ...nameTitle,
  ...(hasDates ? { start: -2589, end: -2566 } : {}),
}
```

- pharaoh 심벌은 다음과 같이 타입이 추론됨

  ```tsx
  const pharaoh:
    | {
        start: number
        end: number
        name: string
        title: string
      }
    | {
        name: string
        title: string
      }
  ```

### 헬퍼 함수 사용해서 선택적 필드 표현하기

```tsx
function addOptional<T extends object, U extends object>(a: T, b: U | null): T & Partial<U> {
  return { ...a, ...b }
}

const pharaoh = addOptional(nameTitle, hasDates ? { start: -2589, end: -2566 } : null)
pharaoh.start // 정상, 타입이 number | undefined
```

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 118 ~ 130p
