---
title: any 다루기 - Item 39 ~ 41
date: '2022-12-13'
tags: ['typescript']
draft: false
summary: any를 구체적으로 변형해서 사용하기, 함수 안으로 타입 단언문 감추기, any의 진화를 이해하기
layout: PostSimple
authors: ['default']
---

# any 다루기 - Item 39 ~ 41

## Item 39) any를 구체적으로 변형해서 사용하기

- `any`는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입이므로 `any`보다 더 구체적으로 표현할 수 있는 타입을 찾아 타입 안정성을 높여야 함

```tsx
function getLengthBad(array: any) {
  // Do not this 🙅🏻‍♂️
  return array.length
}

function getLength(array: any[]) {
  return array.length
}
```

- `any`를 사용하는 함수 보다 `any[]`를 사용하는 함수가 더 권장되는 이유는 3가지로 들 수 있음
  1. 함수 내의 `array.length` 타입이 체크됨
  2. 함수의 반환 타입이 `any` 대신 `number`로 추론됨
  3. 함수 호출될 때 매개변수가 배열인지 체크됨
- 배열이 아닌 값을 넣어서 실행해 보면, `getLength`는 제대로 오류를 표시하지만 `getLengthBad`는 오류를 잡아내지 못함

  ```tsx
  getLengthBad(/123/) // 오류 없음, undefined를 반환
  getLength(/123/)
  // 'RegExp' 형식의 인수는
  // 'any[]'형식의 매개변수에 할당될 수 없습니다.
  ```

  - 함수의 매개변수를 구체화할 때에는 배열의 형태가 명확하다면 `any[][]`처럼 선언하면 됨

    - 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 `{[key: string]: any}`처럼 선언

      ```tsx
      function hasWelveLetterKey(o: { [key: string]: any }) {
        for (const key in o) {
          if (key.length === 12) {
            return true
          }
        }
        return false
      }
      ```

    - 앞의 예제처럼 함수의 매개변수가 객체지만 값을 알 수 없다면 `{[key: string]: any}` 대신 모든 `no-premitive` 타입을 포함하는 `object` 타입을 사용할 수 있음
      - 대신 객체의 키를 열거할 수 있으나 속성에 접근할 수 없음
      - 객체지만 객체의 속성에 접근할 수 없어야 한다면 `unknown` 타입을 사용
    - 함수의 타입에서도 단순히 `any`를 사용하면 안됨

      ```tsx
      type Fn0 = () => any // 매개변수 없이 호출 가능한 모든 함수
      type Fn1 = (arg: any) => any // 매개변수 1개
      type FnN = (...args: any[]) => any // 모든 개수의 매개변수 "Function" 타입과 동일함
      ```

      - 단순히 any 타입을 쓰는 것에 비해, 마지막줄의 `…args`의 타입을 `any[]`로 선언한 것은 이전에도 설명했듯이 배열 형태를 알 수 있어 추론을 더 구체적으로 할 수 있음

## Item 40) 함수 안으로 타입 단언문 감추기

- 타입 추론 시스템을 잘 사용하기 위해서는 모든 함수를 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 상황까지 고려해 가며 타입 정보를 힘들게 구성할 필요는 없음
- 함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 정확히 명시하는 정도로 끝내는 것을 권장함

```tsx
declare function cacheLast<T extends Function>(fn: T): T
declare function shallowEqual(a: any, b: any): boolean
```

- 자신의 마지막 호출을 캐시하도록 만드는 예시로써, 함수 캐싱은 리액트 같은 프레임워크에서 실행 시간이 오래 걸리는 함수 호출을 개선하는 일반적인 기법

```tsx
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null
  let lastResult: any
  return function (...args: any[]) {
    // '(...args: any[]) => any' 형식은 'T' 형식에 할당할 수 없습니다.
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args)
      lastArgs = args
    }
    return lastResult
  }
}
```

- `cacheLast` 함수의 구현체에서 볼 수 있듯이, 타입스크립트는 반환문에 있는 함수와 원본 함수 `T` 타입이 어떤 관련이 있는지 알지 못하므로 오류가 발생함

  - 결과적으로 원본 함수 `T` 타입과 동일한 매개변수로 호출되고 반환값 역시 예상한 결과가 되므로 타입 단언문을 추가해서 오류를 제거하는 것이 큰 문제가 되지는 않음

    ```tsx
    function cacheLast<T extends Function>(fn: T): T {
      let lastArgs: any[] | null = null
      let lastResult: any
      return function (...args: any[]) {
        // '(...args: any[]) => any' 형식은 'T' 형식에 할당할 수 없습니다.
        if (!lastArgs || !shallowEqual(lastArgs, args)) {
          lastResult = fn(...args)
          lastArgs = args
        }
        return lastResult
      } as unknown as T
    }
    ```

    - 함수 내부에는 `any`가 많이 보이지만 타입 정의에는 `any`가 없기 때문에, `cacheLast`를 호출하는 쪽에서는 `any`가 사용됐는지 알지 못함

- 앞 예제에 나온 `shallowEqual`은 두 개의 배열을 매개변수로 받아서 비교하는 함수이며 타입 정의와 구현이 비교적 간단하지만 다음 살펴볼 함수는 객체를 매개변수로 하는 `shallowObjectEqual`의 타입 정의임

  ```tsx
  declare function shallowObjectEqual<T extends object>(a: T, b: T): boolean
  declare function shallowEqual(a: any, b: any): boolean
  ```

  ```tsx
  function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
    for (const [k, aVal] of Object.entries(a)) {
      if (!(k in b) || aVal !== b[k]) {
        // '{}' 형식에 인덱스 시그니처가 없으므로
        // 요소에 암시적으로 'any' 형식이 있습니다.
        return false
      }
    }
    return Obejct.keys(a).length === Object.keys(b).length
  }
  ```

  - `if` 구문의 `k in b` 체크로 `b` 객체에 `k` 속성이 있다는 것을 확인했으나, `b[k]` 부분에서 오류가 발생하는 것이 이상하지만 실질적으로 오류가 아니라는 것을 알고 있으므로 `any`로 단언해야 함

    ```tsx
    function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
      for (const [k, aVal] of Object.entries(a)) {
        if (!(k in b) || aVal !== (b as any)[k]) {
          return false
        }
      }
      return Obejct.keys(a).length === Object.keys(b).length
    }
    ```

    - `b as any` 타입 단언문은 안전하며, 객체가 같은지 체크하기 위해 객체 순회와 단언문이 코드에 직접 들어가므로 해당 코드처럼 별도의 함수로 분리해 내는 것이 훨씬 좋은 설계임

## Item 41) any의 진화를 이해하기

- 타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정되지만 `any` 타입과 관련해서 예외인 경우가 존재함

```tsx
function range(start: number, limit: number) {
  const out = []
  for (let i = start; i < limit; i++) {
    out.push(i)
  }
  return out // 반환 타입이 nubmer[]로 추론됨.
}
```

- 언뜻봐서는 정상인 것처럼 보이나, `out`의 타입이 처음에는 `any` 타입 배열인 `[]`로 초기화되었다가 마지막에는 `number[]`로 추론되고 있음
- 코드의 `out`이 등장하는 세 가지 위치를 보면 이유를 알 수 있음

  - `const out = [];` → 타입이 `any[]`
  - `out.push(i)` → out의 타입이 `any[]`
  - `return out;` → 타입이 `number[]`

  > `out`의 타입은 `any[]`로 선언되었지만 `number` 타입의 값을 넣는 순간부터 타입은 `number[]`로 진화함

- 배열에 다양한 타입의 요소를 넣으면 배열의 타입이 확장되면서 진화함

  ```tsx
  const result = [] // 타입이 any[]
  result.push('a')
  result // 타입이 string[]
  result.push(1)
  result // 타입이 (string | number)[]
  ```

- 조건문에서는 분기에 따라 타입이 변할 수 있음

  ```tsx
  let val // 타입이 any
  if (Math.random() < 0.5) {
    val = /hello/
    val // 타입이 RegExp
  } else {
    val = 12
    val // 타입이 number
  }
  val // 타입이 number | RegExp
  ```

  - 변수의 초깃값이 `null`인 경우도 `any`의 진화가 일어남

- 보통 `try/catch` 블록 안에서 변수를 할당하는 경우에 발생함

  ```tsx
  let val = null // 타입이 any
  try {
    somethingDangerous()
    val = 12
    val // 타입이 number
  } catch (e) {
    consle.warn('alas!')
  }
  val // 타입이 number | null
  ```

- `any` 타입의 진화는 `noImplicitAny`가 설정된 상태에서 변수의 타입이 암시적 `any`인 경우에만 일어나지만 명시적으로 `any`를 선언하면 타입이 그대로 유지됨

  ```tsx
  let val: any // 타입이 any
  if (Math.random() < 0.5) {
    val = /hello/
    val // 타입이 any
  } else {
    val = 12
    val // 타입이 any
  }
  val // 타입이 any
  ```

> 타입을 안전하게 지키기 위해서는 암시적 `any`를 진화시키는 방식보다는 명시적 타입 구문을 사용하는 것이 더 좋은 설계임

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 206 ~ 215p
