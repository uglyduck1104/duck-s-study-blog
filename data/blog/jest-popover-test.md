---
title: Form 복습과 팝오버 테스트
date: '2022-08-21'
tags: ['Jest']
draft: false
summary: Popover 테스트 실습 예제 및 쿼리 메서드 설명
layout: PostSimple
authors: ['default']
---

# 온디맨드 선데이 아이스크림: Form복습과 팝오버

## 프로젝트 초기화

```bash
npx create-react-app [project-name]
```

## ESLint와 Prettier 설정

1. `eslint plugin` package 설치

   ```bash
   npm install eslint-plugin-testing-library eslint-plugin-jest-dom
   ```

- testing-library
- jest-dom

2. `package.json` 내 `eslintConfig` 제거
3. `.eslintrc.json` 생성 및 규칙을 포함하는 구성 추가

   ```json
   {
     "plugins": ["jest-dom", "testing-library"],
     "extends": [
       "react-app",
       "react-app/jest",
       "plugin:testing-library/recommended",
       "plugin:testing-library/react",
       "plugin:jest-dom/recommened"
     ]
   }
   ```

4. `.gitignore` 파일에 `.vscode`, `.eslintcache` 추가

   ```
   .vscode
   .eslintcache
   ```

5. `VSCode` 프로젝트를 위한 설정 및 구성 추가

- `settings.json`

  ```json
  {
    "eslint.options": {
      "configFile": ".eslintrc.json"
    },
    "eslint.validate": ["javascript", "javascriptreact"],
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    },
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  }
  ```

  - ESLint 설정 파일 지정 및 저장 시 ESLint 교정 행위, 포맷팅 설정

## 코드 구성과 SummaryForm 소개

### 요구사항

- 체크박스가 버튼을 활성화하는지 테스트
- 이용 약관 `팝오버` 테스트
  - 페이지에서 사라진 요소를 위한 테스트

### 체계화

- 각 페이지 디렉토리의 테스트 서브디렉토리 구성
- Jest는 전체 디렉토리 트리 내의 `.test.js` 로 끝나는 모든 파일을 찾아 실행하므로 폴더 구성은 따로 필요없지만, 따로 테스트 디렉토리를 구성
- 특정 컴포넌트에 대해서는 `pages` 디렉토리를 만들고 하위 폴더 및 파일 구성
  - `src/pages/summary`
    - `OrderSummary.jsx`
    - `SummaryForm.jsx`
  - `src/pages/summary/test`
    - `SummaryForm.test.jsx`
    - `OrderSummary.test.jsx`

## 체크박스 활성화 버튼 테스트

### 요구사항

- 기본값으로 체크박스에 체크가 되어 있지 않아야 함
  - 체크박스에 체크를 하면 버튼이 활성화
  - 체크하지 않으면 비활성화
- 앱을 렌더링하지 않고 특정 컴포넌트(`<SummaryForm /`>만 렌더링
- 목업 모형에 있는 `{name}` 옵션을 통해 체크박스와 버튼 요소를 찾고, 레드-그린 테스트 진행

### `SummaryForm.jsx`

```jsx
export default function SummaryForm() {
  return <div />
}
```

- 레드-그린 테스트의 레드 부분을 진행할 때 테스트가 중단되는 것을 방지하기 위한 기본 구성을 정의

### `SummaryForm.test.jsx`

```jsx
import { render, screen, fireEvent } from '@testing-library/react'
import SummaryForm from '../SummaryForm'

test('초기화 조건', () => {
  render(<SummaryForm />)
  const checkbox = screen.getByRole('checkbox', {
    name: /terms and conditions/i,
  })
  expect(checkbox).not.toBeChecked()

  const confirmButton = screen.getByRole('button', { name: /confirm order/i })
  expect(confirmButton).toBeDisabled()
})

test('체크박스는 첫 번째 클릭 시 활성화, 두 번째 클릭 시 비활성화되어야 함', () => {
  render(<SummaryForm />)
  const checkbox = screen.getByRole('checkbox', {
    name: /terms and conditions/i,
  })
  expect(checkbox).not.toBeChecked()

  const confirmButton = screen.getByRole('button', { name: /confirm order/i })

  fireEvent.click(checkbox)
  expect(confirmButton).toBeEnabled()

  fireEvent.click(checkbox)
  expect(confirmButton).toBeDisabled()
})
```

- `render`
  - 컴포넌트 렌더링을 위한 컴포넌트를 설정
- `screen`
  - 컴포넌트의 요소를 찾기 위한 선언
  - 문자열 혹은 정규표현식을 통해 요소를 찾아야 함
    - `/terms and conditions/i`, `/confirm order/i`
      - 대문자를 무시하고 `name` option이 `terms and conditions`을 가지는 체크박스 요소, `confirm order`를 포함하는 정규 표현식
- `fireEvent`
  - 찾은 요소와의 DOM 상호작용을 위해 선언

## SummaryForm 체크박스와 버튼 요소 구성

### `SummaryForm.jsx`

```jsx
import React, { useState } from 'react'
import Form from 'react-bootstrap/Form'
import Button from 'react-bootstrap/Button'
export default function SummaryForm() {
  const [tcChecked, setTcChecked] = useState(false)

  const checkboxLabel = (
    <span>
      I agree to <span style={{ color: 'blue' }}> Terms and Conditions</span>
    </span>
  )

  return (
    <Form>
      <Form.Group controlId="terms-and-conditions">
        <Form.Check
          type="checkbox"
          checked={tcChecked}
          onChange={(e) => setTcChecked(e.target.checked)}
          label={checkboxLabel}
        />
      </Form.Group>
      <Button variant="primary" type="submit" disabled={!tcChecked}>
        Confirm order
      </Button>
    </Form>
  )
}
```

- 체크박스의 변경 유무를 form에서 감지하여 `useState` 훅으로 체크박스의 상태를 관리
  - 더불어, 체크박스의 상태를 추적하여 버튼의 활성화를 제어할 수 있음
  - 체크박스에 체크가 있으면 활성화, 체크가 해제되면 비활성화

## React BootStrap - Popover & Testing Library - userEvent

### Popover

- 이용 약관을 부트스트랩 템플릿의 팝오버 컴포넌트를 이용하여, 마우스를 특정 텍스트 위로 올려두었을 때 팝오버가 노출되게 구현

### userEvent

- 이전 예제에서 쓰였던 `fireEvent`와 같은 흐름의 `userEvent`가 있는데 fireEvent에 비해 사용자 이벤트를 더욱 완전하고 현실적인 방식으로 시뮬레이션하는 장점이 있음

  - [참고 링크](https://github.com/testing-library/user-event)

- `testing-library/user-event`와 `testing-library/dom` 설치

  ```bash
  npm install @testing-library/user-event @testing-library/dom
  ```

- `SummaryForm.test.js`

  ```jsx
  import userEvent from '@testing-library/user-event'

  //...
  test('체크박스는 첫 번째 클릭 시 활성화, 두 번째 클릭 시 비활성화되어야 함', () => {
    render(<SummaryForm />)
    const checkbox = screen.getByRole('checkbox', {
      name: /terms and conditions/i,
    })
    expect(checkbox).not.toBeChecked()

    const confirmButton = screen.getByRole('button', { name: /confirm order/i })

    userEvent.click(checkbox)
    expect(confirmButton).toBeEnabled()

    userEvent.click(checkbox)
    expect(confirmButton).toBeDisabled()
  })
  ```

  - 기존의 `fireEvent`를 `userEvent`로 변경

## Screen 쿼리 메서드

- 아래의 `Matcher`들을 조합해서 `DOM`에서 찾고자 하는 내용을 가장 적절한 방식으로 사용
- `command[All]ByQueryType`
  - command
    - `get`: 요소가 DOM 내에 존재하는지 기대함
    - `query`: 요소가 DOM 내에 존재하지 않는지 기대함
    - `find`: 요소가 비동기적으로 나타날 경우를 기대함
      - DOM에 비동기적인 업데이트가 있고, 단언문 실행 전 기다리고자 할 때 사용
  - `[All]`
    - 포함을 시키거나 포함을 시키지 않는 부분
    - 하나 이상의 매칭 포인트를 [All]을 포함시켜 처리할 수 있음
  - `QueryType`
    - 무엇으로 검색하는지를 의미함
    - `Role`: 코드의 접근성을 보장하기 위해 가장 많이 사용함
    - `AltText`: 이미지를 찾기위해서 사용
    - `Text`: 요소를 화면에 출력하기 위해서 사용
    - `Form Elements`: 폼 양식의 요소를 찾을 때 사용
      - PlaceholderText
      - LabelText
      - DisplayValue
- 참고 링크
  - [About Queries | Testing Library](https://testing-library.com/docs/queries/about/)
  - [Cheatsheet | Testing Library](https://testing-library.com/docs/react-testing-library/cheatsheet/)
  - [About Queries | Testing Library](https://testing-library.com/docs/queries/about/#priority)

## Popover 테스트

### `SummaryForm.text.jsx`

```jsx
test('팝오버가 hover 이벤트에 반응해야 함', () => {
  render(<SummaryForm />)

  // 페이지가 로딩되면 팝오버는 표시되지 않음
  const nullPopover = screen.queryByText(/no ice cream will actually be delivered/i)
  expect(nullPopover).not.toBeInTheDocument()

  // 체크박스 라벨로 마우스를 올리면 팝오버는 표시됨
  const termsAndConditions = screen.getByText(/terms and condition/i)
  userEvent.hover(termsAndConditions)

  const popover = screen.getByText(/no ice cream will actually be delivered/i)
  expect(popover).toBeInTheDocument()

  // 체크박스 라벨에서 마우스가 벗어나면 팝오버는 사라짐
  userEvent.unhover(termsAndConditions)
  const nullPopoverAgain = screen.queryByText(/no ice cream will actually be delivered/i)
  expect(nullPopoverAgain).not.toBeInTheDocument()
})
```

1. 페이지가 로딩되면 팝오버는 표시되지 않음

- `screen.queryByText`
  - 페이지가 로딩된 후 팝오버는 표시되지 않아야하므로, `queryByText`로 작성
    - `queryBy`를 사용할 시, 매칭되는 게 없으면 `null`을 반환
- `expect(nullPopover).not.toBeInTheDocument()`
  - 팝오버 요소가 `DOM` 존재하지 않는지를 테스트

2. 체크박스 라벨로 마우스를 올리면 팝오버는 표시됨

- `userEvent.hover(termsAndConditions)`
  - 마우스오버를 시뮬레이션하기 위해 `termsAndConditions` 요소를 `userEvent`를 사용해 hover 상호 작용
- `expect(popover).toBeInTheDocument()`
  - `hover` 했을 때, 해당 요소가 `DOM`에 존재하는지 테스트

3. 체크박스 라벨에서 마우스가 벗어나면 팝오버는 사라짐

- `userEvent.unhover(termsAndConditions)`
  - 마우스오버가 비활성화 상태 즉, 마우스가 해당 요소르 벗어났을 때의 상호 작용을 정의
- `expect(nullPopoverAgain).not.toBeInTheDocument()`
  - 팝오버 요소가 `DOM` 존재하지 않는지를 테스트

## 팝 오버 요소 구성

### `SummaryForm.jsx`

```jsx
import React, { useState } from 'react'
import Form from 'react-bootstrap/Form'
import Button from 'react-bootstrap/Button'
import Popover from 'react-bootstrap/Popover'
import OverlayTrigger from 'react-bootstrap/OverlayTrigger'

export default function SummaryForm() {
  const [tcChecked, setTcChecked] = useState(false)

  const popover = (
    <Popover id="termsandconditions-popover">No ice cream will actually be delivered</Popover>
  )

  const checkboxLabel = (
    <span>
      I agree to
      <OverlayTrigger placement="right" overlay={popover}>
        <span style={{ color: 'blue' }}> Terms and Conditions</span>
      </OverlayTrigger>
    </span>
  )

  return (
    <Form>
      <Form.Group controlId="terms-and-conditions">
        <Form.Check
          type="checkbox"
          checked={tcChecked}
          onChange={(e) => setTcChecked(e.target.checked)}
          label={checkboxLabel}
        />
      </Form.Group>
      <Button variant="primary" type="submit" disabled={!tcChecked}>
        Confirm order
      </Button>
    </Form>
  )
}
```

## Trouble Shooting

- 위의 리액트 코드를 작성한 후, 아래와 같은 오류가 발생함

  - `Warning: An update to Overlay inside a test was not wrapped in act(...).`
    `When testing, code that causes React state updates should be wrapped into act(...):`

    - 비동기 업데이트가 일어날 때 흔히 발생하는 문제점으로, 테스트 내 `Overlay`로의 업데이트를 act(…)로 감싸지 않았다는 내용
    - 또, 리액트 상태 업데이트 시키는 코드를 `act(…)`로 래핑할 것을 제안(하지 않는 것을 권장함)
    - [참고 링크](https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning)

### 해결점

- 테스트가 끝난 후 무엇이 변경되었는지 파악해야 함
  - 위의 에러는 테스트가 끝난 후 DOM이 업데이트 되었으므로, 어떤 것이 변경되었는지 확인

```jsx
test('팝오버가 hover 이벤트에 반응해야 함', () => {
  // ...

  // 체크박스 라벨에서 마우스가 벗어나면 팝오버는 사라짐
  userEvent.unhover(termsAndConditions)
  const nullPopoverAgain = screen.queryByText(/no ice cream will actually be delivered/i)
  expect(nullPopoverAgain).not.toBeInTheDocument()
})
```

- `expect(nullPopoverAgain).not.toBeInTheDocument();`

  - 테스트가 완료된 후 팝오버 요소가 비동기적으로 사라지는 문제를 야기
  - 팝업이 사라진 후까지 기다렸다가 테스트를 마저 진행하게 수정

    ```jsx
    test('팝오버가 hover 이벤트에 반응해야 함', async () => {
      // ...

      // 체크박스 라벨에서 마우스가 벗어나면 팝오버는 사라짐
      userEvent.unhover(termsAndConditions)
      await waitForElementToBeRemoved(() =>
        screen.queryByText(/no ice cream will actually be delivered/i)
      )
    })
    ```

  - `waitForElementToBeRemoved`를 사용하는 함수 전체를 `async` 키워드로 감싸고, `await` 키워드로 요소를 찾는 쿼리앞에 설정
  - [참고 링크](https://testing-library.com/docs/guide-disappearance/)

> **Referenced**

- [Testing React with Jest and React Testing Library (RTL)](https://www.udemy.com/course/understanding-typescript/)
