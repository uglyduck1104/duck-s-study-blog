---
title: 타입 선언과 @types - Item 51 ~ 코드를 작성하고 실행하기 - Item 53
date: '2023-01-17'
tags: ['typescript']
draft: false
summary: 의존성 분리를 위해 미러 타입 사용하기, 테스팅 타입의 함정 주의하기, 타입스크립트 기능보다는 ECMAScript 기능을 사용하기
layout: PostSimple
authors: ['default']
---

# 타입 선언과 @types - Item 51 ~ 코드를 작성하고 실행하기 - Item 53

## Item 51) 의존성 분리를 위해 미러 타입 사용하기

```tsx
function parseCSV(contents: string | Buffer): { [column: string]: string }[] {
  if (typeof contents === 'object') {
    // 버퍼인 경우
    return parseCSV(contents.toString('utf8'))
  }
  // ...
}
```

- CSV 파일의 내용을 매개변수로 받고, 열 이름을 값으로 매핑하는 객체들을 생성하여 배열로 반환하는 함수

  - `Buffer`의 타입 정의는 `NodeJS` 타입 선언을 설치

    ```bash
    $ npm install -D @types/node
    ```

- 앞에서 작성한 CSV 파싱 라이브러리를 공개하면 타입 선언도 포함하게 되므로 `@types/node`를 `devDependencies`로 포함해야 함
  - 다음의 두 그룹의 라이브러리 사용자들에게 문제점이 발생함
    - `@types`와 무관한 자바스크립트 개발자
    - `NodeJS`와 무관한 타입스크립트 웹 개발자
- 각자가 필요한 모듈만 사용할 수 있도록 구조적 타이핑을 적용하여 필요한 메서드와 속성만 별도로 작성하는 방법으로 해결할 수 있음

  ```tsx
  interface CsvBuffer {
    toString(encoding: string): string
  }
  function parseCSV(contents: string | CsvBuffer): { [column: string]: string }[] {
    // ...
  }
  ```

  - `CsvBuffer`는 `Buffer` 인터페이스보다 훨씬 짧으면서도 실제로 필요한 부분만을 떼어 내어 명시

- 작성 중인 라이브러리가 의존하는 라이브러리의 구현과 무관하게 타입에만 의존한다면, 필요한 선언부만 추출하여 작성 중인 라이브러리에 넣는 것을 고려하는 것이 좋음
  - 웹 기반이나 자바스크립트 등 다른 모든 사용자에게는 더 나은 사양을 제공할 수 있음

## Item 52) 테스팅 타입의 함정 주의하기

- 프로젝트 공개에 앞서 테스트 코드를 작성하는 것은 필수이며, 타입 선언도 테스트를 거쳐야 하지만 타입 선언을 테스트하기는 매우 어려움
  - 타입 선언에 대한 테스트 코드를 작성할 때 단언문으로 떼우기 십상이지만, 이러한 방법은 몇 가지 문제를 야기함
  - `dtslint` 또는 타입 시스템 외부에서 타입을 검사하는 유사한 도구를 사용하는 것이 더 안전하고 간단함

### 예시

- 유틸리티 라이브러리에서 제공하는 map 함수의 타입 선언

```tsx
declare function map<U, V>(array: U[], fn: (u: U) => V): V[]
```

- 타입 선언이 예상한 타입으로 결과를 내는지 체크할 수 있는 한 가지 방법은 함수를 호출하는 테스트 파일을 작성하는 것

  ```tsx
  map(['2017, '2018', '2019'], v => Number(v));
  ```

  - `map`의 첫번째 매개변수에 배열이 아닌 단일 값이 있었다면 매개변수의 타입에 대한 오류는 잡을 수 있으나 반환값에 대한 체크가 누락되어 있으므로 완전한 테스트라 할 수 없음

    ```tsx
    test('square a number', () => {
      square(1)
      square(2)
    })
    ```

  - 함수의 런타임 동작을 테스트한다고 가정했을때, `square` 함수의 실행에서 오류가 발생하지 않는지만 체크하므로 실제로는 결과(반환 값)에 대한 테스트라고 볼 수 없음
    - 따라서 `square`의 구현이 잘못되어 있더라도 해당 테스트는 통과함

- 타입 선언언 파일을 테스팅할 때는 단순히 함수를 실행만하는 방식을 일반적으로 적용하게 되는데, 의미가 없지는 않지만 반환 타입을 체크하는 것이 훨씬 더 좋은 코드라고 할 수 있음

  ```tsx
  const lengths: number[] = map(['john', 'paul'], (name) => name.length)
  ```

  - 일반적으로 불필요한 타입선언이라고 볼 수 있으나, 테스트 코드 관점에서는 중요한 역할을 하고 있음
  - `number[]` 타입 선언은 `map` 함수의 반환 타입이 `number[]`임을 보장함

    - 하지만, 테스팅을 위해 할당을 사용하는 방법에는 두 가지 근본적인 문제가 있음

      1. 불필요한 변수 생성

         - 일부 린팅 규칙을 비활성화하거나, 변수를 도입하는 대신 헬퍼 함수를 정의

           ```tsx
           function assertType<T>(x: T) {}
           assertType<number[]>(map(['john', 'paul'], (name) => name.length))
           ```

      2. 두 타입이 동일한지 체크하는 것이 아니라 할당 가능성을 체크하고 있음

         ```tsx
         const n = 12
         assertType<number>(n) // 정상
         ```

         - n 심벌을 확인해보면, 타입이 실제로 숫자 리터럴 타입인 12임을 볼 수 있으나 객체의 타입을 체크하는 경우를 살펴보면 문제가 발견됨

           ```tsx
           const beatles = ['john', 'paul', 'george', 'ringo']
           assertType<{ name: string }[]>(
             map(beatles, (name) => ({
               name,
               inYellowSubmarin: name === 'ringo',
             }))
           ) // 정상
           ```

           - `map`은 `{name: string, inYellowSubmarin: boolean}` 객체의 배열을 반환하고 반환된 배열은 `{name: string}[]`에 할당 가능하나 `inYellowSubmarine` 속성에 대한 부분이 체크되지 않음

         - `assertType`에 함수를 넣어 보면 이상한 결과가 나타남

           ```tsx
           const add = (a: number, b: number) => a + b
           assertType<(a: number, b: number) => number>(add) // 정상

           const double = (x: number) => 2 * x
           assertType<(a: number, b: number) => number>(double) // 정상!?
           ```

           - `double` 함수의 체크가 성공하는 이유는, 타입스크립트의 함수는 매개변수가 더 적은 함수 타입에 할당 가능하기 때문임
           - 타입스크립트는 선언보다 많은 수의 매개변수 동작을 모델링하도록 설계되어 있기 때문에 적은 매개변수를 가진 함수를 할당하는 것이 아무런 문제가 없음

### 더 나은 설계

- 제대로 된 assertType을 사용하기 위해 Parameters와 ReturnType 제너릭 타입을 이용해 함수의 매개변수 타입과 반환 타입만 분리하여 테스트할 수 있음

```tsx
const double = (x: number) => 2 * x;
let p: Parameters<typeof double> = null!;
assertType<[number, number>(p);
// '[number]' 형식의 인수는 '[number, number]'
// 형식의 매개변수에 할당될 수 없습니다.
let r: ReturnType<typeof double> = null!;
assertType<number>(r); // 정상
```

- `map`의 매개변수로 배열을 넣어 함수를 실행하고 반환 타입을 테스트했지만, 중간 단계의 세부 사항은 테스트하지 않았음
- 세부 사항을 테스트하기 위해서 콜백 함수 내부에서 매개변수들의 타입과 `this`를 직접 체크할 수 있음

  ```tsx
  const beatles = ['john', 'paul', 'george', 'ringo']
  assertType<number[]>(
    map(beatles, function (name, i, array) {
      assertType<string>(name)
      assertType<number>(i)
      assertType<string[]>(array)
      assertType<string[]>(this)
      // this에는 암시적으로 any 형식이 포함됩니다.
      return name.length
    })
  )
  ```

  - 해당 예제의 콜백 함수는 화살표 함수가 아니므로 this의 타입을 테스트할 수 있음을 주의해야 함

    - 다음 코드의 선언을 사용하면 타입 체크를 통과함

      ```tsx
      declare function map<U, V>(array: U[], fn: (this: U[], u: U, i: number, array: U[]) => V): V[]
      ```

- 다음 모듈 선언은 까다로운 테스트를 통과할 수 있는 완전한 타입 선언 파일이지만, 결과적으로 좋지 않은 설계가 됨

```tsx
declare module 'overbar'
```

- 이 선언은 전체 모듈에 `any` 타입을 할당하므로 테스트는 전부 통과하지만 모든 타입 안전성을 포기하게 됨
- 타입 시스템 내에서 암묵적인 `any` 타입을 발견하는 것이 어려우므로 타입체커와 독립적으로 동작하는 도구를 사용해 타입 선언을 테스트하는 방법을 권장함

### dtslint

- `DefinitelyTyped`의 타입 선언을 위한 도구로써, 특별한 형태의 주석을 통해 동작함

```tsx
const beatles = ['john', 'paul', 'george', 'ringo']
map(
  beatles,
  function (
    name, // $ExpectType string
    i, // $ExpectType number
    array // $ExpectType string[]
  ) {
    this // // $ExpectType string[]
    return name.length
  }
) // $ExpectType number[]
```

- `dtslint`는 할당 가능성을 체크하는 대신 각 심벌의 타입을 추출하여 글자 자체가 같은지 비교함
- 편집기에서 타입 선언을 눈으로 보고 확인하는 것과 같은데, `dtslint`는 이러한 과정을 자동화하지만 글자 자체가 같은지 비교하는 방식에는 단점이 있음
  - `number|string`, `string|number`는 같은 타입이나 글자 자체로 보면 다르므로 서로 다른 타입으로 인식됨(`string`, `any`도 동일함)

### 정리

- 타입 선언을 테스트한다는 것은 어렵지만 반드시 해야하는 작업으로써 일반적인 기법의 문제점을 인지하고 문제점을 방지하기 위해 `dtslint` 같은 도구를 사용하는 것을 권장함

## Item 53) 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

- 자바스크립트는 결함이 많고 개선해야 할 부분이 많은 언어였으며, 클래스, 데코레이터, 모듈 시스템 같은 기능이 없어서 프레임워크나 트랜스파일러로 보완하는 것이 일반적인 모습이었음
- 시간이 흐르면서 TC39(자바스크립트를 관장하는 표준 기구)는 부족했던 점들을 내장 기능으로 추가함
  - 자바스크립트에 새로 추가된 기능은 타입스크립트 초기 버전에서 독립적으로 개발했던 기능과 호환성 문제를 발생시킴

### 타입스크립트의 전략

- 자바스크립트의 신규 기능을 그대로 채택하고 타입스크립트 초기 버전과 호환성을 포기
- TC39는 런타임 기능을 발전 시키고, 타입스크립트 팀은 타입 기능만 발전시킨다는 명확한 원칙을 세우고 현재까지 지켜옴
- 위의 원칙이 세워지기 전에 타입 공간과 값 공간의 경계를 혼란스럽게 만드는 몇 가지 기능들이 존재함

### 열거형(enum)

- 타입스크립트의 열거형은 다음 목록처럼 상황에 따라 다르게 동작함
  - 숫자 열거형에 0, 1, 2 외의 다른 숫자가 할당되면 매우 위험함
  - 상수 열거형은 보통의 열거형과 달리 런타임에 완전히 제거됨
- 타입스크립트의 일반적인 타입들이 할당 가능성을 체크하기 위해서 구조적 타이핑을 사용하는 반면, 문자열 열거형은 명목적 타이핑을 사용

  ```tsx
  enum Flavor {
    VANILLA = 'vanilla',
    CHOCOLATE = 'chocolate',
    STRAWBERRY = 'strawberry',
  }

  let flavor = Flavor.CHOCOLATE // 타입이 Flabor
  flavor = 'strawberry'
  // 'strawberry' 형식은 'Flavor' 형식에 할당될 수 없습니다.
  ```

- Flavor를 매개변수로 받는 함수로 가정했을때, Flavor는 런타임 시점에는 문자열이므로 자바스크립트에서 다음처럼 호출할 수 있음

  ```tsx
  function scoop(flavor: Flavor) {/*...*/}
  scoop('vanilla'); // 자바스크립트에서 정상
  ```

  - 그러나 타입스크립트에서는 열거형을 임포트하고 문자열 대신 사용해야 함

    ```tsx
    scoop('vanilla')
    // 'vanilla' 형식은 'Flavor' 형식의 매개변수에 할당될 수 없습니다.

    import { Flavor } from 'ice-cream'
    scoop(Flavor.VANILLA) // 정상
    ```

- 이처럼 자바스크립트와 타입스크립트에서 동작이 다르므로 문자열 열거형은 사용하지 않는 것이 좋음
- 대신 열거형 리터럴 타입의 유니온 사용을 권장

  ```tsx
  type Flavor = 'vanilla' | 'chocolate' | 'strawberry'

  let flavor: Flavor = 'chocolate' // 정상
  flavor = 'mint chip'
  // 'mint chip' 형식은 'Flavor' 형식의 매개변수에 할당될 수 없습니다.
  ```

  - 리터럴 타입의 유니온은 열거형만큼 안전하며 자바스크립트와 호환되는 장점이 있음

### 매개변수 속성

- 클래스를 초기화할 때 속성을 할당하기 위해 생성자의 매개변수를 사용

```tsx
class Person {
  constructor(public name: string) {}
}
```

- public name은 매개변수 속성이라고 불리며 더 간결한 문법을 제공하지만 몇 가지 문제점이 존재함
  - 일반적으로 타입스크립트 컴파일은 타입 제거가 이루어지므로 코드가 줄어들지만, 매개변수 속성은 코드가 늘어나는 문법
  - 매개변수 속성이 런타임에는 실제로 사용되나, 타입스크립트 관점에서는 사용되지 않는 것 처럼보임
  - 매개변수 속성과 일반 속성을 섞어서 사용하면 클래스 설계가 혼란스러워짐

```tsx
class Person {
  first: string
  last: string
  constructor(public name: string) {
    [this.first, this.last] = name.split(' ');
  }
}
```

- Person 클래스에는 세 가지 속성(first, last, name)이 있지만, first와 last만 속성에 나열되어 있고 name은 매개변수 속성에만 존재해서 일관성이 깨짐
  - 클래스 대신 인터페이스로 만들고 객체 리터럴을 사용하는 것이 좋음

```tsx
class Person {
  constructor(public name: string) {}
}
const p: Person = { name: 'Jed Bartlet' } // 정상
```

- 매개변수 속성과 일반 속성을 같이 사용하면 설계가 혼란스러워지므로 한 가지만 사용하는 것이 혼선을 방지하는 데 유리함

### 네임스페이스와 트리플 슬래시 임포트

- 타입스크립트 자체의 모듈 시스템인 `module` 키워드와 트리플 슬래시 임포트는 `ECMAScript 2015`와의 충돌을 피하기 위해 `namespae` 키워드를 추가함

```tsx
namespace foo {
  function bar() {}
}
/// <reference path="other.ts"/>
foo.bar()
```

- 트리플 슬래시 임포트와 `module` 키워드는 호환성을 위한 잔재로써, `ECMAScript 2015` 스타일의 모듈(`import`, `export`)을 사용해야 함

### 데코레이터

- 클래스, 메서드, 속성에 `annotation`을 붙이거나 기능을 추가하는 데 사용함

```tsx
class Gretter {
  greeting: string
  constructor(message: string) {
    this.greeting = message
  }
  @logged
  greet() {
    return 'Hello, ' + this.greeting
  }
}

function logged(target: any, name: string, descriptor: PropertyDescriptor) {
  const fn = target[name]
  descriptor.value = function () {
    console.log(`Calling ${name}`)
    return fn.apply(this, arguments)
  }
}

console.log(new Gretter('Dave').greet())
// 출력:
// Calling greet
// Hello, Dave
```

- 데코레이터는 앵귤러 프레임워크를 지원하기 위해 추가되었으며, `tsconfig.json`에 `experimetalDecorators` 속성을 설정하고 사용해야함
- 현재까지 표준화가 완료되지 않았으므로, 작성한 코드가 깨질 우려가 있음

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 251 ~ 267p
