---
title: 타입스크립트의 타입 시스템 - Item 9 ~ 11
date: '2022-09-07'
tags: ['typescript']
draft: false
summary: 타입 단언보다는 타입 선언을 사용하기, 객체 래퍼 타입 피하기, 잉여 속성 체크의 한계 인지하기
layout: PostSimple
authors: ['default']
---

# 타입스크립트의 타입 시스템 - Item 9 ~ 11

## Item 9) 타입 단언보다는 타입 선언을 사용하기

### 변수에 값을 할당하고 타입을 부여하는 방법

```tsx
interface Person {
  name: string
}

const alice: Person = { name: 'Alice' } // 타입은 Person
const bob = { name: 'Bob' } as Person // 타입은 Person
```

1. `alice: Person` - 타입 선언

   - 타입 선언을 붙여서 값이 선언된 타입임을 명시
   - 할당되는 값이 해당 인터페이스를 만족하는지 검사

2. `as Person` - 타입 단언

   - `타입 단언`을 수행하므로 타입스크립트가 추론한 타입이 있더라도 Person 타입으로 간주함
   - `강제로 타입을 지정`했으니 타입 체커에게 오류를 무시하라고 하는 것

> 타입 단언이 꼭 필요한 경우가 아니라면, `안전성 체크`가 되는 타입 선언 사용을 권장

### 화살표 함수 타입 선언의 오류

```tsx
const people = ['alice', 'bob', 'jan'].map(name => ({name});
// Person[]을 원했지만 결과는 { name: string; }[]...
```

- 화살표 함수의 타입 선언은 추론된 타입이 모호할 때가 있는데, 다음과 같이 화살표 함수 안에서 타입과 함께 변수를 선언하는 것이 가장 직관적임

  - 변수의 반환 타입 선언

    ```tsx
    const people = ['alice', 'bob', 'jan'].map(name => {
      const person: Person = {name};
      return person
    }); // 타입은 Person[]
    ```

  - 화살표 함수의 반환 타입 선언

    ```tsx
    const people = ['alice', 'bob', 'jan'].map((name): Person => ({ name })) // 타입은 Person[]
    ```

    - `(name): Person`은 `name`의 타입이 없고 반환 타입이 `Person`이라고 명시
    - `(name: Person)`은 `name`의 `Person`임을 명시하고 반환 타입은 없기 때문에 오류가 발생함

  - 최종 수정 코드

    ```tsx
    const people: Person[] = ['alice', 'bob', 'jan'].map((name): Person => ({ name }))
    ```

    - 최종적으로 원하는 타입을 직접 명시하고, 타입스크립트가 할당문의 유효성을 검사하게 함
    - 함수 호출 체이닝이 연속되는 곳에서는 체이닝 시작에서부터 명명된 타입을 가져야 함

### 타입 단언이 필요한 경우

```tsx
document.querySelector('#myButton').addEventListener('click', (e) => {
  e.currentTarget // 타입은 EventTarget
  const button = e.currentTarget as HTMLButtonElement
  button // 타입은 HTMLButtonElement
})
```

- 타입스크립트는 `DOM`에 접근할 수 없기 때문에 `#myButton`이 버튼 엘리먼트인지 알지 못하며, 이벤트의 `currentTarget`이 같은 버튼이어야 하는 것도 알지 못함
- 타입스크립트가 알지 못하는 정보는 `타입 단언문`을 써야 함
- `!`을 사용해서 `null`이 아님을 단언하는 경우

  ```tsx
  const elNull = document.getElementById('foo') // 타입은 HTMLElement | null
  const el = document.getElementById('foo')! // 타입은 HTMLElement
  ```

  - 변수의 접두사로 쓰인 `!`는 `boolean`의 부정문이지만, 접미사로 쓰인 `!`는 값이 `null`이 아니라는 단언문으로 해석됨
  - 단언문은 컴파일 과정 중에 제거되므로, 타입 체크는 알지 못하지만 값이 null이 아니라고 확신할 수 있을 때 사용해야 함

### 타입 단언이 동작하지 않는 경우

```tsx
interface Person {
  name: string
}
const body = document.body
const el = body as Person
// ~~~~~~~~~~~~~~~~ 'HTMLElement' 형식을 'Person' 형식으로 변환하는 것은
//                  형식이 다른 형식과 충분히 겹치지 않기 때문에
//                  실수일 수 있습니다. 이것이 의도적인 경우에는
//                  먼저 식을 'unknown'으로 변환하십시오.
```

- `HTMLElement`는 `HTMLElement | null`의 서브 타입이므로 타입 단언 동작
- `HTMLButtonElement`는 `EventTarget`의 서브타입이므로 타입 단언 동작
- `Person`과 `HTMLElement`는 서로의 서브타입이 아니므로 변환이 불가능함
  - 이 경우, `unknown` 타입을 사용해서 해결해야 하는데, `unknown`을 사용 한 이상 적어도 무언가 위험한 동작을 하고 있다는 걸 알 수 있음

## Item 10) 객체 래퍼 타입 피하기

- 자바스크립트에는 객체 이외에도 기본형 값들에 대한 일곱 가지 타입이 있는데, 기본형들은 불변하며 메서드를 가지지 않는다는 점에서 객체와 구분됨
  - 기본형인 `string`의 경우 메서드를 가지고 있는 것처럼 보이지만, 실질적으로 `string`의 메서드가 아니며 `string`을 사용할 때 자바스크립트 내부적으로 많은 동작이 일어남
  - `string` 기본형에는 메서드가 없지만, 메서드를 가지는 `String` 객체 타입이 정의되어 있음
    - `string` 기본형에 `charAt` 같은 메서드를 사용할 경우 기본형을 `String` 객체로 래핑하고, 메서드를 호출하며 마지막에 래핑한 객체를 버림

### 객체 래퍼 타입의 자동 변환

```tsx
console.log('hello' === new String('hello'))
// false
console.log(new String('hello') === new String('hello'))
// false
```

- String 객체는 오직 자기 자신하고만 동일함

```tsx
let x = 'hello'
x.language = 'English'
// 'English'
console.log(x.language)
// undefined
```

- 객체 래퍼 타입의 자동 변환으로 인해, 원하는 방식으로 동작하지 못하는 경우가 발생하는데 이는 `x`가 `String` 객체로 변환된 후 `language` 속성이 추가되었고, `language` 속성이 추가된 객체는 버려짐
- 타입스크립트는 `기본형`과 `객체 래퍼타입`을 별도로 모델링함
  - string / String
  - number / Number
  - boolean / Boolean
  - symbol / Symbol
  - bigint / BigInt

### `string`을 사용할 때 유의할 사항

- `string`을 `String`이라고 잘못 타이핑될 수 있으며, 실수를 하더라도 인지하지 못하는 경우가 발생(Human Error)

  ```tsx
  function getStringLen(foo: String) {
    return foo.length
  }

  getStringLen('hello') // 정상
  getStringLen(new String('hello')) // 정상
  ```

- 다음과 같이 처음에는 잘 동작하는 것처럼 보이지만 `string`을 매개변수로 받는 메서드에 `String` 객체를 전달하는 순간 문제가 발생함

  ```tsx
  function isGreeting(phrase: String) {
    return ['hello', 'good day'].includes(phrase)
    // ~~~~~~~~~~
    // 'String' 형식의 인수는
    // 'string' 형식의 매개변수에 할당될 수 없습니다.
    // 'string'은(는) 기본 개체이지만 'String'은(는) 래퍼 객체입니다.
    // 가능한 경우 'string'을(를) 사용하세요.
  }
  ```

  - `string`은 `String`에 할당할 수 있지만 `String`은 `string`에 할당할 수 없음
  - 대부분의 라이브러리와 마찬가지로 타입스크립트가 제공하는 타입 선언은 전부 기본형 타입으로 되어있음

- 래퍼 객체는 타입 구문의 첫 글자를 대문자로 표기하는 방법으로도 사용할 수 있음

  ```tsx
  const s: String = 'primitive'
  const n: Number = 12
  const b: Boolean = true
  ```

  - `런타임`의 값은 객체가 아니고 `기본형`이므로 객체 래퍼에 할당할 수 있기 때문에 타입스크립트는 기본형 타입을 객체 래퍼에 할당하는 선언을 허용함
  - 웬만하면 기본형 타입을 사용하는 것이 나음

## Item 11) 잉여 속성 체크의 한계 인지하기

- 타입이 명시된 변수에 객체 리터럴을 할당할 때 타입스크립트는 해당 타입의 속성이 있는지, 그 외의 속성은 없는지 확인함

  ```tsx
  interface Room {
    numDoors: number
    ceilingHeightFt: number
  }
  const r: Room = {
    numDoors: 1,
    ceilingHeightFt: 10,
    elephant: 'present',
    // ~~~~~~~~~~~~~~~~~~~ 객체 리터럴은 알려진 속성만 지정할 수 있으며
    //                     'Room' 형식에 'elephant'이(가) 없습니다.
  }
  ```

  - 구조적 타이핑 관점으로 생각해보면 오류가 발생하지 않아야하지만 발생하는 이유는 `잉여 속성 체크 과정`이 수행되었기 때문임
  - 잉여 속성 체크 과정은 조건에 따라 동작하지 않는 다는 한계가 있음
    - 할당 가능 검사와는 완전히 별도의 과정임을 인지해야 함

### 잉여 속성 체크

- 타입스크립트는 단순히 런타임에 예외를 던지는 코드에 오류를 표시하는 것뿐 아니라, 의도와 다르게 작성된 코드까지 찾으려 함

  ```tsx
  interface Options {
    title: string
    darkMode?: boolean
  }
  function createWindow(options: Options) {
    if (options.darkMode) {
      setDarkMode()
    }
    // ...
  }
  createWindow({
    title: 'Spider Solitaire',
    darkMode: true,
    // ~~~~~~~~~~~~~~~~~~~~~~~ 객체 리터럴은 알려진 속성만 지정할 수 있지만
    //                         'Options' 형식에 'darkmode'이(가) 없습니다.
    //                         'darkMode'을(를) 쓰려고 했습니까?
  })
  ```

  - `darkMode` 속성에 `boolean` 타입이 아닌 다른 타입의 값이 지정된 경우를 제외하면, `string` 타입인 `title` 속성과 또 다른 어떤 속성을 가지는 모든 객체는 `Options` 타입의 범위에 속함
  - 위의 코드는 런타임에 어떠한 종류의 오류를 발생하지 않음

    - 타입스크립트가 알려 주는 오류 메시지처럼 의도한 대로 동작하지 않을 수 있음
    - 오류가 발생한 부분은 `darkmode`가 아닌 `darkMode`(대문자 M)이어야 함

      ```tsx
      const o1: Options = document // 정상
      const o2: Options = new HTMLAnchorElement() // 정상
      ```

    - `document`와 `HTMLAnchorElement`의 인스턴스 모두 `string` 타입인 `title` 속성을 가지고 있기 때문에 할당문은 정상임
      - 하지만 객체 리터럴이 아니므로 잉여 속성 체크가 되지 않음

- `Options`는 넓은 범주의 타입이므로 잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써, 앞에서 다룬 `Room`이나 `Options` 예제 같은 문제점을 방지할 수 있음
  - `엄격한 객체 리터럴 체크`라고 불림

> 잉여 속성 체크는 타입 단언문을 사용할 때에 적용되지 않으므로, 타입 단언문보다 선언문을 사용해야 하는 단적인 이유가 됨

### 잉여 속성 체크 제외

- 인덱스 시그니처를 사용해서 타입스크립트가 추가적인 속성을 예상하도록 할 수 있음

  ```tsx
  interface Options {
    darkMode?: boolean
    [otherOptions: string]: unknown
  }
  const o: Options = { darkmode: true } // 정상
  ```

### 약한 타입(weak)에서 동작

```tsx
interface LineChartOptions {
  logscale?: boolean
  invertedYAxis?: boolean
  areaChart?: boolean
}
const opts = { logScale: true }
const o: LineChartOptions = opts
// ~ '{ logScale: boolean; }' 유형에
//   'LineChartOptions' 유형과 공통적인 속성이 없습니다.
```

- 구조적 관점에서 `LineChartOptions` 타입은 모든 속성이 선택적이므로 모든 객체를 포함할 수 있음
- 약한 타입에서는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행함
  - `공통 속성 체크`는 잉여 속성 체크와 다르게, 약한 타입과 관련된 할당문마다 수행됨

### 잉여 속성 체크의 구분과 한계점 인지

- 구조적 타이핑 시스템에서 허용되는 속성 이름의 오타 같은 실수를 잡는데 효과적이나 적용 `범위`가 매우 제한적이고 `객체 리터럴`에만 사용되는 한계점이 있음
- `잉여 속성 체크`와 일반적인 `타입 체크`의 개념을 잘 구분해야 함

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 53 ~ 64p
