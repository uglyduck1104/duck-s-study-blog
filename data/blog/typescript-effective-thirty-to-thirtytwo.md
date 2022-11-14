---
title: 타입 설계 - Item 30 ~ 32
date: '2022-11-10'
tags: ['typescript']
draft: false
summary: 문서에 타입 정보를 쓰지 않기, 타입 주변에 null 값 배치하기, 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기
layout: PostSimple
authors: ['default']
---

# 타입 설계 - Item 30 ~ 32

## Item 30) 문서에 타입 정보를 쓰지 않기

```typescript
/**
 * 전경색(foreground) 문자열을 반환합니다.
 * 0 개 또는 1 개의 매개변수를 받습니다.
 * 매개변수가 없을 때는 표준 전경색을 반환합니다.
 * 매개변수가 있을 때는 특정 페이지의 전경색을 반환합니다.
 */
function getFourgroundColor(page?: string) {
  return page === 'login' ? { r: 127, g: 127, b: 127 } : { r: 0, g: 0, b: 0 }
}
```

- 코드와 주석의 정보가 맞지 않고, 둘 중 어느 것이 옳은지 판단하기에는 정보가 부족하며 잘못된 상태라는 것은 분명함

### 문제점

1. 함수가 `string` 형태의 색깔을 반환한다고 적혀 있지만 실제로는 `{r, g, b}` 객체를 반환함
2. 주석에는 함수가 0개 또는 1개의 매개변수를 받는다고 설명하고 있지만, 타입 시그니처만 보아도 명확하게 알 수 있는 정보임
3. 불필요하게 장황하여 함수 선언과 구현체보다 주석이 더 김

### 타입스크립트 타입 구문 시스템

- 타입스크립트의 타입 구문 시스템은 간결하고, 구체적이며, 쉽게 읽을 수 있도록 설계되었으므로 함수의 입력과 출력만으로도 타입을 코드로 표현할 수 있음
- 누군가 강제하지 않는 이상 주석은 코드와 동기화되지 않지만 타입 구문은 타입스크립트 타입 체커가 타입 정보를 동기화하도록 강제함

### 주석 개선

```tsx
/** 애플리케이션 또는 특정 페이지의 전경색을 가져옵니다. */
function getFourgroundColor(page?: string): Color {
  // ...
}
```

- 특정 매개변수를 설명하고 싶다면 JSDoc의 `@param` 구문을 사용하여 명세할 수 있음
- 값을 변경하지 않는다고 설명하는 것도 좋지 않으며, 매개변수를 변경하지 않는다는 주석도 사용하지 않는 것이 좋음
  - `readonly`를 선언하여 타입스크립트가 규칙을 강제할 수 있게 하면 됨
- 주석에 적용한 규칙은 변수명에도 그대로 적용할 수 있으므로 변수명에 타입 정보를 넣지 않도록 하는 것이 좋음

## Item 31) 타입 주변에 null 값 배치하기

- `strictNullChecks` 설정을 키면 `null`이나 `undefined` 값 관련된 오류들이 나타나므로 오류를 걸러 내는 if 구문을 코드 전체에 추가해야 한다고 생각함
  - 어떤 변수가 `null`이 될 수 있는지 없는지를 타입만으로는 명확하기 표현하기 어려움
- 값이 전부 null이거나 전부 null이 아닌 경우로 분명히 구분된다면, 값이 섞여 있을 때보다 다루기 쉬움

  ```tsx
  function extent(nums: number[]) {
    let min, max
    for (const num of nums) {
      if (!min) {
        min = num
        max = num
      } else {
        min = Math.min(min, num)
        max = Math.max(max, num)
      }
    }
    return [min, max]
  }
  ```

  - 이 코드는 타입 체커를 통과하고, 반환 타입은 `number[]`으로 추론되나, 설계적 결함이 있음
    - 최솟값이나 최댓값이 0인 경우, 값이 덧씌워져 버림
    - `nums` 배열이 비어 있다면 함수는 `[undefined,` `undefined]`를 반환함
  - `undefined`를 포함하는 객체는 다루기 어렵고 절대 권장되는 사항이 아니므로 이러한 정보는 타입 시스템에서 표현할 수 없음

### `strictNullChecks` 활성화

```tsx
function extent(nums: number[]) {
  let min, max
  for (const num of nums) {
    if (!min) {
      min = num
      max = num
    } else {
      min = Math.min(min, num)
      max = Math.max(max, num)
      // 'number | undefined' 형식의 인수는 'number' 형식의 매개변수에 할당될 수 없습니다.
    }
  }
  return [min, max]
}
```

- extent의 반환 타입이 `(number | undefined)[]`로 추론되어서 설계적 결함이 분명해짐

  - extent를 호출하는 곳마다 타입 오류의 형태로 나타남

    ```tsx
    const [min, max] = extent([0, 1, 2])
    const span = max - min
    // 개체가 'undefined'인 것 같습니다.
    ```

  - extent 함수의 오류는 undefined를 min에서만 제외했고 max에서는 제외하지 않았기 때문에 발생함

### 코드 개선

```tsx
function extent(nums: number[]) {
  let result: [number, number] | null = null
  for (const num of nums) {
    if (!result) {
      result = [num, num]
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])]
    }
  }
  return result
}
```

- `min`과 `max`를 한 객체 안에 넣고, `null` 이거나 `null`이 아니게 함
- 반환 타입이 `[number, number] | null`이 되어서 사용하기가 더 수월해짐
- `null`이 아님 단언(`!`)을 사용하면 `min`과 `max`를 얻을 수 있음

```tsx
const [min, max] = extent([0, 1, 2])!
const span = max - min // 정상
```

- `null` 아님 단언 대신 단순 `if` 구문으로도 체크할 수 있음

```tsx
const range = extent([0, 1, 2])
if (range) {
  const [min, max] = range
  const span = max - min // 정상
}
```

> `extent`의 결괏값으로 단일 객체를 사용함으로써 설계를 개선했고, 타입스크립트가 `null` 값 사이의 관계를 이해할 수 있도록 했으며 버그도 제거함

### 클래스에서 `null`과 `null이 아닌 값`을 섞어서 사용하는 경우

```tsx
class UserPosts {
  user: UserInfo | null
  posts: Post[] | null

  constructor() {
    this.user = null
    this.posts = null
  }

  async init(userId: string) {
    return Promise.all([
      async () => (this.user = await fetchUser(userId)),
      async () => (this.posts = await fetchPostsForUser(userId)),
    ])
  }

  getUserName() {
    // ...?
  }
}
```

- 두 번의 네트워크 요청이 로드되는 동안 `user`와 `posts` 속성은 null 상태
- 어떤 시점에는 둘 다 `null`이거나, 둘 중 하나만 `null`이거나, 둘 다 `null`이 아닐 수 있음(총 4가지 경우)
  - `null` 체크가 난무해져 버그를 양산함

### 개선된 코드

```tsx
class UserPosts {
  user: UserInfo
  posts: Post[]

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user
    this.posts = posts
  }

  static async init(userId: string): Promise<UserPosts> {
    const [user, posts] = await Promise.all([fetchUser(userId), fetchPostsForUser(userId)])
    return new UserPosts(user, posts)
  }

  getUserName() {
    return this.user.name
  }
}
```

- `UserPosts` 클래스는 완전히 `null`이 아니게 되었고, 메서드를 작성하기 쉬어졌음
  - 데이터가 부분적으로 준비되었을 때 작업을 시작해야 한다면, `null`과 `null`이 아닌 경우의 상태를 다뤄야 함
- `Promise`는 데이터를 로드하는 코드를 단순하게 만들어 주지만, 데이터를 사용하는 클래스에서는 반대로 코드가 복잡해지므로 남용해서는 안됨

## Item 32) 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

- 유니온 타입의 속성을 가지는 인터페이스를 작성 중일때에는 인터페이스의 유니온 타입을 사용하는게 더 알맞는지에 대해 검토해 봐야 함

```tsx
interface Layer {
  layout: FillLayout | LineLayout | PointLayout
  paint: FillPaint | LinePaint | PointPaint
}
```

- `layout` 속성은 모양이 그려지는 방법과 위치를 제어하는 반면, `paint` 속성은 스타일을 제어함
- `layout`이 `LineLayout`임과 동시에 `paint` 속성이 `FillPaint` 타입인 것은 말이 되지 않음

  - 더 나은 방법으로 모델링하려면 각각 타입의 계층을 분리된 인터페이스로 둬야 함

    ```tsx
    interface FillLayer {
      layout: FillLayout
      paint: FillPaint
    }
    interface LineLayer {
      layout: LineLayout
      paint: LinePaint
    }
    interface PointLayer {
      layout: PointLayout
      paint: PointPaint
    }
    type Layer = FillLayer | LineLayer | PointLayer
    ```

  - 이런 형태로 `Layer`를 정의하면 `layout`과 `paint` 속성이 잘못된 조합으로 섞이는 경우를 방지할 수 있음

### 태그된 유니온 사용하기

- 태그된 유니온을 사용해 `Layer`의 경우 속성 중의 하나는 문자열 리터럴 타입의 유니온이 됨

  ```tsx
  interface Layer {
    type: 'fill' | 'line' | 'point'
    layout: FillLayout | LineLayout | PointLayout
    paint: FillPaint | LinePaint | PointPaint
  }
  ```

  - type: ‘fill’과 함께 LineLayout과 PointPaint 타입이 쓰이는 것은 말이 되지 않음
  - 이러한 경우를 방지하기 위해 Layer를 인터페이스의 유니온으로 변환할 수 있음

    ```tsx
    interface FillLayout {
      type: 'fill'
      layout: FillLayout
      paint: FillPaint
    }
    interface LineLayout {
      type: 'line'
      layout: LineLayout
      paint: LinePaint
    }
    interface PointLayout {
      type: 'point'
      layout: PointLayout
      paint: PointPaint
    }
    type Layer = FillLayer | LineLayer | PointLayer
    ```

    - `type` 속성은 태그이며 런타임에 어떤 타입의 `Layer`가 사용되는지 판단하는데 쓰임

  - 타입스크립트는 태그를 참고하여 `Layer`의 타입의 볌위를 좁힐 수도 있음

    ```tsx
    function drawLayer(layer: Layer) {
      if (layer.type === 'fill') {
        const { paint } = layer // 타입이 FillPaint
        const { layout } = layer // 타입이 FillLayout
      } else if (layer.type === 'line') {
        const { paint } = layer // 타입이 LinePaint
        const { layout } = layer // 타입이 LineLayout
      } else {
        const { paint } = layer // 타입이 PointPaint
        const { layout } = layer // 타입이 PointLayout
      }
    }
    ```

    - 타입스크립트가 코드의 정확성을 체크하지만, 타입 분기 후 `layer`가 포함된 동일한 코드 반복이 발생함

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 166 ~ 177p
