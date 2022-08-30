---
title: 타입스크립트의 타입 시스템 - Item 6 ~ 8
date: '2022-08-30'
tags: ['typescript']
draft: false
summary: 타입스크립트 컴파일러를 실행하는 것이 주된 목적이나, 타입스크립트 서버 또한 언어 서비스를 제공함
layout: PostSimple
authors: ['default']
---

# 타입스크립트의 타입 시스템 - Item 6 ~ 8

## Item 6) 에디터를 사용하여 타입 시스템 탐색하기

- 타입스크립트 설치 후 아래의 항목을 실행할 수 있음
  1. 타입스크립트 컴파일러(`tsc`)
  2. 단독으로 실행할 수 있는 타입스크립트 서버(`tsserver`)
- 타입스크립트 컴파일러를 실행하는 것이 주된 목적이나, 타입스크립트 서버 또한 `언어 서비스`를 제공함
  - `언어 서비스`
    - 코드 자동 완성
    - 명세(사양) 검사
    - 검색
    - 리팩토링
- 에디터에서 언어 서비스를 제공하는 것보다 타입스크립트 서버에 언어 서비스를 제공하도록 설정하는 것을 권장함

> 보통 에디터에서 마우스 오버를하면 타입스크립트가 타입을 어떻게 추론하는지 볼 수 있는데, 특히 조건문의 분기에서 값의 타입이 어떻게 변하는지 살펴보는 것은 타입 시스템을 연마하기 좋은 방법임

- 에디터가 제공하는 언어 서비스는 라이브러리와 라이브러리의 타입 선언을 탐색할 때 도움이 됨
  - `Go to Definition` 기능을 통해 타입스크립트의 타입 선언 파일로 이동할 수 있음
  - 타입 선언을 탐색하다 보면 처음에 이해하기는 어려웠지만 타입스크립트가 무엇을 하는지, 또 어떻게 라이브러리가 모델링되었는지 살펴볼 수 있는 좋은 기회가 됨

## Item 7) 타입이 값들의 집합이라고 생각하기

- `런타임`의 모든 변수는 자바스크립트 세상의 값으로부터 정해지는 각자의 고유한 값을 가짐
- 타입스크립트가 오류를 체크하는 순간에는 `타입`을 가지고 있음
  - `타입` - 할당 가능한 값들의 집합

### `never` Type

- 가장 작은 집합으로써, 아무 값도 포함하지 않는 공집합
- never 타입으로 선언된 변수의 범위는 `공집합`이므로 아무런 값도 할당할 수 없음

```tsx
const x: never = 12
// ~ '12' 형식은 'never' 형식에 할당할 수 없습니다.
```

### `literal` Type

- `never` Type 다음으로 작은 집합으로써, 한 가지 값만 포함하는 타입
- 유닛(`unit`) 타입이라고도 불림

  ```tsx
  type A = 'A'
  type B = 'B'
  type Twelve = 12
  ```

  ```tsx
  type AB = 'A' | 'B'
  type AB12 = 'A' | 'B' | 12
  ```

  - 두 개 혹은 세 개로 묶으려면 `유니온 타입`을 사용해 이어줌
  - 값 집합들의 `합집합`을 일컫음

- 다양한 타입스크립트 오류에서 `할당 가능한`이라는 문구를 볼 수 있는데 이는 집합의 관점으로 봤을때, `~이 원소(값과 타입의 관계)` 또는 `~의 부분 집합(두 타입의 관계)`를 의미함

  ```tsx
  const a: AB = 'A' // 정상
  const c: AB = 'C'
  // ~ '"C"' 형식은 'AB' 형식에 할당할 수 없습니다.
  ```

  - `C`는 유닛 타입이므로 단일 값 `C`로 구성되며, `AB`의 부분 집합이 아니므로 오류로 판단

### 타입 연산자

```tsx
interface Person {
  name: string
}
interface Lifespan {
  birth: Date
  death?: Date
}
type PersonSpan = Person & Lifespan

const ps: PersonSpan = {
  name: 'Alan Turing',
  birth: new Date('1912/06/23'),
  death: new Date('1954/06/07'),
} // 정상
```

- `&` 연산자는 두 타입의 인터섹션(intersection, `교집합`)을 계산함
- 타입 연산자는 인터페이스의 속성이 아닌, 값의 집합에 적용됨
  - `Person`과 `Lifespan`을 둘 다 가지는 값은 인터섹션 타입에 속하게 됨
- `Person`, `Lifespan` 두 인터페이스의 유니온 타입에 속하는 값은 어떠한 키도 없기 때문에, 유니온에 대한 `keyof`는 공집합이어야만 함

  ```text
  keyof (A&B) = (keyof A) | (keyof B)
  keyof (A|B) = (keyof A) & (keyof B)
  ```

- 일반적으로 `PersonSpan` 타입을 선언하려면 `extends` 키워드를 씀

  ```tsx
  interface Person {
    name: string
  }
  interface PersonSpan extends Person {
    birth: Date
    death?: Date
  }
  ```

  - `extends`의 의미는 `~의 부분 집합`이라는 의미로 받아들일 수 있음

    - `PersonSpan` 타입의 모든 값은 문자열 `name`, `birth` 속성을 가져야 제대로 부분 집합이 됨
    - 제네릭 타입에서 한정자로도 쓰이며, 여기서도 `~의 부분 집합`을 의미함

      ```tsx
      function getKey<K extends string>(val: any, key: K) {
        // ...
      }
      ```

      - string을 상속한다는 의미를 집합의 관점으로 생각해보면 아래와 같음

        ```tsx
        getKey({}, 'x') // 정상, 'x'는 string을 상속
        getKey({}, Math.random() < 0.5 ? 'a' : 'b') // 정상, 'a'|'b'는 string을 상속
        getKey({}, document.title) // 정상, string은 string을 상속
        getKey({}, 12)
        // ~~ '12' 형식의 인수는 'string' 형식의 매개변수에 할당될 수 없습니다.
        ```

- 타입의 집합이라는 관점은 `배열`과 `튜플`의 관계 역시 명확하게 만듦

  ```tsx
  const list = [1, 2]; // 타입은 nubmer[]
  cosnt tuple: [number, number] = list;
  	// ~~~~ 'number[]' 타입은 '[number, number]' 타입의 0, 1 속성에 없습니다
  ```

  - 숫자 배열을 숫자들의 쌍이라고 할 수 없는 것이 빈 리스트와 [1]이 반례임
  - `number[]`는 `[number, number]` 의 부분 집합이 아니기 때문에 할당할 수 없음

### 타입스크립트 타입에 되지 못하는 값들의 집합

- `정수`에 대한 타입, 또는 `x와 y 속성` 외에 다른 속성이 없는 객체는 타입스크립트 타입에 존재하지 않음

  ```tsx
  type T = Exclude<string | Date, string | number> // 타입은 Date
  type NonZeroNumbs = Exclude<number, 0> // 타입은 여전히 number
  ```

## Item 8) 타입 공간과 값 공간의 심벌 구분하기

- 타입스크립트의 `심벌`은 타입 공간이나 값 공간 중의 한 곳에 존재함
- 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있음

```tsx
interface Cylinder {
  radius: number
  height: number
}

const Cylinder = (radius: number, height: number) => ({ radius, height })
```

- `interface Cylinder`에서 `Cylinder`는 타입으로 쓰이고, 변수로 선언한 `Cylinder`와 이름은 같지만 값으로 쓰이며, 서로 아무런 관련도 없음
- 상황에 따라서 Cylinder는 타입으로 쓰일 수도 있고, 값으로 쓰일 수도 있기 때문에 가끔 오류를 야기함

  ```tsx
  function calculateVolume(shape: unknown) {
    if (shape instanceof Cylinder) {
      shape.radius
      // ~~~~~~ '{}' 형식에 'radius' 속성이 없습니다.
    }
  }
  ```

  - `instanceof`를 이용해 `shape`가 `Cylinder` 타입인지 체크하려고 했지만 `instanceof`는 자바스크립트의 런타임 연산자이므로 값에 대해서 연산을 함
    - `instanceof Cylinder`는 타입이 아니라 함수를 참조함

- 타입 선언(`:`) 또는 단언문(`as`) 다음에 나오는 심벌은 `타입`
- `=` 다음에 나오는 모든 것은 `값`

### typeof

- 타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 하는 연산자

```tsx
type T1 = typeof p // 타입은 Person
type T2 = typeof email
// 타입은 (p: Person, subject: string, body: string) => Response

const v1 = typeof p // 값은 "object"
const v2 = typeof email // 값은 "function"
```

- `타입`의 관점에서, `typeof`는 값을 읽어서 타입스크립트 타입을 반환함
- `값`의 관점에서 `typeof`는 자바스크립트 런타임의 typeof 연산자가 됨

### 타입 공간과 값 공간의 혼동

- `값`으로 쓰이는 this는 자바스크립트의 `this` 키워드이고, `타입`으로 쓰이는 this는 `다형성 this`라고 불리는 this의 타입스크립트 타입
- 값에서 `&`와 `|`는 `AND`와 `OR` 비트연산이지만 타입에서는 `인터섹션`과 `유니온`
- `const`는 새 변수를 선언한지만, `as const`는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꿈
- `extends`는 서브클래스(`class A extends B`) 또는 서브타입(`interface A extends B`), 제네릭 타입의 한정자(`Generic<T extends number>`)를 정의할 수 있음
- in은 루프(`for (key in object)`) 또는 매핑된(`mapped`) 타입에 등장함

### 타입 스크립트내의 구조 분해 할당

```tsx
function email({
	person: Person,
		// ~~~~~~~ 바인딩 요소 'Person'에 암시적으로 'any' 형식이 있습니다.
	subject: string,
		// ~~~~~~~ 'string' 식별자가 중복되었습니다.
		//         바인딩 요소 'string'에 암시적으로 'any' 형식이 있습니다.
	body: string
		// ~~~~~~~ 'string' 식별자가 중복되었습니다.
		//         바인딩 요소 'string'에 암시적으로 'any' 형식이 있습니다.
}){/*...*/}
```

- `값`의 관점에서 `Person`과 `string`이 해석되었기 때문에 오류가 발생함

```tsx
function email({ person, subject, body }: { person: Person; subject: string; body: string }) {
  //...
}
```

- `Person`이라는 변수명과 `string`이라는 이름을 가진 두 개의 변수를 생성하려 했기 때문에 문제를 해결하려면 `타입`과 `값`을 구분해야 함

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 33 ~ 52p
