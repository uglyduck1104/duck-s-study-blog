---
title: Mock Service Worker(MSW)로 서버 응답 시뮬레이션
date: '2022-08-24'
tags: ['Jest']
draft: false
summary: Mock Service Worker 설정 방법과 사용 예시
layout: PostSimple
authors: ['default']
---

# Mock Service Worker(MSW)로 서버 응답 시뮬레이션

## OrderEntry 서버 데이터 소개

### 테스트 개요

- 옵션 이미지가 렌더링되는지 테스트
- Mock Server Worker를 이용해 모킹하기
- Mock Server 응답
  - `/scoops` 테스트 코드 작성
  - `/toppings` 테스트 코드 작성

## Mock Service Worker Server 설정

### MSW 설치

```bash
npm install msw
```

### 핸들러 생성

- 특정 URL과 라우트에 무엇을 반환할지 결정하는 함수

  <img alt="jest_msw_1" src="/static/images/jest_msw_1.png" width="700"/>

- `src/mocks/handlers.js`

```jsx
  import { rest } from 'msw'

  export const handlers = [
    rest.get('http://localhost:3030/scoops', (req, res, ctx) => {
      return res(
        ctx.json([
          { name: 'Chocolate', imagePath: '/images/chocolate.png' },
          { name: 'Vanilla', imagePath: '/images/vanilla.png' },
        ])
      )
    }),
  ]
  ```

  - 핸들러 타입, HTTP Method, 응답 리졸버 함수를 정의
  - `return res`로 응답을 생성하는 응답 함수를 반환
  - `ctx.json`
    - `JSON` 배열 형태의 실제 서버로 부터 받는 응답의 객체 구조를 정의

### 테스트 서버 생성

- 요청을 처리할 Mock 서버 생성

  - `src/mocks/server.js`

    ```jsx
    import { setupServer } from 'msw/node'
    import { handlers } from './handlers'

    export const server = setupServer(...handlers)
    ```

    - `setupServer`와 설정한 `handler` 파일을 가져오고 `sertupServer`의 매개변수에 handlers `JSON` 배열을 `rest parameter` 형태로 전달하고 실행

- `Mock Service Worker`가 네트워크 요청을 가로채 핸들러에서 설정한 응답을 반환하도록 구성해야 함

  - `src/setupTest.js`

    ```jsx
    import '@testing-library/jest-dom'
    import { server } from './mocks/server.js'

    beforeAll(() => server.listen())

    afterEach(() => server.resetHandlers())

    afterAll(() => server.close())
    ```

    - CRA로 프로젝트 생성 시, 초기 설정 파일인 `setupTest.js` 파일에 추가 설정을 작성
    - `beforeAll(() => server.listen());`
      - 테스트를 하기 전에 항상 서버가 수신을 대기하도록 함
      - 들어오는 모든 네트워크 요청을 실제 네트워크가 아닌 MSW로 라우팅
    - `afterEach(() => server.resetHandlers());`
      - 각 테스트가 끝나면 서버를 정의했을 때의 핸들러로 재설정
      - 서버가 오류를 반환하면 앱에서 무슨 일이 생기는지 테스트하기 위함임
    - `afterAll(() => server.close());`
      - 테스트가 끝나면 서버를 닫음

### 참고링크

- [https://mswjs.io/docs/getting-started/install](https://mswjs.io/docs/getting-started/install)

## Mock Service Work로 테스트

### Dummy Component 파일 생성

- `pages/entry/Options.jsx`

  ```jsx
  export default function Options({ optionsType }) {
    return <div />
  }
  ```

  - 스쿱 옵션 컴포넌트나 토핑의 렌더링 여부를 알기 위해서 `optionType` 프로퍼티를 전달

- `pages/entry/ScoopOptions.jsx`

  ```jsx
  export default function ScoopOption() {
    return <div />
  }
  ```

### 테스트 코드 작성

- 테스트의 대상이되는 `Options` 컴포넌트의 테스트 파일을 생성
- `pages/entry/tests/Options.test.jsx`

  ```jsx
  import { render, screen } from '@testing-library/react'
  import Options from '../Options'

  test('각 스쿱 옵션의 이미지가 서버로부터 응답받는가', () => {
    render(<Options optionType="scoops" />)

    // 이미지 찾기
    const scoopImages = screen.getAllByRole('img', { name: /scoop$/i })
    expect(scoopImages).toHaveLength(2)

    // 이미지의 대체 텍스트 확인하기
    const altText = scoopImages.map((element) => element.alt)
    expect(altText).toEqual(['Chocolate scoop', 'Vanilla scoop'])
  })
  ```

  - 서버에서 반환할 각 옵션의 이미지를 띄우기 위한 테스트 코드 작성
    - `render` 함수에 테스트의 대상이되는 컴포넌트(Option Component)를 렌더링하기 위해서 전달
    - 컴포넌트에 `optionType`으로 `scoops` 문자열을 전달
    - 모든 이미지의 대체 텍스트가 `scoop`(`/scoop$/i`)으로 끝나는 정규 표현식을 작성
    - 핸들러가 스쿱 옵션을 두개 반환하는 지 확인하는 단언문 작성
      - `expect(scoopImages).toHaveLength(2);`
        - `toHaveLength`는 `Jest` 단언이 아님
        - `scoopImages`의 길이가 `2`가 되는 것을 찾음
    - 이미지의 대체 텍스트를 찾는 단언문 작성
      - `const altText = scoopImages.map((element) => element.alt);`
        - `map` 함수를 이용해 모든 이미지에 대한 `alt` 텍스트를 취득
      - `expect(altText).toEqual(["Chocolate scoop", "Vanilla scoop"]);`
        - 매핑된 alt 텍스트 배열과 같은지 `toEqual` 옵션 사용
        - 숫자, 문자열 → `ToBe` Matcher 사용
        - 배열, 객체 → `toEqual` Matcher 사용

### MSW 요청 흐름

<img alt="jest_msw_2" src="/static/images/jest_msw_2.png" width="700"/>

- 옵션 컴포넌트를 실행하고 서버에 `get` 요청을 보내는 일반적인 흐름과 달리, `MSW`가 요청을 가로채서 옵션 컴포넌트에 응답 핸들러를 반환함

## `Option` 컴포넌트와 `ScoopOption` 컴포넌트 작성

### Axios 설치

```jsx
npm install axios
```

### `Options.jsx`

```jsx
import axios from 'axios'
import { useEffect, useState } from 'react'
import Row from 'react-bootstrap/Row'
import ScoopOption from './ScoopOption'

export default function Options({ optionType }) {
  const [items, setItems] = useState([])

  useEffect(() => {
    axios
      .get(`http://localhost:3030/${optionType}`)
      .then((response) => setItems(response.data))
      .catch((error) => {
        console.log(error)
      })
  }, [optionType])

  const ItemComponent = optionType === 'scoops' ? ScoopOption : null

  const optionItems = items.map((item) => (
    <ItemComponent key={item.name} name={item.name} imagePath={item.imagePath} />
  ))

  return <Row>{optionItems}</Row>
}
```

- `axios.get()`
  - URL을 호출하고 템플릿 리터럴을 사용해 `optionType`에 전달된 매개변수의 값을 구조 분해 할당으로 가져와서 axios로 get 요청
  - `items state`에 결과 값을 저장
- `useEffect Hook`으로 `optionType`의 상태가 변경될때마다, `axios` 요청
- `optionType`이 `scoops` 문자열일 경우 `ScoopOption` 컴포넌트를 `ItemComponent`에 할당하고 그렇지 않으면 `null`은 반환하는 삼항 연산자를 정의

### `ScoopOptions.jsx`

```jsx
import Col from 'react-bootstrap/Col'

export default function ScoopOptions({ name, imagePath }) {
  return (
    <Col xs={12} sm={6} md={4} lg={3} style={{ textAlign: 'center' }}>
      <img
        style={{ width: '75%' }}
        src={`http://localhost:3030/${imagePath}`}
        alt={`${name} scoop`}
      />
    </Col>
  )
}
```

- 핸들러에서 설정한 `JSON` 배열의 객체 구조를 참고해서 `name`과 `imagesPath`를 구조분해 할당으로 전달받음

- `alt={${name} scoop}`
  - `scoop`으로 끝나는 대체 텍스트를 구성

### TroubleShooting

- 서버에서 데이터를 가져올 때 Axios를 사용해 비동기 방식으로 페이지를 응답 결과로 채우므로 `async/await` 방식으로 처리해야 함

## `await findBy` 비동기 방식으로 채워지는 요소 찾는 방법

### `Options.test.jsx`

```jsx
import { render, screen } from '@testing-library/react'
import Options from '../Options'

test('각 스쿱 옵션의 이미지가 서버로부터 응답받는가', async () => {
  render(<Options optionType="scoops" />)

  // 이미지 찾기
  const scoopImages = await screen.findAllByRole('img', { name: /scoop$/i })
  expect(scoopImages).toHaveLength(2)

  // 이미지의 대체 텍스트 확인하기
  const altText = scoopImages.map((element) => element.alt)
  expect(altText).toEqual(['Chocolate scoop', 'Vanilla scoop'])
})
```

- 비동기 작업을 할 때마다 `async/await`와 `findBy` matcher를 사용해야 함

## 테스트에서 서버 에러 응답 시뮬레이션

### 테스트 개요

- 서버에서 에러 응답을 받았을 때 경고창이 나타나는지 확인
- `OrderEntry` 컴포넌트를 생성하여 `Scoops`, `Toppings`에 대해 총 2번 렌더링 진행
  - 에러 발생 시 경고창 배너 출력

### Dummy Component 생성

### `OrderEntry.jsx`

```jsx
export default function OrderEntry() {
  return <div />
}
```

### 테스트 파일 생성

### `OrderEntry.test.jsx`

```jsx
import { render, screen, waitFor } from '@testing-library/react'
import { rest } from 'msw'
import { server } from '../../../mocks/server'
import OrderEntry from '../OrderEntry'
const host = 'http://localhost:3030'
test('스쿱 및 토핑 라우트 핸들링', async () => {
  server.resetHandlers(
    rest.get(`${host}/scoops`, (req, res, ctx) => res(ctx.status(500))),
    rest.get(`${host}/toppings`, (req, res, ctx) => res(ctx.status(500)))
  )
  render(<OrderEntry />)

  const alerts = await screen.findAllByRole('alert', {
    name: '에러가 발생했습니다. 다시 시도해주세요.',
  })
})
```

- 렌더링할 `OrderEntry` 컴포넌트를 가져오고 `MSW`를 설정
- `handler`를 오버라이드하기 위해 새 핸들러를 생성하고 `rest`를 가져옴
- `resetHandler` 메서드를 사용해 핸들러를 인수로 취하고 서버에 관한 엔드포인트가 있는 모든 핸들러를 재설정
  - URL과 엔드포인트를 설정하고 리졸버로 에러 상태를 반환해 `500`인 상태 코드를 응답
- 경고창이 비동기식일 것을 예상해 `findBy` Matcher를 사용

## 옵션 서버 에러에 대한 경고 배너 코드 작성

### `pages/common/AlertBanner.jsx`

```jsx
import { Alert } from 'react-bootstrap'

export default function AlertBanner({ message, variant }) {
  const alertMessage = message || '에러가 발생했습니다. 다시 시도해주세요.'
  const alertVariant = variant || 'danger'

  return (
    <Alert variant={alertVariant} style={{ backgroundColor: 'red' }}>
      {alertMessage}
    </Alert>
  )
}
```

- `AlertBanner` 컴포넌트를 생성해서 `Error Message`를 정의하고 react-bootstrap 컴포넌트를 가져옴
- OR Operator(`||`)를 사용해 falsy 값을 처리

### `Options.jsx`

```jsx
export default function Options({ optionType }) {
  const [items, setItems] = useState([])
  const [error, setError] = useState(false)

  // optionType is 'scoops' or 'toppings'
  useEffect(() => {
    axios
      .get(`http://localhost:3030/${optionType}`)
      .then((response) => setItems(response.data))
      .catch((error) => setError(true))
  }, [optionType])

  if (error) {
    return <AlertBanner />
  }

  const ItemComponent = optionType === 'scoops' ? ScoopOption : ToppingOption

  const optionItems = items.map((item) => (
    <ItemComponent key={item.name} name={item.name} imagePath={item.imagePath} />
  ))

  return <Row>{optionItems}</Row>
}
```

- `Error State`를 선언하고 `Error`가 발생하여 `catch`문이 실행되는 경우 `Error State`를 `true`값으로 설정하고 `AlertBanner` 컴포넌트를 반환하도록 처리

### `OrderEntry.jsx`

```jsx
import Options from './Options'

export default function OrderEntry() {
  return (
    <div>
      <Options optionType="scoops" />
      <Options optionType="toppings" />
    </div>
  )
}
```

- 스쿱과 토핑에 관한 `Options` 컴포넌트로 반환하는 컴포넌트 코드를 작성

### Trouble Shooting

- `alert`의 역할을 찾을 수 없다는 `Test Faild` 에러가 발생

## 선택된 테스트만 실행하는 방법과 `waitFor`

- alert의 역할은 명확한데 모호한 에러로 실패한 경우, 즉 테스트 실패 원인이 불명확할 때 다수의 파일이 아닌 특정 파일만 테스트할 수 있음

1. `npm test` → `p` 옵션을 통해 테스트하고자 하는 파일 이름을 필터링
2. `test.only` 혹은 `test.skip`을 통해 테스트를 건너뛰거나, 특정 테스트만 실행할 수 있음

```jsx
import { render, screen, waitFor } from '@testing-library/react'
import { rest } from 'msw'
import { server } from '../../../mocks/server'
import OrderEntry from '../OrderEntry'
const host = 'http://localhost:3030'
test('스쿱 및 토핑 라우트 핸들링', async () => {
  server.resetHandlers(
    rest.get(`${host}/scoops`, (req, res, ctx) => res(ctx.status(500))),
    rest.get(`${host}/toppings`, (req, res, ctx) => res(ctx.status(500)))
  )
  render(<OrderEntry />)

  await waitFor(async () => {
    const alerts = await screen.findAllByRole('alert')
    expect(alerts).toHaveLength(2)
  })
})
```

- 두 개의 컴포넌트에서 `alert`을 출력하기 위해서 결과값은 2로 예상했지만 1개만 얻는 결과를 얻음
  - `toppings`와 `screen`이라는 2개의 서버호출을 다 처리하지 못한채 둘 중 하나가 먼저 돌아온 후 테스트가 실행되므로 원하는 결과를 얻지 못했음
  - 페이지에 경고창이 나타날 때까지 기다리기 위해서 `waitFor` 메서드와 `await/async` 키워드를 사용해 코드를 구현해야 함
- 앞서 살펴본 `waitForElementToBeRemoved` 함수와 유사하게 동작함

> **Referenced**

- [Testing React with Jest and React Testing Library (RTL)](https://www.udemy.com/course/understanding-typescript/)
