---
title: 타입스크립트의 타입 시스템 - Item 15 ~ 17
date: '2022-09-12'
tags: ['typescript']
draft: false
summary: 동적 데이터에 인덱스 시그니처 사용하기, number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기, 변경 관련된 오류 방지를 위해 readonly 사용하기 
layout: PostSimple
authors: ['default']
---

# 타입스크립트의 타입 시스템 - Item 15 ~ 17

## Item 15) 동적 데이터에 인덱스 시그니처 사용하기

### 인덱스 시그니처의 오용

- 자바스크립트는 객체를 생성하는 문법이 간단하지만, 문자열 키를 타입의 값에 관계없이 매핑함
- 타입스크립트에서는 타입에 `인덱스 시그니처`를 명시하여 유연하게 매핑을 표현할 수 있음

  ```tsx
  type Rocket = {[property: string]: string};
  const rocket: Rocket = {
  	name: 'Falcon 9',
  	variant: 'v1.0',
  	thrust '4,940 kN',
  }; // 정상
  ```

  - `[property: string]: string` 이 인덱스 시그니처이며, 세 가지 의미를 담고 있음
    1. 키의 이름: 키의 위치만 표시하는 용도로서, 타입 체커에서는 사용하지 않음
    2. 키의 타입: `string`이나 `number` 또는 `symbol`의 조합이어야 하지만, 보통은 `string`을 사용함
    3. 값의 타입: 어떤 것이든 될 수 있음
  - 타입 체크가 수행되면 네 가지 단점이 드러남
    1. 잘못된 키를 포함해 모든 키를 허용함
    - `name` 대신 `Name`으로 작성해도 유효한 `Rocket` 타입이 됨
    2. 특정 키가 필요하지 않음
    - `{}`도 유효한 Rocket 타입
    3. 키마다 다른 타입을 가질 수 없음
    - `thrust`는 `string`이 아니라 `number`여야 할 수도 있음
    4. 타입스크립트 언어 서비스는 다음과 같은 경우에 도움이 되지 못함
    - `name:`을 입력할 때, 키는 무엇이든 가능하기 때문에 자동 완성 기능이 동작하지 않음

### 인터페이스로 개선

- 위의 인덱스 시그니처는 부정확하므로 더 나은 방법인 인터페이스로 개선

```tsx
interface Rocket {
  name: string
  variant: string
  thrust_kN: number
}
const falconHeavy: Rocket = {
  name: 'Falcon Heavy',
  variant: 'v1',
  thrust_kN: 15_200,
}
```

- `thrusk_kN`은 `number` 타입이며, 타입스크립트는 모든 필수 필트가 존재하는지 확인

### 인덱스 시그니처의 사용

- 동적 데이터를 표현할 때 주로 사용함
- CSV 파일처럼 헤더 행(`row`)에 열(`column`)이름이 있고, 데이터 행을 열 이름과 값으로 매핑하는 객체로 나타내고 싶은 경우

  ```tsx
  function parseCSV(input: string): { [columnName: string]: string }[] {
    const lines = input.split('\n')
    const [header, ...rows] = lines
    const headerColumns = header.split(',')
    return rows.map((rowStr) => {
      const row: { [columnName: string]: string } = {}
      rowStr.split(',').forEach((cell, i) => {
        row[headerColumns[i]] = cell
      })
      return row
    })
  }
  ```

  - 일반적인 상황에서 열 이름이 무엇인지 미리 알 방법은 없으므로 이럴 때는 인덱스 시그니처를 사용

    ```tsx
    interface ProductRow {
      productId: string
      name: string
      price: string
    }

    declare let csvData: string
    const products = parseCSV(csvData) as unknown as ProductRow[]
    ```

  - 선언해 둔 열들이 런타임에 실제로 일치한다는 보장은 없으므로 값 타입에 `undefined`를 추가할 수 있음

    ```tsx
    function saveParseCSV(input: string): { [columnName: string]: string | undefined }[] {
      return parseCSV(input)
    }
    ```

  - 모든 열의 `undefined` 여부를 체크해야 함

    ```tsx
    const rows = parseCSV(csvData)
    const prices: { [product: string]: number } = {}
    for (const row of rows) {
      prices[row.productId] = Number(row.price)
    }

    const safeRows = safeParseCSV(csvData)
    for (const row of safeRows) {
      prices[row.productId] = Number(row.price)
      // ~~~~~~~~~~~~~~~ 'undefined' 형식을 인덱스 형식으로 사용할 수 없습니다.
    }
    ```

  - 체크를 추가해야 하기에 작업이 조금 번거롭지만 `undefined`를 타입에 추가할지는 상황에 맞게 판단해야 함

### 인덱스 시그니처 모델링

- 어떤 타입에 가능한 필드가 제한되어 있는 경우라면 인덱스 시그니처로 모델링하지 말아야 함
- 데이터에 A, B, C, D 같은 키가 있지만, 얼마나 많이 있는지 모른다면 `선택적 필드` 또는 `유니온 타입`으로 모델링하면 됨

  ```tsx
  interface Row1 {
    [column: string]: number
  }
  interface Row2 {
    a: number
    b?: number
    c?: number
    d?: number
  }
  type Row3 =
    | { a: number }
    | { a: number; b: number }
    | { a: number; b: number; c: number }
    | { a: number; b: number; c: number; d: number }
  ```

  - `interface Row1` → 너무 광범위 함
  - `interface Row2` → 최선
  - `type Row3` → 가장 정확하지만 사용하기 번거로움

> string 타입이 너무 광범위해서 인덱스 시그니처를 사용하는 데 문제가 있다면, 두 가지 다른 대안을 생각해 볼 수 있음

1. `Record`를 사용

- 키 타입에 유연성을 제공하는 제너릭 타입
- 특히 string의 부분 집합을 사용할 수 있음

  ```tsx
  type Vec3D = Record<'x' | 'y' | 'z', number>
  // Type Vec3D = {
  //   x: number;
  //   y: number;
  //   z: number;
  // }
  ```

2. 매핑된 타입을 사용

- 매핑된 타입은 키마다 별도의 타입을 사용하게 함

  ```tsx
  type Vec3D = { [k in 'x' | 'y' | 'z']: number }
  // Type Vec3D = {
  //   x: number;
  //   y: number;
  //   z: number;
  // }
  type ABC = { [k in 'a' | 'b' | 'c']: k extends 'b' ? string : number }
  // Type ABC = {
  //   a: number;
  //   b: string;
  //   c: number;
  // }
  ```

## Item 16) number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

- 자바스크립트는 `암시적 타입 강제`와 같은 타 언어 대비 이상한 동작을 하는 부분이 존재함

  ```
  > "0" == 0
  true
  ```

  - 암시적 타입 강제와 관련된 문제는 대부분 `===`와 `!==`를 사용해서 해결이 가능함

- 자바스크립트 객체 모델에도 이상한 동작을 야기하는 부분들이 있으며, 객체 모델을 이해하는 것이 타입스크립트 타입 시스템 모델링을 이해하는데 중요한 키포인트임
- 자바스크립트에서 객체는 키/값 쌍의 모음으로 보통, 키는 `문자열`이며 값은 어떤 것이든 될 수 있음

### 자바스크립트의 객체 표현

- 자바스크립트에는 `해시 가능` 객체 표현이 없음

  - `toString` 메서드가 호출되어 객체가 문자열로 변환됨

    ```
    > x = {}
    {}
    > x[[1, 2, 3]] = 2
    2
    > x
    { '1,2,3': 1 }
    ```

- 숫자는 키로 사용할 수 없음

  - 숫자를 사용하려하면 자바스크립트 런타임은 문자열로 변환함

    ```
    > { 1: 2, 3: 4 }
    { '1': 2, '3': 4 }
    ```

- 배열은 분명히 객체임

  - 그러므로 숫자 인덱스를 사용하는 것이 당연함

    ```
    > typeof []
    'object'
    > x = [1, 2, 3]
    [ 1, 2, 3 ]
    > x[0]
    1
    ```

  - 앞의 인덱스들은 문자열로 변환되어 사용됨, 문자열 키를 사용해도 역시 배열의 요소에 접근할 수 있음

    ```
    > x['1']
    2
    ```

  - Object.keys를 이용해 배열의 키를 나열해 보면, 키가 문자열로 출력됨

    ```
    > Object.keys(x)
    ['0', '1', '2']
    ```

### 타입스크립트의 객체 표현

- 타입스크립트는 이러한 혼란을 바로잡기 위해 숫자 키를 허용하고, 문자열 키와 다른 것으로 인식함

  ```tsx
  interface Array<T> {
    // ...
    [n: number]: T
  }
  ```

- 타입 스크립트 타입 시스템의 다른 것들과 마찬가지로, 타입 정보는 런타임에 제거되지만 `Object.keys`와 같은 구문은 여전히 문자열로 반환됨

  ```tsx
  const keys = Object.keys(xs) // 타입이 string[]
  for (const key in xs) {
    key // 타입이 string
    const x = xs[key] // 타입이 number
  }
  ```

- 인덱스에 신경 쓰지 않는다면 `for-of`을 권장

  ```tsx
  for (const x of xs) {
    x // 타입이 number
  }
  ```

- 인덱스의 타입이 중요하다면, `number` 타입을 제공해 줄 `Array.prototype.forEach`를 사용

  ```tsx
  xs.forEach((x, i) => {
    i // 타입이 number
    x // 타입이 number
  })
  ```

- 루프 중간에 멈춰야 한다면, C 스타일인 for(;;) 루프 사용

  ```tsx
  for (let i = 0; i < xs.length; i++) {
    const x = xs[i]
    if (x < 0) break
  }
  ```

> 타입이 불확실하다면, `for-in` 루프는 `for-of` 또는 `C` 스타일 `for` 루프에 비해 몇 배나 느림

### ArrayLike 사용

- 어떤 길이를 가지는 배열과 비슷한 형태의 튜플을 사용하고 싶다면 타입스크립트에 있는 `ArrayLike` 타입을 사용해야 함

```tsx
function checkedAccess<T>(xs: ArrayLike<T>, i: number): T {
  if (i < xs.length) {
    return xs[i]
  }
  throw new Error(`배열의 끝을 지나서 ${i}를 접근하려고 했습니다.`)
}
```

- 실제로 이런 경우는 드물기는 하지만 필요하다면 `ArrayLike`를 사용해야 함
- `ArrayLike`를 사용하더라도 키는 여전히 문자열임

  ```tsx
  const tupleLike: ArrayLike<string> = {
    '0': 'A',
    '1': 'B',
    length: 2,
  } // 정상
  ```

## Item 17) 변경 관련된 오류 방지를 위해 readonly 사용하기

### 문제 인식

```tsx
function printTriangles(n: number) {
  const nums = []
  for (let i = 0; i < n; i++) {
    nums.push(i)
    console.log(arraySum(nums))
  }
}
```

```tsx
console.log(printTriangles(5))
// 0
// 1
// 2
// 3
// 4
```

- `arraySum`이 `nums`을 변경하지 않는다고 간주해서 문제가 발생함

### 문제 해결

```tsx
function arraySum(arr: number[]) {
  let sum = 0,
    num
  while ((num = arr.pop()) !== undefined) {
    sum += num
  }
  return sum
}
```

- `arraySum` 함수는 배열 안의 숫자들을 모두 합치지만, 계산이 끝나면 원래 배열이 모두 비게됨
  - 자바스크립트 배열은 내용을 변경할 수 있으므로 타입스크립트 역시 오류 없이 통과함

### readonly 접근 제어자 사용

- `arraySum`이 배열을 변경하지 않는다는 선언을 `readonly` 접근 제어자를 통해 오류의 범위를 좁힐 수 있음

  ```tsx
  function arraySum(arr: readonly number[]) {
    let sum = 0,
      num
    while ((num = arr.pop()) !== undefined) {
      // ~~~ 'readonly number[]' 형식에 'pop' 속성이 없습니다.
      sum += num
    }
    return sum
  }
  ```

  - `readonly number[]`는 타입이고 `number[]`와 구분되는 몇 가지 특징이 있음
    - 배열의 요소를 읽을 수 있지만, 쓸 수는 없음
    - `length`를 읽을 수 있지만, 바꿀 수는 없음
    - 배열을 변경하는 `pop`을 비롯한 다른 메서드를 호출할 수 없음
  - `number[]`는 `readonly number[]`보다 기능이 많으므로, `readonly number[]`의 서브타입이 됨

    - 따라서 변경 가능한 배열을 `readonly` 배열에 할당할 수 있지만, 반대의 경우는 불가능함

      ```tsx
      const a: number[] = [1, 2, 3]
      const b: readonly number[] = a
      const c: number[] = b
      // ~ 'readonly number[]' 타입은 'readonly'이므로
      // 변경 가능한 'number[]' 타입에 할당될 수 없습니다.
      ```

  - 매개 변수를 `readonly`로 선언하면 다음과 같은 일이 생김
    - 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크함
    - 호출하는 쪽에서 함수가 매개변수를 변경하지 않는다는 보장을 받게 됨
    - 호출하는 쪽에서 함수에 `readonly` 배열을 매개변수로 넣을 수 있음

- `readonly` 접근 제어자를 사용해 배열을 변경하지 않는 쪽으로 수정

  ```tsx
  function arraySum(arr: readonly number[]) {
    let sum = 0
    for (const num of arr) {
      sum += num
    }
    return sum
  }
  ```

  ```tsx
  console.log(printTriangles(5))
  // 0
  // 1
  // 3
  // 6
  // 10
  ```

  - 함수가 매개변수를 변경하지 않는다면 `readonly`로 선언해 의도치 않은 변경을 방지해야 함
  - 어떤 함수를 `readonly`로 만드려면, 그 함수를 호출하는 다른 함수도 모두 `readonly`로 만들어야 함
  - 다른 라이브러리에 있는 함수를 호출하는 경우라면, 타입 선언을 바꿀 수 없으므로 타입 단언문(`as number[]`)을 사용해야 함

- `readonly`는 얕게(`shallow`) 동작함
  - 현재 시점에는 깊은(`deep`) `readonly` 타입이 기본으로 지원되지 않지만, 제너릭을 만들면 깊은 `readonly` 타입을 사용할 수 있음
  - 제너릭은 만들기 까다롭기 때문에 라이브러리를 사용하는 것을 권장
    - ex) `ts-essentials`에 있는 `DeepReadonly` 제너릭 사용
- 인덱스 시그니처에도 readonly를 쓸 수 있음

  - 읽기는 허용하되 쓰기를 방지하는 효과가 있음

    ```tsx
    let obj: {readonly [k: string]: number} = {};
    // 또는 Readonly<{[k: string]: number}
    obj.hi = 45;
    // ~~ ... 형식의 인덱스 시그니처는 읽기만 허용됨
    obj = {...obj, hi: 12}; // 정상
    obj = [...obj, bye: 34}; // 정상
    ```

  - 해당 코드처럼 인덱스 시그니처에 `readonly`를 사용하면 객체의 속성이 변경되는 것을 방지할 수 있음

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 84 ~ 101p
