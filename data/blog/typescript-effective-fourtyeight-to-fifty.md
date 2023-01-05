---
title: 타입 선언과 @types - Item 48 ~ 50
date: '2023-01-05'
tags: ['typescript']
draft: false
summary: API 주석에 TSDoc 사용하기, 콜백에서 this에 대한 타입 제공하기, 오버로딩 타입보다는 조건부 타입을 사용하기
layout: PostSimple
authors: ['default']
---

# 타입 선언과 @types - Item 45 ~ 47

## Item 48) API 주석에 TSDoc 사용하기

### JSDoc 스타일 사용하기

```tsx
/** 인사말을 생성합니다. 결과는 보기 좋게 꾸며집니다 */
function greetJSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`
}
```

- 사용자를 위한 문서라면 `JSDoc` 스타일의 주석으로 만드는 것이 좋은 이유는 대부분의 편집기는 함수가 호출되는 곳에서 함수에 붙어 있는 `JSDoc` 스타일의 주석을 툴팁으로 표시해 줌
- 타입스크립트 언어 서비스가 JSDoc 스타일을 지원하므로 적극적으로 활용하는 것을 권장하고 만약 공개 API에 주석을 붙인다면 JSDoc 형태로 작성해야 함

### `@param, @returns`

```tsx
/**
 * 인사말을 생성합니다.
 * @param name 인사할 사람의 이름
 * @param title 그 사람의 칭호
 * @returns 사람이 보기 좋은 형태의 인사말
 */
function greetFullTSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`
}
```

- `@param`과 `@returns`를 추가하면 함수를 호출하는 부분에서 각 매개변수와 관련된 설명을 보여줌

### TSDoc

- 타입 정의에 TSDoc을 사용할 수도 있음

```tsx
/** 특정 시간과 장소에서 수행된 측정 */
interface Measurement {
  /** 어디에서 측정되었나? */
  position: Vector3D
  /** 언제 측정되었나? epoch에서부터 초 단위로 */
  time: number
  /** 측정된 운동량 */
  momentum: Vector3D
}
```

- `Measurement` 객체의 각 필드에 마우스를 올려 보면 필드별로 설명을 볼 수 있음
- `TSDoc` 주석은 마크다운 형식으로 꾸며지므로 굵은 글씨, 기울임 글씨, 글머리기호 목록을 사용할 수 있음
- `JSDoc`에는 타입 정보를 명시하는 규칙(`@parmas {string} name..`)이 있지만, 타입스크립트에서는 타입 정보가 코드에 있기 때문에 `TSDoc`에서는 타입 정보를 명시해서는 안됨

## Item 49) 콜백에서 this에 대한 타입 제공하기

### 자바스크립트 this

- 자바스크립트에서 this 키워드는 매우 혼란스러운 기능임
- `let`이나 `const`로 선언된 변수가 렉시컬 스코프인 반면, `this`는 다이나믹 스코프임
  - 다이나믹 스코프는 호출된 방식에 따라 달라짐
- 주로 객체의 현재 인스턴스를 참조하는 클래스에서 가장 많이 쓰임
- `call` method를 사용하면 명시적으로 this를 바인딩하여 문제를 해결할 수 있음

  ```tsx
  class C {
    vals = [1, 2, 3]
    logSquares() {
      for (const val of this.vals) {
        console.log(val * val)
      }
    }
  }

  const c = new C()
  const method = c.logSquares()
  method.call(c) // 제곱을 출력
  ```

- `this` 바인딩은 종종 콜백 함수에서 쓰임

  ```tsx
  class ResetButton {
    render() {
      return makeButton({ text: 'Reset', onClick: this.onClick })
    }
    onClick() {
      alert(`Reset ${this}`)
    }
  }
  ```

  - `ResetButton`에서 `onClick`을 호출하면 `this` 바인딩 문제 발생
  - `해결 방안`

    ```tsx
    class ResetButton {
      constructor() {
        this.onClick = this.onClick.bind(this)
      }
      render() {
        return makeButton({ text: 'Reset', onClick: this.onClick })
      }
      onClick() {
        alert(`Reset ${this}`)
      }
    }
    ```

    - `onClick() {…}`은 `ResetButton.prototype`의 속성을 정의하므로 `ResetButton`의 모든 인스턴스에서 공유되지만 생성자에서 `this.onClick`으로 바인딩하면 `onClick` 속성에 `this`가 바인딩되어 해당 인스턴스에 생성됨
    - 속성 탐색 순서에서 `onClick` 인스턴스 속성은 `onClick` 프로토타입 속성보다 앞에 놓이므로, `render()` 메서드의 `this.onClick`은 바인딩된 함수를 참조함
    - 조금 더 쉬운 `this` 바인딩 방법은 `onClick`을 `Arrow Function`으로 사용해 `ResetButton`이 생성될 때마다 제대로 바인딩된 `this`를 가지는 새 함수를 생성할 수 있음

### 타입스크립트 this

- `this` 바인딩은 자바스크립트의 동작이므로 타입스크립트 역시 `this` 바인딩을 그대로 모델링함

  - 만약 작성 중인 라이브러리에 `this`를 사용하는 콜백 함수가 있다면, `this` 바인딩 문제를 고려해야 함
  - 이 문제는 콜백 함수의 매개변수에 `this`를 추가하고, 콜백 함수를 `call`로 호출해서 해결할 수 있음

    ```tsx
    function addKeyListener(el: HTMLElement, fn: (this: HTMLElement, e: KeyboardEvent) => void) {
      el.addEventListener('keydown', (e) => {
        fn.call(el, e)
      })
    }
    ```

## Item 50) 오버로딩 타입보다는 조건부 타입을 사용하기

### 유니온 타입

```tsx
function double(x) {
  return x + x
}
```

- double 함수에는 string 또는 number 타입의 매개변수가 들어올 수 있으므로 유니온 타입을 추가

  ```tsx
  function double(x: number | string): number | string
  function double(x: any) {
    return x + x
  }
  ```

  - 다음과 같은 모호한 부분을 발견할 수 있음

    ```tsx
    const num = double(12) // string | number
    const str = double('x') // string | number
    ```

  - `double`에 `number` 타입을 매개변수로 넣으면 `number` 타입을 반환하고 `string` 타입을 넣으면 string 타입으로 반환하나, 선언문에는 `number` 타입을 매개변수로 넣고 `string` 타입을 반환하는 경우도 포함되어 있음

### 제네릭

- 제네릭을 사용하면 위의 동작을 아래와 같이 모델링할 수 있음

  ```tsx
  function double<T extends number | string>(x: T): T
  function double(x: any) {
    return x + x
  }

  const num = double(12) // 타입이 12
  const str = double('x') // 타입이 "x"
  ```

  - 타입이 너무 과하게 구체적임
    - 리터럴 문자열 `x`를 매개변수로 넘긴다고 해서 동일한 리터럴 문자열 `x` 타입이 반환되어야 하는 것은 아님(`x`의 두 배는 `x`가 아니라 `xx`임)

### 여러 가지 타입 선언으로 분리(다형성)

- 타입스크립트에서 함수의 구현체는 하나지만, 타입 선언은 몇 개든지 만들 수 있음

```tsx
function double(x: number): number
function double(x: string): string
function double(x: any) {
  return x + x
}

const num = double(12) // 타입이 number
const str = double('x') // 타입이 string
```

- 함수 타입이 조금 명확해졌지만 여전히 버그는 남아 있음
- `string`이나 `number` 타입의 값으로는 잘 동작하지만, 유니온 타입 관련해서 문제가 발생함

```tsx
function f(x: number | string) {
  return double(x)
  // 'string | number' 형식의 인수는
  // 'string' 형식의 매개변수에 할당될 수 없음
}
```

- 타입스크립트는 오버로딩 타입 중에서 일치하는 타입을 찾을 때까지 순차적으로 검색하므로 오버로딩 타입의 마지막 선언까지 검색했을 때, `string|number` 타입은 `string`에 할당할 수 없기 때문에 오류가 발생함

### 조건부 타입 사용

- 가장 좋은 해결책은 조건부 타입을 사용

  ```tsx
  function double<T extends number | string>(x: T): T extends strign ? string : number
  function double(x: any) {
    return x + x
  }
  ```

  - 제너릭을 사용했던 예제와 유사한, 반환 타입이 더 정교함
    - `T`가 `string`의 부분 집합(string, 또는 문자열 리터럴, 또는 문자열 리터럴의 유니온), 반환 타입이 `string`임
    - 그 외의 경우는 반환 타입이 `number`

- 조건부 타입이라면 앞선 모든 예제가 동작함

  ```tsx
  const num = double(12) // number
  const str = double('x') // string

  // function f(x: string | number): string | number
  function f(x: number | string) {
    return double(x)
  }
  ```

  - 유니온에 조건부 타입을 적용하면, 조건부 타입의 유니온으로 분리되므로 number|string이 경우에도 동작함
    - `T`가 `number|string`이라면, 조건부 타입을 다음 단계로 해석함
      - (number|string) extends string ? string : number
        - (number extends string ? string : number) |
          (string extends string ? string : number)
      - number | string
  - 오버로딩 타입이 작성하기는 쉽지만, 조건부 타입은 개별 타입이 유니온으로 일반화하기 때문에 타입이 더 정확해짐
  - 조건부 타입은 타입 체커가 단일 표현식으로 받아들이기 때문에 유니온 문제를 해결할 수 있음

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 239 ~ 250p
