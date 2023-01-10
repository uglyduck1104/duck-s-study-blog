---
title: 최종 실습 테스트 코드
date: '2023-01-10'
tags: ['Jest']
draft: false
summary: 마지막 테스트 - 주문 단계
layout: PostSimple
authors: ['default']
---

# 최종 실습 테스트 코드

## 마지막 테스트: 주문 단계

```jsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

import App from '../App'

test('happy path를 위한 주문 단계', async () => {
  const user = userEvent.setup()
  render(<App />)

  const vanillaInput = await screen.findByRole('spinbutton', {
    name: 'Vanilla',
  })
  await user.clear(vanillaInput)
  await user.type(vanillaInput, '1')

  const chocolateInput = screen.getByRole('spinbutton', { name: 'Chocolate' })
  await user.clear(chocolateInput)
  await user.type(chocolateInput, '2')

  const cherriesCheckbox = await screen.findByRole('checkbox', {
    name: 'Chrries',
  })
  await user.click(cherriesCheckbox)

  const orderSummaryButton = screen.getByRole('button', {
    name: /order sundae/i,
  })
  await user.click(orderSummaryButton)

  const summaryHeading = screen.getByRole('heading', { name: 'Order Summary' })
  expect(summaryHeading).toBeInTheDocument()

  const scoopsHeading = screen.getByRole('heading', { name: 'Scoops: $6.00' })
  expect(scoopsHeading).toBeInTheDocument()

  const toppingsHeading = screen.getByRole('heading', {
    name: 'Topping: $1.50',
  })
  expect(toppingsHeading).toBeInTheDocument()

  expect(screen.getByText('1 Vanilla')).toBeInTheDocument()
  expect(screen.getByText('2 Chocolate')).toBeInTheDocument()
  expect(screen.getByText('Cherries')).toBeInTheDocument()

  const tcCheckBox = screen.getByRole('checkbox', {
    name: /terms and conditions/i,
  })
  await user.click(tcCheckBox)

  const confirmOrderButton = screen.getByRole('button', {
    name: /confirm order/i,
  })
  await user.click(confirmOrderButton)

  const thankYouHeader = await screen.findByRole('heading', {
    name: /thank you/i,
  })
  expect(thankYouHeader).toBeInTheDocument()

  const orderNumber = await screen.findByText(/order number/i)
  expect(orderNumber).toBeInTheDocument()

  const scoopTotal = await screen.findByText('Scoops total: $0.00')
  expect(scoopTotal).toBeInTheDocument()
  const toppingsTotal = await screen.findByText('Toppings total: $0.00')
  expect(toppingsTotal).toBeInTheDocument()

  await screen.findByRole('spinbutton', { name: 'Vanilla' })
  await screen.findByRole('checkbox', { name: 'Cherries' })
})
```

- 앱 렌더링 하기

  ```jsx
  const user = userEvent.setup()
  render(<App />)
  ```

  - 사용자 인스턴스를 가져온 후 앱을 렌더링

- 아이스크림 스쿱과 토핑 추가하기

  ```jsx
  const vanillaInput = await screen.findByRole('spinbutton', {
    name: 'Vanilla',
  })
  await user.clear(vanillaInput)
  await user.type(vanillaInput, '1')

  const chocolateInput = screen.getByRole('spinbutton', { name: 'Chocolate' })
  await user.clear(chocolateInput)
  await user.type(chocolateInput, '2')

  const cherriesCheckbox = await screen.findByRole('checkbox', {
    name: 'Chrries',
  })
  await user.click(cherriesCheckbox)
  ```

  - 아이스크림 스쿱과 토핑을 추가하기 위해 `await`를 사용해 테스트 함수를 비동기 테스트로 만들어야 함
  - 옵션을 가져오기 위해 서버로 보낸 `Axios` 호출이 `resolve` 되기 전까지 `Vanilla` 옵션은 보이지 않기 때문에 `user.clear`를 사용해 입력란에 있는 텍스트를 제거하고 숫자 1을 입력
    - `userEvent` 메서드를 `await` 하는 건 매우 중요한게, 프로미스 방식이므로 `await`로 대기하지 않으면 예상치 못한 동작이 발생할 수 있음
    - `Vanilla`가 표시되면 `Chocolate`도 표시되므로 `await`를 사용하지 않아도 됨
  - `cherriesCheckbox` 체크 박스를 클릭해 선데이 아이스크림을 주문

- 주문 입력 페이지에서 주문 버튼을 찾아 클릭하기

  ```jsx
  const orderSummaryButton = screen.getByRole('button', {
    name: /order sundae/i,
  })
  await user.click(orderSummaryButton)
  ```

  - `order sundae` 이름을 가지 버튼 요소를 찾아 클릭

- 주문 내용을 기반으로 요약 정보가 올바른지 확인하기

  ```jsx
  const summaryHeading = screen.getByRole('heading', { name: 'Order Summary' })
  expect(summaryHeading).toBeInTheDocument()

  const scoopsHeading = screen.getByRole('heading', { name: 'Scoops: $6.00' })
  expect(scoopsHeading).toBeInTheDocument()

  const toppingsHeading = screen.getByRole('heading', {
    name: 'Topping: $1.50',
  })
  expect(toppingsHeading).toBeInTheDocument()
  ```

  - 요약 페이지로 이동해 요약 정보를 확인하고 `heading` 요소를 찾고 스쿱 소계가 표시되는지 확인

- 요약 항목에서 옵션 아이템 정보 확인하기

  ```jsx
  expect(screen.getByText('1 Vanilla')).toBeInTheDocument()
  expect(screen.getByText('2 Chocolate')).toBeInTheDocument()
  expect(screen.getByText('Cherries')).toBeInTheDocument()
  // 대체 가능
  // const optionItems = screen.getAllByRole('listItem');
  // const optionItemsText = optionItems.map((item) => item.textContent);
  // expect(optionItemsText).toEqual(['1 Vanilla', '2 Chocolate', 'Cherries']);
  ```

  - 요약 항목에서 목업을 기반한 옵션 아이템 정보 확인
  - `map`을 사용해 목록 항목의 텍스트를 순회하여 확인할 수 있음

- 이용 약관 체크박스를 찾아 클릭하고 활성화된 주문 버튼 클릭하기

  ```jsx
  const tcCheckBox = screen.getByRole('checkbox', {
    name: /terms and conditions/i,
  })
  await user.click(tcCheckBox)

  const confirmOrderButton = screen.getByRole('button', {
    name: /confirm order/i,
  })
  await user.click(confirmOrderButton)
  ```

  - 이용 약관 `checkbox` 요소를 찾아 클릭하고, 활성화된 주문 버튼을 클릭

- 확인 페이지에서 새 주문 버튼 클릭하기

  ```jsx
  const thankYouHeader = await screen.findByRole('heading', {
    name: /thank you/i,
  })
  expect(thankYouHeader).toBeInTheDocument()

  const orderNumber = await screen.findByText(/order number/i)
  expect(orderNumber).toBeInTheDocument()
  ```

  - 서버에 `POST` 요청을 보내므로 요청이 완료될 때까지 `await`해야 함
    - 요청이 완료되면 감사 인사와 주문 번호를 표시

- 아이스크립 스쿱과 토핑 소계가 재설정됐는지 확인하기

  ```jsx
  const scoopTotal = await screen.findByText('Scoops total: $0.00')
  expect(scoopTotal).toBeInTheDocument()
  const toppingsTotal = await screen.findByText('Toppings total: $0.00')
  expect(toppingsTotal).toBeInTheDocument()
  ```

  - 주문 입력 페이지에서 토핑과 스쿱이 제거되는지 확인하기 위해 총계가 0.00으로 리셋되는지 확인

- 테스트 완료된 후

  ```jsx
  await screen.findByRole('spinbutton', { name: 'Vanilla' })
  await screen.findByRole('checkbox', { name: 'Cherries' })
  ```

  - 바닐라 스핀 버튼과 체리 체크박스를 통해 스쿱에 대한 `Axios` 호출과 토핑에 대한 `Axios` 호출이 테스트 종료전에 반환되게 함

> **Referenced**

- [Testing React with Jest and React Testing Library (RTL)](https://www.udemy.com/course/understanding-typescript/)
