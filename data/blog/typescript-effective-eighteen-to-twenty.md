---
title: 타입 추론 - Item 18 ~ 20
date: '2022-09-25'
tags: ['typescript']
draft: false
summary: 매핑된 타입을 사용하여 값을 동기화하기, 추론 가능한 타입을 사용해 장황한 코드 방지하기, 다른 타입에는 다른 변수 사용하기
layout: PostSimple
authors: ['default']
---

# 타입 추론 - Item 18 ~ 20

## Item 18) 매핑된 타입을 사용하여 값을 동기화하기

```tsx
interface ScatterProps {
  // The data
  xs: number[]
  ys: number[]

  // Display
  xRange: [number, number]
  yRange: [number, number]
  color: string

  // Events
  onClick: (x: number, y: number, index: number) => void
}
```

- 산점도를 그리기 위한 UI 컴포넌트를 작성하는 코드

### 최적화 첫 번째

```tsx
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== 'onClick') return true
    }
  }
  return false
}
```

- `conservative` 접근법, `fail close` 접근법
  - 새로운 속성이 추가되면 shouldUpdate 함수는 값이 변경될 때마다 차트를 다시 그림
  - 차트가 정확하지만 너무 자주 그려질 가능성이 있음

### 최적화 두번째

```tsx
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // (no check for onClick)
  )
}
```

- 차트를 불필요하게 다시 그리는 단점을 해결했지만, 차트를 다시 그려야하는 경우에 누락되는 일이 생길 수 있음

### 타입 체커로 해결

```tsx
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
}

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true
    }
  }
  return false
}
```

- `[k in keyof ScatterProps]`은 타입 체커에게 `REQUIRES_UPDATE`가 `ScatterProps`과 동일한 속성을 가져야 한다는 정보를 제공함
- 나중에 `ScatterProps`에 새로운 속성을 추가하는경우 다음 코드와 같은 형태가 됨

```tsx
interface ScatterProps {
  // ...
  onDoubleClick: () => void
}
```

- `REQUIRES_UPDATE`의 정의에 오류가 발생함

  ```tsx
  const REQUIRES_UPDATE: { [k in keyof ScatteProps]: boolean } = {
    // ~~~~~~~~~~~~~~~~~~ 'onDoubleClick' 속성이 타입에 없습니다.
    // ...
  }
  ```

- 해당 방식은 오류를 정확히 잡아내는데, 속성을 삭제하거나 이름을 바꾸어도 비슷한 오류가 발생함
- 여기서 `boolean` 값을 가진 객체를 사용했다는 점이 중요한데, 배열을 사용했다면 다음과 같은 코드가 됨

  ```tsx
  const PROPS_REQUIRING_UPDATE: (keyof ScatterProps)[] = [
    'xs',
    'ys',
    // ...
  ]
  ```

- 이처럼 매핑된 타입은 한 객체가 또 다른 객체와 정확히 같은 속성을 가지게 할 때 이상적임
- 이번 예제처럼 매핑된 타입을 사용해 타입스크립트가 코드에 제약을 강제하도록 할 수 있음

## Item 19) 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 타입스크립트가 타입을 위한 언어기때문에, 변수를 선언할 때마다 타입을 명시해야 한다고 오해를 하지만 타입스크립트의 많은 타입 구문은 사실 불필요함

### 타입 추론의 오용

- 타입 추론이 된다면 명시적 타입 구문을 필요하지 않음

```tsx
const person: {
  name: string
  born: {
    where: string
    when: string
  }
  died: {
    where: string
    when: string
  }
} = {
  name: 'Sojourner Truth',
  born: {
    where: 'Swartekill, NY',
    when: 'c.1797',
  },
  died: {
    where: 'Battle Creek, MI',
    when: 'Nov. 26, 1883',
  },
}
```

- 타입을 생략하고 다음처럼 작성해도 충분함

  ```tsx
  const person = {
    name: 'Sojourner Truth',
    born: {
      where: 'Swartekill, NY',
      when: 'c.1797',
    },
    died: {
      where: 'Battle Creek, MI',
      when: 'Nov. 26, 1883',
    },
  }
  ```

- 두 예제에서 person의 타입은 동일하며, 값에 추가로 타입을 작성하는 것은 거추장스러울 뿐임
- 배열의 경우에도 객체와 마찬가지로 타입스크립트는 입력 받아 연산을 하는 함수가 어떤 타입을 반환하는지 정확히 알고 있음

  ```tsx
  function square(nums: number[]) {
    return nums.map((x) => x * x)
  }
  const squares = square([1, 2, 3, 4]) // 타입은 number[]
  ```

- 이처럼 타입스크립트는 개발자가 예상한 것보다 더 정확하게 추론하기도 함

  ```tsx
  const axis1: string = 'x' // 타입은 string
  const axis2 = 'y' // 타입은 "y"
  ```

  - `axis2` 변수를 `string`으로 예상하기 쉽지만 타입스크립트가 추론한 `y`가 더 정확한 타입임

### 타입 추론을 통해 리팩터링 하기

```tsx
interface Product {
  id: number
  name: string
  price: number
}

function logProduct(product: Product) {
  const id: number = product.id
  const name: string = product.name
  const price: number = product.price
  console.log(id, name, price)
}
```

- id에 문자도 들어 있을 수 있음을 나중에 알게 되었다고 가정해서 Product 내의 id 타입으 변경

```tsx
interface Product {
  id: string
  name: string
  price: number
}

function logProduct(product: Product) {
  const id: number = product.id
  // ~~ 'string' 형식은 'number' 형식에 할당할 수 없습니다.
  const name: string = product.name
  const price: number = product.price
  console.log(id, name, price)
}
```

- `logProduct` 내의 `id` 변수 선언에 있는 타입과 맞지 않기 때문에 오류가 발생함
- `logProduct` 함수 내의 명시적 타입 구문이 없었다면, 코드는 아무런 수정 없이도 타입 체커를 통과함
- `logProduct`는 비구조화 할당문을 사용해 구현하는게 나음

  ```tsx
  function logProduct(product: Product) {
    const { id, name, price } = product
    console.log(id, name, price)
  }
  ```

  - 비구조화 할당문은 모든 지역 변수의 타입이 추론되도록 함

- 이상적인 타입스크립트 코드는 함수/메서드 시그니처에 타입 구문을 포함하지만, 함수 내에서 생성된 지역 변수에는 타입 구문을 넣지 않음
- 타입 구문을 생략하여 방행되는 것들을 최소화하고 코드를 읽는 사람이 구현 로직에 집중할 수 있게 하는 것이 좋음

### 함수 매개변수에 타입 구문을 생략하는 경우

```tsx
function parseNumber(str: string, base = 10) {
  // ...
}
```

- 기본값이 10이기 때문에 `base`의 타입은 `number`로 추론함
- 보통 타입 정보가 있는 라이브러리에서, 콜백 함수의 매개 변수 타입은 자동으로 추론됨

  ```tsx
  // Bad Case
  app.get('/health', (request: express.Request, response: express.Response) => {
    response.send('OK')
  })

  // Good Case
  app.get('/health', (request, response) => {
    response.send('OK')
  })
  ```

  - `express HTTP` 서버 라이브러를 사용하는 `request`와 `response`의 타입 선언은 필요하지 않음

### 타입 추론을 위해 타입을 명시하는 경우

```tsx
const elmo: Product = {
  name: 'Tickle Me Elmo',
  id: '048188 627152',
  price: 28.99,
}
```

- 이런 정의에 타입을 명시하면 잉여 속성 체크가 동작함
  - 선택적 속성이 있는 타입의 오타 같은 오류를 잡는 데 효과적이고 변수가 사용되는 순간이 아닌 할당하는 시점에 오류가 표시되도록 해줌
  - 만약 타입 구문을 제거한다면 잉여 속성 체크가 동작하지 않고, 객체를 선언한 곳이 아니라 객체가 사용되는 곳에서 타입 오류가 발생함
- 타입 구문을 제대로 명시한다면, 실제로 실수가 발생한 부분에 오류를 표시해줌

  ```tsx
  const furby: Product = {
    name: 'Furby',
    id: 630509430963,
    // ~~~ 'number' 형식은 'string' 형식에 할당할 수 없습니다.
    price: 35,
  }
  logProduct(furby)
  ```

### 함수의 반환에 타입 명시하기

- 반환 타입을 명시해야 하는 이유

  1. 오류의 위치를 제대로 표시해줌
  2. 반환 타입을 명시하면 함수에 대해 더욱 명확하게 알 수 있음

  - 반환 타입을 명시하려면 구현하기 전에 입력 타입과 출력 타입이 무엇인지 알아야 함
  - 미리 타입을 명시하는 방법은 함수를 구현하기 전에 테스트를 먼저 작성하는 `TDD`와 비슷함
  - 전체 타입 시그니처를 먼저 작성하고 구현에 맞추면 주먹구구식으로 시그니처가 작성되는 것을 방지할 수 있음

  3. 명명된 타입을 사용하기 위해서 명시해야 함

     ```tsx
     interface Vector2D {
       x: number
       y: number
     }
     function add(a: Vector2D, b: Vector2D) {
       return { x: a.x + b.x, y: a.y + b.y }
     }
     ```

  - 타입스크립트는 반환 타입을 `{ x: number; y: number; }`로 추론함
  - 이런 경우 `Vector2D`와 호환되지만, 입력이 `Vetor2D`인데 반해 출력은 `Vector2D`가 아니므로 사용자 입장에서 당황스러울 수 있음

- 반환 타입을 명시하면 더욱 직관적인 표현이 되고 반환 값을 별도의 타입으로 정의하면 타입에 대한 주석을 작성할 수 있어서 더욱 자세한 설명이 가능함

### 린터(linter) 설정하기

- `eslint` 규칙 중 `no-inferrable-type`을 사용해서 모든 타입 구문이 정말로 필요한지 확인할 수 있음

## Item 20) 다른 타입에는 다른 변수 사용하기

- 자바스크립트에서는 한 변수를 다른 목적을 가지는 다른 타입으로 재사용해도 됨

  ```tsx
  let id = '12-34-56'
  fetchProduct(id) // string으로 사용
  id = 123456
  fetchProductBySerialNumber(id) // number로 사용
  ```

- 반면 타입스크립트에서는 두 가지 오류가 발생함

  ```tsx
  let id = '12-34-56'
  fetchProduct(id)

  id = 123456
  // ~ '123456' 형식은 'string' 형식에 할당할 수 없습니다.
  fetchProductBySerialNumber(id)
  // ~~ 'string' 형식의 인수는
  //    'number' 형식의 매개변수에 할당될 수 없습니다.
  ```

  - 타입스크립트는 `12-34-56` 이라는 값을 보고, `id`의 타입을 `string`으로 추론함
  - `string` 타입에는 `number` 타입을 할당할 수 없기 때문에 오류가 발생함
  - `변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다`는 중요한 관점을 알 수 있음

### 유니온 타입으로 해결하기

```tsx
let id: string | number = '12-34-56'
fetchProduct(id)
id = 123456 // 정상
fetchProductBySerialNumber(id) // 정상
```

- 타입스크립트는 첫 번째 함수 호출에서 `id`는 string으로, 두 번째 호출에서는 `number`라고 제대로 판단함
  - 할당문에서 유니온 타입으로 범위가 좁혀짐
- 유니온 타입으로 코드가 동작하긴 하지만 id를 사용할 때마다 값이 어떤 타입인지 확인해야 하기 때문에 유니온 타입은 `string`이나 `numer` 같은 간단한 타입에 비해 다루기 더 어려움

### 별도의 변수로 할당하기

```tsx
const id = '12-34-56'
fetchProduct(id)

const serial = 123456 // 정상
fetchProductBySerialNumber(serial) // 정상
```

- 별도의 변수를 사용하는 게 더 바람직한 이유
  - 서로 관련이 없는 두개의 값을 분리함(`id`와 `serial`)
  - 변수명을 더 구체적으로 지을 수 있음
  - 타입 추론을 향상시키며, 타입 구문이 불필요해짐
  - 타입 추론을 향상시키며, 타입 구문이 불필요해짐
  - 타입이 좀 더 간결해짐(`string|number` 대신 `string`과 `number`를 사용).
  - `let` 대신 `const`로 변수를 선언하게 됨
    - `const`로 변수를 선언하면 코드가 간결해지며 타입 체커가 타입을 추론하기에도 좋음
- 타입이 바뀌는 변수는 되도록 피해야 하며, 목적이 다른 곳에는 별도의 변수명을 사용해야 함

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 101 ~ 118p
