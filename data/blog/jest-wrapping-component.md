---
title: 제공자에 래핑된 컴포넌트 테스트하기
date: '2022-09-04'
tags: ['Jest']
draft: false
summary: Context API Wrapping Component Test Example
layout: PostSimple
authors: ['default']
---

# 제공자에 래핑된 컴포넌트 테스트하기

## 텍스트 입력란 채우기: 소계 테스트

### `entry/tests/totalUpdates.test.jsx`

```jsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Options from '../Options'

test('스쿱의 수가 변경되면 스쿱 소계가 업데이트 되는가?', async () => {
  render(<Options optionType="scoops" />)

  // 처음에는 0달러로 시작
  const scoopsSubtotal = screen.getByText('Scoops total: $', { exact: false })
  expect(scoopsSubtotal).toHaveTextContent('0.00')

  // 바닐라 스쿱을 1로 업데이트하고, 소계 확인
  const vanillaInput = await screen.findByRole('spinbutton', {
    name: 'Vanilla',
  })
  userEvent.clear(vanillaInput)
  userEvent.type(vanillaInput, '1')
  expect(scoopsSubtotal).toHaveTextContent('2.00')

  // 초콜릿 스쿱을 2로 업데이트하고, 소계 확인
  const chocolateInput = await screen.findByRole('spinbutton', {
    name: 'Chocolate',
  })
  userEvent.clear(chocolateInput)
  userEvent.type(chocolateInput, '2')
  expect(scoopsSubtotal).toHaveTextContent('6.00')
})
```

- 스쿱의 수를 변경하면 스쿱의 소계가 업데이트 되는지 테스트
  1. `render(<Options optionType="scoops" />);`
  - `Options type`이 `scoops`인 컴포넌트를 렌더링
  2. 0.00달러로 시작하는 scoops의 가격을 가져와 단언함
  - `Scoops total: $`: 텍스트를 가지는 데이터를 가져옴
    - 정규 표현식 대신 문자열을 사용해 옵션 자체(`exact`)를 이용해야만 한다는 의미로 사용
    - `exact`는 기본값이 `true`이므로, `false`로 설정해야 전체 문자열이 아니어도 해당 요소를 찾을 수 있음
  - `toHaveTextContent()`: 해당 부분을 포함하는 경우에는 단언이 성공하고 그렇지 않으면 실패
  3. 바닐라 스쿱을 1로 업데이트하고, 소계 확인
  - `async / await`를 사용하고 find 단언문을 통해 테스트 함수를 비동기화해야 함
    - 서버에서 옵션을 받기전에 값을 채우지 않기 때문임
    - 역할이 스핀 버튼이고 이름 옵션이 라벨을 인식함
  - `userEvent.clear();`
    - 커서가 어디에 놓여있는지 알 수 없기 때문에, 텍스트 이미지를 업데이트 할 때마다 `clear(element)`로 요소를 삭제
  - `userEvent.type();`
    - 요소를 가져와서 텍스트를 지정함
    - (`vanillaInput`, `'1'`)를 입력해 먼저 요소를 넣고, 텍스트를 넣음
      - 숫자가 아닌 문자열을 지정해야 함
  4. 초콜릿 스쿱을 2로 업데이트하고, 소계 확인
  - 3번과 동일

## `OrderDetails` 컨텍스트 - ReactJS

### `contexts/OrderDetails.jsx`

```jsx
import { createContext, useContext, useEffect, useMemo, useState } from 'react'
import { pricePerItem } from '../constants'

const OrderDetails = createContext()

// Provider 내부에 있는지의 여부를 확인해 주는 사용자 정의 훅
export function useOrderDetails() {
  const context = useContext(OrderDetails)

  if (!context) {
    throw new Error('useOrderDetails must be used within an OrderDetailsProvider')
  }

  return context
}

function calculateSubtotal(optionType, optionCounts) {
  let optionCount = 0
  for (const count of optionCounts[optionType].value()) {
    optionCount += count
  }

  return optionCount * pricePerItem[optionType]
}

export function OrderDetailsProvider(props) {
  const [optionCounts, setOptionsCounts] = useState({
    scoops: new Map(),
    toppings: new Map(),
  })
  const [totals, setTotals] = useState({
    scoops: 0,
    toppings: 0,
    grandTotal: 0,
  })

  useEffect(() => {
    const scoopsSubtotal = calculateSubtotal('scoops', optionCounts)
    const toppingsSubtotal = calculateSubtotal('toppings', optionCounts)
    const grandTotal = scoopsSubtotal + toppingsSubtotal
    setTotals({
      scoops: scoopsSubtotal,
      toppings: toppingsSubtotal,
      grandTotal,
    })
  }, [optionCounts])

  const value = useMemo(() => {
    function updateItemCount(itemName, newItemCount, optionsType) {
      const newOptionsCounts = { ...optionCounts }

      // update option count for this item with the new value
      const optionCountsMap = optionCounts[optionsType]
      optionCountsMap.set(itemName, parseInt(newItemCount))

      setOptionsCounts(newOptionsCounts)
    }
    // getter: 주문한 스쿱 및 토핑 모두의 옵션 세부사항을 포함하는 객체(소계와 총 합계 포함)
    // setter: 옵션 갯수 업데이트
    return [{ ...optionCounts }, updateItemCount]
  }, [optionCounts, totals])
  return <OrderDetails.Provider value={value} {...props} />
}
```

- Kent C. Dodds가 고안한 Context Pattern을 사용
  - 임베딩 상태를 가진 컨텍스트를 설정할 수 있는 패턴
  - `createContext` 훅을 사용해 컨텍스트를 생성
  - `useContext` 훅을 사용해 `Provider` 내에 있는 컨텍스트를 반환할 커스텀 훅을 생성
  - 상태의 `getter`와 `setter`를 가지는 컨텍스트를 실행할 때마다 컨텍스트 값을 받아 액세스 권한을 가진 다수의 컴포넌트 간에 값을 공유할 수 있음
- 주문 입력란과 주문 요약 컴포넌트를 컨텍스트로 래핑
- `useMemo` 훅을 사용해 컨텍스트가 불필요하게 자주 호출되는 것을 방지
- `useOrderDetails` 훅을 정의해 `context`가 `Provider`에 있는지 여부를 확인

### `updateItemCount` Func & `optionCounts` State

```jsx
const [optionCounts, setOptionsCounts] = useState({
  scoops: new Map(),
  toppings: new Map(),
})

function updateItemCount(itemName, newItemCount, optionsType) {
  const newOptionsCounts = { ...optionCounts }

  // 옵션 유형에 관련된 맵을 사용하고 set 메서드를 통해 항목 개수를 업데이트함
  const optionCountsMap = optionCounts[optionsType]
  optionCountsMap.set(itemName, parseInt(newItemCount))

  setOptionsCounts(newOptionsCounts)
}
```

- 옵션 개수에 대한 새로운 상태를 생성
  - `scoops`와 `toppings`의 키를 가지는 맵(`Map - [key, value] pair`) 형태의 객체
- `setter` 역할을 하는 `updateItemCount` 함수 정의

  ```jsx
  const newOptionsCounts = { …optionsCounts };
  ```

  - `optionCounts` 상태를 `spread operator`(`…`)를 사용해 복사본을 만들어 옵션 개수를 설정

    ```jsx
    const optionCountsMap = optionCounts[optionsType]
    optionCountsMap.set(itemName, parseInt(newItemCount))
    ```

  - 옵션 유형에 관련된 맵을 사용하고 set 메서드를 통해 항목 개수를 업데이트함
  - `newItemCount`가 문자열로 처리될 수 있으므로, `parseInt` 메소드를 통해 형변환 처리

### `totals` State & `useEffect` Hook

```jsx
const [totals, setTotals] = useState({
  scoops: 0,
  toppings: 0,
  grandTotal: 0,
})

useEffect(() => {
  const scoopsSubtotal = calculateSubtotal('scoops', optionCounts)
  const toppingsSubtotal = calculateSubtotal('toppings', optionCounts)
  const grandTotal = scoopsSubtotal + toppingsSubtotal
  setTotals({
    scoops: scoopsSubtotal,
    toppings: toppingsSubtotal,
    grandTotal,
  })
}, [optionCounts])
```

- 소계와 총합계를 가지는 상태를 정의하고, `optionCounts`가 총 합계를 업데이트할 때마다 `useEffect`를 실행하도록 함
- 총합계가 변동될때마다 `calculateSubtotal` 헬퍼 함수에 `optionType`과 `optionCounts`를 인수로 넘겨줌

### `calculateSubtotal` Helper Func

```jsx
function calculateSubtotal(optionType, optionCounts) {
  let optionCount = 0
  for (const count of optionCounts[optionType].values()) {
    optionCount += count
  }

  return optionCount * pricePerItem[optionType]
}
```

- 소계를 계산하는 헬퍼 함수를 작성
- `optionType`과 `optionCounts`를 매개변수로 받아 맵을 거쳐 사용자들이 선택한 모든 옵션들을 한데 누적해서 계산
- `constants/index.js`

  ```jsx
  export const pricePerItem = {
    scoops: 2,
    toppings: 1.5,
  }
  ```

  - 일반적으로 서버로부터 받는 정보를 `dummy data`로 만들어 특정 아이스크림 가게의 옵션 당 가격을 보관하고 있는 상수 파일을 만들어서 관리

```jsx
return <OrderDetails.Provider value={value} {...props} />
```

- 받게 될 프로퍼티를 외부의 Provider에 모두 반환

## 컨텍스트로 스쿱 소계 표시하기 - ReactJS

### `App.js`

```jsx
import Container from 'react-bootstrap/Container'
import { OrderDetailsProvider } from './contexts/OrderDetails'
import OrderEntry from './pages/entry/OrderEntry'

function App() {
  return (
    <Container>
      <OrderDetailsProvider>
        <OrderEntry />
      </OrderDetailsProvider>
    </Container>
  )
}

export default App
```

- OrderDeatilsProvider 내에 주문 입력란(OrderEntry)을 래핑

### `Options.jsx`

```jsx
import axios from 'axios'
import { useEffect, useState } from 'react'
import Row from 'react-bootstrap/Row'
import { pricePerItem } from '../../constants'
import { useOrderDetails } from '../../contexts/OrderDetails'
import AlertBanner from '../common/AlertBanner'
import ScoopOption from './ScoopOption'
import ToppingOption from './ToppingOption'

export default function Options({ optionType }) {
  const [items, setItems] = useState([])
  const [error, setError] = useState(false)
  const [orderDetails, updateItemCount] = useOrderDetails()

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
  const title = optionType[0].toUpperCase() + optionType.slice(1).toLowerCase()

  const optionItems = items.map((item) => (
    <ItemComponent
      key={item.name}
      name={item.name}
      imagePath={item.imagePath}
      updateItemCount={(itemName, newItemCount) =>
        updateItemCount(itemName, newItemCount, optionType)
      }
    />
  ))

  return (
    <>
      <h2>{title}</h2>
      <p>{pricePerItem[optionType]} each</p>
      <p>
        {title} total: {orderDetails.totals[optionType]}
      </p>
      <Row>{optionItems}</Row>
    </>
  )
}
```

- 자식 컴포넌트로 전달할 수 있는 업데이트를 생성하기 위해서 컨텍스트를 적용

```jsx
const title = optionType[0].toUpperCase() + optionType.slice(1).toLowerCase()
```

- `optionType`을 가져와서 첫 글자를 대문자 처리하고 나머지 글자는 소문자로 처리
- `slice method`를 사용해 인덱스를 1부터 시작

```jsx
return (
  <>
    <h2>{title}</h2>
    <p>{pricePerItem[optionType]} each</p>
    <p>
      {title} total: {orderDetails.totals[optionType]}
    </p>
    <Row>{optionItems}</Row>
  </>
)
```

- 아이템 당 가격(`pricePerItem`)을 출력하는 constant 파일을 출력

```jsx
const [orderDetails, updateItemCount] = useOrderDetails()
```

- `useOrderDetails` 훅을 통해 소계의 업데이트를 위한 `getter`, 자식 컴포넌트에 전달하기 위한 `setter`를 모두 가져옴

```jsx
const optionItems = items.map((item) => (
  <ItemComponent
    key={item.name}
    name={item.name}
    imagePath={item.imagePath}
    updateItemCount={(itemName, newItemCount) =>
      updateItemCount(itemName, newItemCount, optionType)
    }
  />
))
```

- 자식 컴포넌트로 보낼 `updateItemCount` 함수를 전달
  - `itemName`, `newItemCount`를 받아 `useOrderDetails`에서 가져온 `updateItemCount`를 실행
  - `updateItemCount` 함수에 전달되어야 하는 옵션 컴포넌트의 유형(`optionType`) 전달

```jsx
<p>
  {title} total: {orderDetails.totals[optionType]}
</p>
```

- `orderDetails`의 `totals` 키를 사용해 컨텍스트에서 `optionType`에 대한 합계를 추출

### `ScoopOption.jsx`

```jsx
import { Form } from 'react-bootstrap'
import Col from 'react-bootstrap/Col'
import Row from 'react-bootstrap/Row'

export default function ScoopOption({ name, imagePath, updateItemCount }) {
  const handleChange = (event) => {
    updateItemCount(name, event.target.value)
  }
  return (
    <Col xs={12} sm={6} md={4} lg={3} style={{ textAlign: 'center' }}>
      <img
        style={{ width: '75%' }}
        src={`http://localhost:3030/${imagePath}`}
        alt={`${name} scoop`}
      />
      <Form.Group controlId={`${name}-count`} as={Row} style={{ marginTop: '10px' }}>
        <Form.Label column xs="6" style={{ textAlign: 'right' }}>
          {name}
        </Form.Label>
        <Col xs="5" style={{ textAlign: 'left' }}>
          <Form.Control type="number" defaultValue={0} onChange={handleChange} />
        </Col>
      </Form.Group>
    </Col>
  )
}
```

- 부모 컴포넌트로부터 전달받는 `updateItemCount` props를 추가하고 사람들이 옵션과 원하는 스쿱의 개수를 입력할 수 있는 폼을 추가

```jsx
const handleChange = (event) => {
  updateItemCount(name, event.target.value)
}
```

- 입력폼이 바뀔때마다 change event인 `handleChange`를 정의해서 `updateItemCount` 함수를 실행하고 새로운 값인 `event.target.value`을 전달

### TroubleShotting

- 테스트 실패 원인 찾기
  - `OrderDetails`는 `OrderDetailsProvider` 내에서만 사용되어야 함
  - `useOrderDetails`를 제공자의 외부에서 사용하려 할 때, 커스텀 훅이 발생시키는 오류 발생

## 테스트 설정에 컨텍스트 추가하기 - 코드 에러 catch 테스트

1. 컨텍스트를 위해 만든 커스텀 훅인 `useOrderDetails` 가 `OrderDetailsProvider` 내에서만 사용해야한다는 오류를 잡아내는 테스트 코드 작성
2. 포맷팅 테스트 코드 작성

### `totalUpdates.test.jsx`

```jsx
test('스쿱의 수가 변경되면 스쿱 소계가 업데이트 되는가?', async () => {
  render(<Options optionType="scoops" />, { wrapper: OrderDetailsProvider })
  // ...
})
```

### `Options.test.jsx`

```jsx
test('각 스쿱 옵션의 이미지가 서버로부터 응답받는가', async () => {
  render(<Options optionType="scoops" />, { wrapper: OrderDetailsProvider })
  // ...
})
```

- 옵션 컴포넌트를 개별적으로 렌더링하고 있으므로, `Options` 컴포넌트를 위한 컨텍스트를 제공해야함
  - 래퍼 옵션(`wrapper`)을 사용해 컴포넌트(`OrderDetailsProvider`)를 지정
    - `OrderDetailsProvider`로 래핑하면 `Provider`에 접근해 내부 상태를 업데이트하기 위해 커스텀 훅을 사용할 수 있게됨
  - 컨텍스트 뿐만아니라, `Redux Provider`, `Router`도 가능함

### `OrderDetails.jsx`

```jsx
import { createContext, useContext, useEffect, useMemo, useState } from 'react'
import { pricePerItem } from '../constants'

// 통화 포맷팅
function formatCurrency(amount) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format(amount)
}
// ...
export function OrderDetailsProvider(props) {
  const [optionCounts, setOptionsCounts] = useState({
    scoops: new Map(),
    toppings: new Map(),
  })
  const zeroCurrency = formatCurrency(0)
  const [totals, setTotals] = useState({
    scoops: zeroCurrency,
    toppings: zeroCurrency,
    grandTotal: zeroCurrency,
  })

  useEffect(() => {
    const scoopsSubtotal = calculateSubtotal('scoops', optionCounts)
    const toppingsSubtotal = calculateSubtotal('toppings', optionCounts)
    const grandTotal = scoopsSubtotal + toppingsSubtotal
    setTotals({
      scoops: formatCurrency(scoopsSubtotal),
      toppings: formatCurrency(toppingsSubtotal),
      grandTotal: formatCurrency(grandTotal),
    })
  }, [optionCounts])
  // ...
}
```

- JavaScript 객체인 `Intl.NumberFormat`을 사용해 예제상의 미국 통화 단위로 포맷팅
  - 통화 단위, 최소 두 자리의 센트 단위까지 지정(소수점 아래 두 자리까지 출력)
- 통화가 사용되는 부분을 찾아 적용

## 기본값으로 제공자에 래핑할 커스텀 렌더링 생성

- 파편화되어있는 래퍼를 렌더가 자동으로 래퍼를 포함할 수 있게 중앙 집중화하는 작업
- 기본 React 테스팅 라이브러리 렌더를 래퍼를 포함하고 있는 자체 커스텀 렌더로 교체

### `src/test-utils/testing-library-utils.jsx`

```jsx
import { render } from '@testing-library/react'
import { OrderDetailsProvider } from '../contexts/OrderDetails'

const renderWithContext = (ui, options) => render(ui, { wrapper: OrderDetailsProvider, ...options })

export * from '@testing-library/react'

// render 메소드 오버라이드
export { renderWithContext as render }
```

- 공식 문서에서는 `test-utils.js` 파일을 생성해 `testing-library/react`가 아니라 `test-utils`로부터 가져올 것을 권장하고 있음
  - [공식문서 참고](https://testing-library.com/docs/react-testing-library/setup/#custom-render)
- `Provider`를 가져와 렌더로 내보낼 커스텀 렌더를 만듦
- 렌더링하고자 하는 `.jsx` 파일인 `ui`와 `options` 객체를 받아 `wrapper` 옵션에 주입하고 `renderWithContext` 별칭을 가지는 `render`를 반환

## Toppings 소계 테스트

```jsx
test('토핑이 변경되면 토핑의 소계가 업데이트 되는가', async () => {
  // 부모 컴포넌트 render
  render(<Options optionType="toppings" />)

  // 처음에는 0달러로 시작
  const toppingsTotal = screen.getByText('Toppings total: $', { exact: false })
  expect(toppingsTotal).toHaveTextContent('0,00')

  // 체리 토핑 추가 및 소계 체크
  const cherriesCheckbox = await screen.findByRole('checkbox', {
    name: 'Cherries',
  })
  userEvent.click(cherriesCheckbox)
  expect(toppingsTotal).toHaveTextContent('1.50')

  // hot fudge 토핑 추가 및 소계 체크
  const hotFudgeCheckbox = screen.getByRole('checkbox', { name: 'Hot fudge' })
  userEvent.click(hotFudgeCheckbox)
  expect(toppingsTotal).toHaveTextContent('3,00')

  // hotd fudge 토핑 및 소계 삭제
  userEvent.click(hotFudgeCheckbox)
  expect(toppingsTotal).toHaveTextContent('1.50')
})
```

1. 부모 컴포넌트를 `render`
2. 상단의 스쿱 테스트와 동일하게 토핑 값이 0 달러로 시작하는 테스트 코드 작성
3. 옵션 중 `Cherries`라는 이름의 체크박스 Role을 찾아 클릭했을 때, `toppingsTotal`값이 적용되는지 테스트

- `Axios` 호출에 의해 채워지므로 `async/await`로 감싸줌
- 체크박스를 클릭하기 위해 userEvent.click 이벤트 사용

4. 옵션 중 Hot fudge라는 이름의 체크박스를 찾아 클릭했을 때, `toppingsTotal` 값이 적용되는지 테스트

- 기존의 `Cherries` 옵션에 Hot fudge 옵션마저 추가했으니 `toppingsTotal` 값은 3.00달러를 반환함

5. 옵션 중 Hot fudge를 한번 더 클릭했을 때, 체크박스는 해제되어 `toppingsTotal` 값이 갱신되는지 테스트

- 3.00 달러인 `toppingsTotal` 값은 1.50값이 감한 1.50달러를 반환해야 함

## Toppings 체크박스 - ReactJS

### `ToppingOptions.jsx`

```jsx
import React from 'react'
import { Form } from 'react-bootstrap'
import Col from 'react-bootstrap/Col'

export default function ToppingOption({ name, imagePath, updateItemCount }) {
  return (
    <Col xs={12} sm={6} md={4} lg={3} style={{ textAlign: 'center' }}>
      <img
        style={{ width: '75%' }}
        src={`http://localhost:3030/${imagePath}`}
        alt={`${name} topping`}
      />
      <Form.Group controlId={`${name}-topping-checkbox`}>
        <Form.Check
          type="checkbox"
          onChange={(e) => {
            updateItemCount(name, e.target.checked ? 1 : 0)
          }}
          label={name}
        />
      </Form.Group>
    </Col>
  )
}
```

- `ScoopOption` 컴포넌트와 동일하게 `updateItemCount` 함수를 부모 컴포넌트로 부터 전달받아 체크박스가 변경됨에 따라 값을 전달함
  - `onChange Event`를 통해 체크박스가 체크되면 옵션의 개수가 1이 되도록하고 체크 되어 있지 않으면 0으로 업데이트 함

## 총 합계를 반환하는 테스트

### `totalUpdates.test.jsx`

- 비동기 환경에서 테스트할 때는 무엇이 먼저 끝나게 될지 알 수 없으므로, 스쿱 요소를 대기하고 토핑 요소를 대기하거나 토핑 요소가 먼저 선택되는 테스트의 경우 반대 순서로 대기
- 총 합계 요소를 `heading` 요소로 페이지에서 찾아서 테스트

```jsx
describe('grand total', () => {
  test('총 합은 0달러로 시작해야 함', () => {
    render(<OrderEntry />)
    const grandTotal = screen.getByRole('heading', {
      name: /grand total: \$/i,
    })
    expect(grandTotal).toHaveTextContent('0.00')
  })

  test('스쿱을 먼저 추가할 때, 총합이 업데이트 되야 함', async () => {
    render(<OrderEntry />)

    // 바닐라 스쿱을 2로 업데이트하고, 총 합계 확인
    const grandTotal = screen.getByRole('heading', {
      name: /grand total: \$/i,
    })
    const vanillaInput = await screen.findByRole('spinbutton', {
      name: 'Vanilla',
    })
    userEvent.clear(vanillaInput)
    userEvent.type(vanillaInput, '2')
    expect(grandTotal).toHaveTextContent('4.00')

    // 체리 토핑이 추가될 때, 총 합계 확인
    const cherriesCheckbox = await screen.findByRole('checkbox', {
      name: 'Cherries',
    })
    userEvent.click(cherriesCheckbox)
    expect(grandTotal).toHaveTextContent('5.50')
  })

  test('토핑을 먼저 추가할 때, 총합이 업데이트 되야 함', async () => {
    render(<OrderEntry />)

    // 체리 토핑이 추가될 때, 총 합계 확인
    const cherriesCheckbox = await screen.findByRole('checkbox', {
      name: 'Cherries',
    })
    userEvent.click(cherriesCheckbox)
    const grandTotal = screen.getByRole('heading', {
      name: /grand total: \$/i,
    })
    expect(grandTotal).toHaveTextContent('1.50')

    // 바닐라 스쿱을 2로 업데이트하고, 총 합계 확인
    const vanillaInput = await screen.findByRole('spinbutton', {
      name: 'Vanilla',
    })
    userEvent.clear(vanillaInput)
    userEvent.type(vanillaInput, '2')
    expect(grandTotal).toHaveTextContent('5.50')
  })

  test('옵션 항목이 삭제될 때, 총합이 업데이트 되야 함', async () => {
    render(<OrderEntry />)

    // 체리 토핑 추가
    const cherriesCheckbox = await screen.findByRole('checkbox', {
      name: 'Cherries',
    })
    userEvent.click(cherriesCheckbox)
    // 총 합 $1.50

    // 바닐라 스쿱을 2로 업데이트하고 총합은 $5.50
    const vanillaInput = await screen.findByRole('spinbutton', {
      name: 'Vanilla',
    })
    userEvent.clear(vanillaInput)
    userEvent.type(vanillaInput, '2')

    // 바닐라 스쿱 옵션을 1개 삭제하고 총합 확인
    userEvent.clear(vanillaInput)
    userEvent.type(vanillaInput, '1')

    // 총합 확인
    const grandTotal = screen.getByRole('heading', {
      name: /grand total: \$/i,
    })
    expect(grandTotal).toHaveTextContent('3.50')

    // 체리 토핑이 삭제된 후 총합 확인
    userEvent.click(cherriesCheckbox)
    expect(grandTotal).toHaveTextContent('2.00')
  })
})
```

- `describe`에 몇 개의 테스트를 작성해서, 옵션을 추가하거나 삭제했을 때 총합이 올바르게 변하는지 테스트
- 총합계 업데이트 테스트의 경우 `async` 키워드를 사용해 실제로 옵션을 클릭하기 전에 옵션이 페이지 상으로 나타나기를 대기(`await`)함

### `OrderEntry.jsx`

```jsx
import { useOrderDetails } from '../../contexts/OrderDetails'
import Options from './Options'

export default function OrderEntry() {
  const [orderDetails] = useOrderDetails()
  return (
    <div>
      <h1>Design Your Sundae!</h1>
      <Options optionType="scoops" />
      <Options optionType="toppings" />
      <h2>Grand total: {orderDetails.totals.grandTotal}</h2>
    </div>
  )
}
```

- 그린 테스트를 통과하기 위해, 총합 출력을 하는 `heading` 요소를 추가
- `useOrderDetails`로부터 `orderDetails`를 받아 총합을 출력

## Unmounted Component 에러

### 원인

- 컴포넌트가 테스트 종류 후에 비동기 업데이트를 계속하고 있음
  - OrderEntry를 렌더링 중인데 OrderEntry는 Options 컴포넌트 두개를 렌더링하고 Options 컴포넌트별로 Axios 호출을 실행
  - Axios 호출을 반환하면 컴포넌트를 업데이트하며 테스트로 확인 과정을 거친 후 컴포넌트에서 설명할 수 없는 발생하고 있다고 알려줌

### 해결

- 상태 변경 대기 중인 테스트 시작 시 테스트하려하는 코드를 포함하는 방법
- `Axios` 프로미스가 해결되고 상태를 업데이트할 때 변경이 일어나므로 흐름의 일부로 변경을 기다리는 테스트에 포함할 수 있음
- 총계가 0으로 시작하는 테스트는 포함하지 않아도 됨
  - 매번 테스트를 실행할 필요가 없음

```jsx
describe('grand total', () => {
  // 총계가 0으로 시작하는 테스트는 포함하지 않아도 됨
  /*
		test("총 합은 0달러로 시작해야 함", () => {
    render(<OrderEntry />);
    const grandTotal = screen.getByRole("heading", {
      name: /grand total: \$/i,
    });
    expect(grandTotal).toHaveTextContent("0.00");
  });
	*/
  // ...
})
```

> **Referenced**

- [Testing React with Jest and React Testing Library (RTL)](https://www.udemy.com/course/understanding-typescript/)
