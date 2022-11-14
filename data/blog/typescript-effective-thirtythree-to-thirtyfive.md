---
title: 타입 설계 - Item 33 ~ 35
date: '2022-11-14'
tags: ['typescript']
draft: false
summary: string 타입보다 더 구체적인 타입 사용하기, 부정확한 타입보다는 미완성 타입을 사용하기, 데이터가 아닌, API와 명세를 보고 타입 만들기
layout: PostSimple
authors: ['default']
---

# 타입 설계 - Item 33 ~ 35

## Item 33) string 타입보다 더 구체적인 타입 사용하기

- string 타입은 범위는 매우 넓으므로 변수를 선언하려면 더 좁은 타입이 적절하지 않은지 검토해야 함

### 문제 인식

```tsx
interface Album {
  artist: string
  title: string
  releaseDate: string // YY-MM-DD
  recordingType: string // 예를 들어, "live" 또는 "studio"
}
```

- string 타입이 남발된 모습이고 주석을 통해 의미를 추론해야하는 것으로 보아 인터페이스 설계가 잘못되었다는 것을 알 수 있음

```tsx
const kindOfBlue: Album = {
  artist: 'Miles Davis',
  title: 'Kind of Blue',
  releaseDate: 'August 17th, 1959', // 날짜 형식이 다름
  recordingType: 'Studio', // 오타 (대문자 S)
} // 정상
```

- `releaseDate` 필드의 값은 주석에 설명된 형식과 다르며, `recordingType` 필드의 값 `Studio`는 소문자 대신 대문자가 쓰임

  - 그러나 이 두 값 모두 문자열이고, 해당 객체는 `Album` 타입에 할당 가능하며 타입 체커를 통과함
  - 또한 `string` 타입의 범위가 넓기 때문에 제대로 된 `Album` 객체를 사용하더라도 매개변수 순서가 잘못된 것이 오류로 드러나지 않음

    ```tsx
    function recordRelease(title: string, date: string) {
      /* ... */
    }
    recordRelease(kindOfBlue.releaseDate, kindOfBlue.title) // 오류여야 하지만 정상
    ```

  - `recordRelease` 함수의 호출에서 매개변수들의 순서가 바뀌었으나, 둘 다 문자열이기 때문에 타입 체커가 정상으로 인식함
  - 앞의 예제처럼 `string` 타입이 남용된 코드를 `문자열을 남발하여 선언되었다`고 표현함

### 개선된 필드 선언

```tsx
type RecordingType = 'studio' | 'live'

interface Album {
  artist: string
  title: string
  releaseDate: Date
  recordingType: RecoringType
}
```

- `releaseDate` 필드는 `Date` 객체를 사용해서 날짜 형식으로만 제한함
- `recordingType` 필드는 `live`와 `studio` 단 두개의 값으로 유니온 타입을 정의할 수 있음

### 장점

1. 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지됨

   ```tsx
   function getAlbumsOfType(recordingType: string): Album[] {
     // ...
   }
   ```

- getAlbumsOfType 함수를 호출하는 곳에서 recoringType의 값이 string타입이어야 한다는 것 외에는 다른 정보가 없음
- 주석으로 써놓은 `studio` 또는 `live`는 `Album` 정의에 숨어 있고, 함수를 사용하는 사람은 `recording Type`이 `studio` 또는 `live`여야 한다는 것을 알 수 없음

2. 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있음

   ```tsx
   /** 이 녹음은 어떤 환경에서 이루어졌는지 ? */
   type RecordingType = 'live' | 'studio'
   ```

- `getAlbumsOfType`이 받는 매개변수를 `string` 대신 `RecordingType`으로 바꾸면, 함수를 사용하는 곳에서 `RecordingType`의 설명을 볼 수 있음

3. `keyof` 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해짐

   ```tsx
   function pluck<T>(records: T[], key: string): any[] {
     return records.map((r) => r[key])
     // '{}' 형식에 인덱스 시그니처가 없으므로
     // 요소에 암시적으로 'any' 형식이 있습니다.
   }
   ```

- 언더스코어 라이브러리의 `pluck`라는 함수를 예로 들었을 때, 타입 체크가 되긴 하지만 `any` 타입이 있어서 정밀하지 못함
- 특히 반환 값에 `any`를 사용하는 것은 매우 좋지 않은 설계임
- 타입스크립트는 `key`의 타입이 `string`이기 때문에 범위가 너무 넓다는 오류를 발생시킴
- `string`을 `keyof T`로 바꾸는 타입 체커를 통과함

  - 또한 타입스크립트가 반환 타입을 추론할 수 있게 해줌

    ```tsx
    function pluck<T>(records: T[], key: keyof T): T[keyof T][]{}
    ```

- `T[keyof T]`는 `T` 객체 내의 가능한 모든 값의 타입임
- `keyof T`는 `string`에 비하면 훨씬 범위가 좁기는 하지만 여전히 넓음
- 범위를 더 좁히기 위해 `keyof T`의 부분 집합으로 두 번째 제너릭 매개변수를 도입해야 함

  ```tsx
  function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
    return records.map((r) => r[key])
  }
  ```

> `string` 은 `any`와 비슷한 문제를 가지고 있으므로 잘못 사용하게 되면 무효한 값을 허용하고 타입 간의 관계도 감춰버림.
> `string`의 부분 집합을 정의할 수 있는 기능을 통해 타입 안정성과 오류 방지 및 코드의 가독성을 향상 시킬 수 있음

## Item 34) 부정확한 타입보다는 미완성 타입을 사용하기

- 타입 선언을 작성하다 보면 코드의 동작을 더 구체적으로 또는 덜 구체적으로 모델링하게 되는 상황을 맞닥뜨리게 됨
- 타입 선언의 정밀도를 높이는 일에 실수가 발생하기 쉽고 잘못된 타입 설계로 인한 부작용을 경계해야 함

### 예시 - 맵박스 라이브러리

- `JSON`으로 정의된 `Lisp`와 비슷한 언어의 타입 선언을 작성하는 경우

```tsx
12
"red"
["+", 1, 2] // 3
["/", 20, 2] // 10
["case", [">", 20, 10], "red", "blue"] // "red"
["rgb", 255, 0, 127] // "#FF007F"
```

- 맵박스 라이브러리는 해당 시스템을 사용하여 수많은 기기에서 지도 기능의 형태를 결정함
- 입력값의 전체 종류
  1. 모두 허용
  2. 문자열, 숫자, 배열 허용
  3. 문자열, 숫자, 알려진 함수 이름으로 시작하는 배열 허용
  4. 각 함수가 받는 매개변수의 개수가 정확한지 확인
  5. 각 함수가 받는 매개변수의 타입이 정확한지 확인
- 처음의 두개 옵션은 간단함
  - `type Expression1 = any;`
  - `type Expression2 = number | string | any[];`
- 표현식의 유효성을 체크하는 테스트 세트를 도입해 정밀도가 손상되는 것을 방지

  ```tsx
  const tests: Expression2[] = [
    10,
    'red',
    true,
    // ~~~~'true' 형식은 'Expression2' 형식에 할당할 수 없습니다.
    ['+', 10, 5],
    ['case', ['>', 20, 10], 'red', 'blue', 'green'], // 값이 너무 많습니다.
    ['**', 2, 31], // "**"는 함수가 아니므로 오류가 발생해야 합니다.
    ['rgb', 255, 128, 64],
    ['rgb', 255, 0, 127, 0], // 값이 너무 많습니다.
  ]
  ```

### 정밀도 개선 - 유니온 사용

- 정밀도를 한 단계 더 끌어 올리기 위해 튜플의 첫 번째 요소에 문자열 리터럴 타입의 유니온을 사용

```tsx
type FnName = '+' | '-' | '*' | '/' | '>' | '<' | 'case' | 'rgb'
type CallExpression = [FnName, ...any[]]
type Expression3 = number | string | CallExpression

const tests: Expression3[] = [
  10,
  'red',
  true,
  // ~~~~ 'true' 형식은 'Expression3' 형식에 할당할 수 없습니다.
  ['+', 10, 5],
  ['case', ['>', 20, 10], 'red', 'blue', 'green'],
  ['**', 2, 31],
  // ~~~~~~~~~~~~~ '"**"' 형식은 'FnName' 형식에 할당할 수 없습니다.
  ['rgb', 255, 128, 64],
]
```

- 정밀도를 유지하면서 오류를 하나 더 잡음

> 각 함수의 매개변수 개수가 정확한지 확인하기 위해 모든 함수 호출을 확인할 수도 있지만 재귀적으로 동작하므로 좋은 방법은 아님

### 호출 표현식 - 인터페이스 나열

- 여러 인터페이스를 호출 표현식으로 한번에 묶을 수 없으므로, 각 인터페이스를 나열해 호출 표현식 작성

```tsx
type Expression4 = number | string | CallExpression

type CallExpression = MathCall | CaseCall | RGBCall

interface MathCall {
  0: '+' | '-' | '*' | '/' | '>' | '<'
  1: Expression4
  2: Expression4
  length: 3
}

interface CaseCall {
  0: 'case'
  1: Expression4
  2: Expression4
  3: Expression4
  length: 4 | 6 | 8 | 10 | 12 | 14 | 16 // 등등
}

interface RGBCall {
  0: 'rgb'
  1: Expression4
  2: Expression4
  3: Expression4
  length: 4
}

const tests: Expression4[] = [
  10,
  'red',
  true,
  // ~~~~ 'true' 형식은 'Expression4' 형식에 할당할 수 없습니다.
  ['+', 10, 5],
  ['case', ['>', 20, 10], 'red', 'blue', 'green'],
  // '["case", [">", ...], ...]' 형식은 'string' 형식에 할당할 수 없습니다.
  ['**', 2, 31],
  // ~~~~~~~~~~~~~ 'number' 형식은 'string' 형식에 할당할 수 없습니다.
  ['rgb', 255, 128, 64][('rgb', 255, 128, 64, 73)],
  // 'number' 형식은 'string' 형식에 할당할 수 없습니다.
]
```

- 이전 코드보다 \*\*에 대한 오류는 이전 버전보다 더 부정확해지며, 타입 정보가 더 정밀해졌으나 결과적으로 개선되었다고 보기 어려움
- 새 타입 선언은 더 구체적이지만 자동 완성을 방해하므로 타입스크립트 개발 경험을 해치게 됨
- 타입 선언의 복잡성으로 인해 버그가 발생할 가능성도 높아짐

> 부정확함을 바로잡는 방법을 쓰는 대신, 테스트 세트를 추가하여 놓친 부분이 없는지 확인해야 함. 타입 정보를 구체적으로 만들수록 오류 메시지와 자동 완성 기능에 주의를 기울여야 함

## Item 35) 데이터가 아닌, API와 명세를 보고 타입 만들기

- 파일 형식, API, 명세 등 우리가 다루는 타입 중 최소한 몇 개는 프로젝트 외부에서 비롯된 것이므로 명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있도록 도와줌

### 예시 - API 호출

- GraphQL API는 타입스크립트와 비슷한 타입 시스템을 사용하여, 가능한 모든 쿼리와 인터페이스를 명세하는 스키마로 이루어짐
- 이러한 인터페이스를 사용해 특정 필드를 요청하는 쿼리를 작성함

```graphql
query {
  repository(owner: "Microsoft", name: "TypeScript") {
    createdAt
    description
  }
}
```

```json
{
  "data": {
    "repository": {
      "createdAt": "2014-06-17T15:28:39Z",
      "description": "TypeScript is a superset of JavaScript that compiles to JavaScript."
    }
  }
}
```

- GraphQL의 장점은 특정 쿼리에 대해 타입스크립트 타입을 생성할 수 있음

```graphql
query getLicense($owner: String!, $name: String!) {
  repository(owner: $owner, name: $name) {
    description
    licenseInfo {
      spdxId
      name
    }
  }
}
```

- `$owner`와 `$name`은 타입이 정의된 `GraphQL` 변수임
- `String`은 `GraphQL`의 타입이고, 타입스크립트에서는 `string`이 됨

### Apollo GraphQL to TypeScript

- `Apollo` 도구를 통해 `GraphQL` 쿼리를 타입스크립트 타입으로 변환할 수 있음

```bash
$ apollo client: codegen \
		--endpoint https://api.github.com/graphql \
		--includes license.graphql \
		--target typescript
Loading Apollo Project
Generating query files with 'typescript' target - wrote 2 files
```

- 쿼리에서 타입을 생성하려면 `GraphQL` 스키마가 필요한데 `Apollo`는 `api.github.com/graphql`로부터 스키마를 얻을 수 있음

```tsx
export interface getLicense_repository_licenseInfo {
	__typename: "License";
	/** Short identifier specified by <https://spdx.org/licenses> */
	spdxId: string | null;
	/** The license full name specified by <https://spdx.org/license> */
	name: string;
}

export interface getLicense_repository {
	__typename: "Repository";
	/** The description of the repository. */
	description: string | null;
	/** the license associated with the repository */
	licenseInfo: getLicense_repository_licenseInfo | null;
}

export interface getLicense {
	/** Lookup a given repository by the owner and repository name. */
	repository: getLicense_repository | null;
}

export interface getLicenseVariables {
	owner: string;
	name: string;
}
```

- 쿼리 매개변수(`getLicenseVariable`)와 응답(`getLicense`) 모두 인터페이스가 생성됨
- `null` 가능 여부는 스키마로부터 응답 인터페이스로 변환됨
- 편집기에서 확인할 수 있도록 주석은 `JSDoc`으로 변환됨
- 자동으로 생성된 타입 정보는 `API`를 정확히 사용할 수 있도록 도와주고 쿼리가 바뀌면 타입도 자동으로 바뀌며, 스키마가 바뀐다면 타입도 자동으로 바뀜
  - 타입은 단 하나의 원천 정보인 `GraphQL` 스키마로부터 생성되기 때문에 타입과 실제 값이 항상 일치함

> 명세 정보나 공식 스키마가 없다면 데이터로부터 타입을 생성해야 하고 생성된 타입은 실제 데이터와 일치하지 않을 수 있음

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 178 ~ 194p
