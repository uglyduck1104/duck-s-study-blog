---
title: any 다루기 - Item 42 ~ 44
date: '2022-12-18'
tags: ['typescript']
draft: false
summary: 모르는 타입의 값에는 any 대신 unknown을 사용하기, 몽키 패치보다는 안전한 타입을 사용하기, 타입 커버리지를 추적하여 타입 안전성 유지하기
layout: PostSimple
authors: ['default']
---

# any 다루기 - Item 42 ~ 44

## Item 42) 모르는 타입의 값에는 any 대신 unknown을 사용하기

- `unknown`
  - 함수의 반환값과 관련된 형태, 변수 선언과 관련된 형태, 단언문과 관련된 형태로 나뉘어져 있음

### 반환값과 관련된 형태

- `YAML`(`JSON`은 물론 `JSON` 문법의 상위집합까지 표현가능함)
- `parseYAML` 함수 예제

  ```tsx
  function parseYAML(yaml: string): any {
    //...
  }
  ```

  - `JSON.parse`의 반환 타입과 동일하게 `parseYAML` 메서드의 반환 타입을 `any`로 만듦
  - 함수의 반환 타입은 `any`로 사용하는 것을 자제해야 함

    - `parseYAML`을 호출한 곳에서 반환값을 원하는 타입으로 할당하는 것을 권장

      ```tsx
      interface Book {
        name: string
        author: string
      }
      const book: Book = parseYAML(`
        name: Wuthering Heights
        author: Emily Bronte
      `)
      ```

    - 함수의 반환값에 타입 선언을 강제하지 못하므로 호출되는 곳에서 타입 선언을 생략하면 book 변수는 암시적 any 타입이 되고, 사용되는 곳마다 타입오류가 발생함

      ```tsx
      const book: Book = parseYAML(`
        name: Jane Eyre
        author: Charlotte Bronte
      `)
      alert(book.title) // 오류 없음, 런타임에 'undefined' 경고
      book('read') // 오류 없음, 런타임에 'TypeError: book은 함수가 아닙니다' 예외 발생
      ```

    - parseYAML이 unknow 타입을 반환하게 만드는 것이 더 안전함

      ```tsx
      function safeParseYAML(yaml: string): unknonw {
        return parseYAML(yaml)
      }
      const book = safeParseYAML(`
        name: The Tenant of Wildfell Hall
        author: Anne Bronte
      `)
      alert(book.title)
      // 개체가 'unknown' 형식입니다.
      book('read')
      // 개체가 'unknown' 형식입니다.
      ```

- `any`가 강력하면서도 위험한 이유는 다음과 같음
  - 어떠한 타입이든 `any` 타입에 할당 가능(`unknown` 타입에 만족)
  - `any` 타입은 어떠한 타입으로도 할당 가능(`never` 타입에 만족)
- 타입 체커는 집합 기반이므로 `any`를 사용하면 타입 체커가 무용지물이 됨

> unknown은 any 대신 쓸 수 있는 타입 시스템에 부합하는 타입임

- unknown 타입은 값으로 단독으로 사용할 수 없으므로, 적적한 타입으로 변환하도록 강제해야 함

  ```tsx
  const book: Book = safeParseYAML(`
  	name: Villette
  	author: Charlotte Bronte
  `) as Book
  alert(book.title)
  // 'Book' 형식에 'title' 속성이 없습니다.
  book('read')
  // 이 식은 호출할 수 없습니다.
  ```

  - `Book` 타입 기준으로 타입 체크가 되므로, `unknown` 타입 기준으로 오류를 표시했던 예제보다 오류의 정보가 더 정확함

### 변수 선언과 관련된 형태

```tsx
interface Feature {
  id?: string | number
  geometry: Geometry
  properties: unknown
}
```

- 타입 단언문이 `unknown`에서 원하는 타입으로 변환하는 유일한 방법은 아님
- `interfaceof`를 체크한 후 `unknown`에서 원하는 타입으로 변환할 수 있음

  ```tsx
  function processValue(val: unknown) {
    if (val instanceof Date) {
      val // 타입이 Date
    }
  }
  ```

- 사용자 정의 타입 가드도 `unknown`에서 원하는 타입으로 변환할 수 있음

  ```tsx
  function isBook(val: unknown): val is Book {
    return typeof val === 'object' && val !== null && 'name' in val && 'author' in val
  }
  function processValue(val: unknown) {
    if (isBool(val)) {
      val // 타입이 Book
    }
  }
  ```

  - `unknown` 타입의 범위를 좁히기 위해서는 상당한 노력을 요함
    - `in` 연산자에서 오류를 피하기 위해 먼저 `val`이 객체임을 확인
    - `typeof null === 'object'` 이므로 별도의 `val`이 `null` 아님을 확인

- 가끔 `unknown` 대신 제네릭 매개변수를 사용하는 경우도 있음

  ```tsx
  function safeParseYAML<T>(yaml: string): T {
    return parseYAML(yaml)
  }
  ```

  - 타입 단언문과 달라 보이지만 기능적으로는 동일함
  - 제네릭보다는 `unknown`을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋음

### 단언문과 관련된 형태

```tsx
declare const foo: Foo
let barAny = foo as any as Bar
let barUnk = foo as unknown as Bar
```

- `barAny`와 `barUnk`는 기능적으로 동일하지만, 추후에 두 개의 단언문을 분리하는 리팩토링을 한다면 `unknown` 형태가 더 안전함
  - `unknown` 형태는 `any`와 다르게 분리되는 즉시 오류를 발생하므로 안전함

### unknown과 유사하지만 다른 타입

- `{}` 타입은 `null`과 `undefined`를 제외한 모든 값을 포함
- `object` 타입은 모든 비기본형(`non-primitive`) 타입으로 일어짐
  - `true` 또는 `12` 또는 `"foo"` 가 포함되지 않지만 객체와 배열은 포함됨

> `null`과 `undefined`가 불가능하다고 판단되는 경우만 `unknown` 대신 `{}`를 사용

## Item 43) 몽키 패치보다는 안전한 타입을 사용하기

### 자바스크립트 몽키 패치

- 자바스크립트 특징 중 가장 유명한 것은 객체와 클래스에 임의의 속성을 추가할 수 있을 만큼 유연하기 때문에, 가끔 예기지 못한 결과를 초래하기도 함

```tsx
RegExp.prototype.monkey = 'Capuchin'
// "Capuchin"
/123/.monkey
// "Capuchin"
```

- 정규식(/123/)에 `monkey`라는 속성을 추가한 적이 없는데 `Capuchin`이라는 값이 들어 있음
- 객체에 임의의 속성을 추가하는 것은 일반적으로 좋은 설계가 아니고, 사이드 이펙트의 위험성을 가지고 있음(ex. window, DOM 노드 …)

### 타입스크립트 몽키 패치

- 타입스크립트 또한 `any` 단언문을 오용하게 되면 타입 안전성을 상실하고, 언어 서비스를 사용할 수 없음

  ```tsx
  ;(document as any).monky = 'Tamarin' // 정상, 오타
  ;(document as any).monkey = /Tamarin/ // 정상, 잘못된 타입
  ```

- 최선의 해결책은 `document` 또는 `DOM`으로부터 데이터를 분리해야 함
- 분리할 수 없는 경우

  1. `interface`의 특수 기능 중 하나인 보강(`augmentation`)을 사용

     ```tsx
     interface Document {
       /** 몽키 페치의 속(genus) 또는 종(species) */
       monkey: string
     }

     document.monkey = 'Tamarin' // 정상
     ```

  - 장점
    - 타입이 더 안전해지고 타입 체커는 오타나 잘못된 타입의 할당을 오류로 표시함
    - 속성에 주석을 붙일 수 있음
    - 속성에 자동완성 사용 가능
    - 몽키 패치가 어떤 부분에서 적용되었는지 정확한 기록이 남음
  - 단점

    - 모듈의 관점에서 제대로 동작하게 하려면 global 선언을 추가해야 함

      ```tsx
      export {}
      declare global {
        interface Document {
          /** 몽키 페치의 속(genus) 또는 종(species) */
          monkey: string
        }
      }
      document.monkey = 'Tamarin' // 정상
      ```

      - 보강은 전역적으로 적용되므로 코드의 다른 부분이나 라이브러리로부터 분리할 수 없으며, 애플리케이션이 실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법이 없음

  2. 구체적인 타입 단언문의 사용

     ```tsx
     interface MonkeyDocument extends Document {
       /** 몽키 페치의 속(genus) 또는 종(species) */
       monkey: string
     }
     ;(document as MonkeyDocument).monkey = 'Macaque'
     ```

  - `MonkeyDocument`는 `Document`를 확장하기 때문에 타입 단언문은 정상이며 할당문의 타입은 안전함
  - 또, `Document` 타입을 건드리지 않고 별도록 확장하는 새로운 타입을 도입했기 때문에 모듈 영역 문제도 해결할 수 있음(`import`하는 곳의 영역에만 해당)

- 몽키 패치된 속성을 참조하는 경우에만 단언문을 사용하거나, 새로운 변수를 도입함
- 몽키 패치를 남용해서는 안되며 궁극적으로 더 잘 설계된 구조로 리팩토링하는 것이 좋음

## Item 44) 타입 커버리지를 추적하여 타입 안전성 유지하기

- `noImplicitAny`를 설정하고 모든 암시적 any 대신 명시적 타입 구문을 추가해도 `any` 타입과 관련된 문제들로부터 안전할 수 없음
  1. 명시적 `any` 타입
  - `any` 타입의 범위를 좁히고 구체적으로 만들어도 여전히 `any` 타입임
  - 특히 `any[]`와 `{[key : string] : any}` 같은 인덱스를 생성하면 단순 `any`가 되고 코드 전반에 영향을 미침
  2. 서드파티 타입 선언
  - `@types` 선언 파일로부터 `any` 타입이 전파되기 때문에 특별히 조심해야 함
- `any` 타입은 타입 안전성과 생산성에 부정적 영향을 미칠 수 있으므로, `any`의 개수를 추적하는 것이 좋음

  - npm의 type-coverage 패키지를 활용해서 any를 추적할 수 있는 몇 가지 방법이 있음

    ```bash
    $ npx type-coverage
    9985 / 10117 98.69%
    ```

    - 10,117개 심벌 중 9,985개가 `any`가 아니거나 `any`의 별칭이 아닌 타입을 가지고 있음을 알 수 있음

  - `—detail` 플래그를 붙이면, `any` 타입이 있는 곳을 모두 출력해 줌

    ```bash
    $ npx type-coverage --detail
    path/to/code.ts:1:10 getColumnInfo
    path/to/module.ts:7:1 pt2
    ...
    ```

    - 타입 커버리지 정보를 수집하는 데 유용하게 사용할 수 있음

- 표 형태의 데이터에서 어떤 종류의 열 정보를 만들어 내는 함수

  ```tsx
  function getColumnInfo(name: string): any {
    return utils.buildColumnInfo(appState.dataSchema, name) // any 반환
  }
  ```

  - `getColumnInfo` 함수의 반환에는 주석과 함꼐 명시적으로 : any 구분을 추가함
  - 타입 정보를 추가하기 위해 `ColumnInfo` 타입을 정의하고 `utils.buildColumnInfo`가 `any` 대신 `ColumnInfo`를 반환하도록 개선해도 `getColumnInfo` 함수의 반환문에 있는 `any` 타입이 모든 타입 정보를 날려 버리게 됨
    - `getColumnInfo`에 남아 있는 `any`까지 제거해야 문제가 해결됨

- 서드파티 라이브러리로부터 비롯되는 any 타입은 몇 가지 형태로 등장할 수 있으며 가장 극단적인 예는 전체 모듈에 any 타입을 부여하는 것임

  - `declare module 'my-module';`

    - `my-module`에서 어떤 것이든 오류 없이 임포트할 수 있으나 모든 심벌은 `any` 타입이고, 임포트한 값이 사용되는 곳마다 `any` 타입을 양산 함

      ```tsx
      import { someMethod, someSymbol } from 'my-module' // 정상

      const pt1 = {
        x: 1,
        y: 2,
      } // 타입이 {x: number, y: number}
      const pt2 = someMethod(pt1, someSymbol) // 정상, pt2의 타입이 any
      ```

  - 가끔 사용하는 모듈을 점검하거나 모듈을 충분히 이해한 후에 타입선언을 직접 작성해야 함

- 타입 커버리지를 추적하면 이러한 부분들을 쉽게 발견할 수 있으므로 코드를 꾸준히 점검할 필요가 있음

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 147 ~ 166p
