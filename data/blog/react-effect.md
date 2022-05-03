---
title: React Effect
date: '2022-04-29'
tags: ['react']
draft: false
summary: 생명주기 componentDidMount, componentDidUpdate와 같은 방식으로 수행
layout: PostSimple
authors: ['default']
---

# Effect

- 생명주기 componentDidMount, componentDidUpdate와 같은 방식으로 수행
  - 컴포넌트가 최초 렌더링될 때, 한 번만 실행되는 코드를 작성할 때 주로 사용

```typescript
function useEffect(effect: EffectCallback, deps?: DependencyList): void
```

- 첫번째 argument(effect)
  - callback function
- 두번째 argument(deps)
  - `[]`: 최초 한번만 실행
  - `[STATE]`: 선언한 state가 갱신될 때 실행
  - 이전 렌더링 시의 값과 그다음 렌더링 때의 값과 비교하여 같지 않을때에만 재실행

## 일반적인 상황

```javascript
import { useState } from 'react'

function App() {
  const [counter, setValue] = useState(0)
  const onClick = () => setValue((prev) => prev + 1)
  console.log('항상 실행')
  return (
    <div>
      <h1>{counter}</h1>
      <button onClick={onClick}>Click me</button>
    </div>
  )
}

export default App
```

- Click 버튼을 누르면 `counter state`가 1씩 증가함
- state(value)의 변동사항이 생길 때, 모든 컴포넌트는 다시 실행됨
  - onClick 이벤트를 계속 실행하면 state의 변동사항으로 인해 아래의 `console.log("항상 실행);`이 지속적으로 출력됨
- API Call Code까지 다시 실행(리소스 낭비)되므로 특정 한 시점을 기준으로 딱 한번만 호출하고 더이상 실행되지 않도록 해야함

## useEffect 적용

```javascript
import { useState, useEffect } from 'react'

function App() {
  const [counter, setValue] = useState(0)
  const [keyword, setKeyword] = useState('')
  const onClick = () => setValue((prev) => prev + 1)
  const onChange = (event) => setKeyword(event.target.value)
  console.log('항상 실행')
  useEffect(() => {
    console.log('한번만 실행')
  }, [])
  useEffect(() => {
    if (keywork !== '' && keyword.length > 5) console.log('SEARCH FOR')
  }, [keyword])
  return (
    <div>
      <input value={keyword} onChange={onChange} type="text" placeholder="Search here..." />
      <h1>{counter}</h1>
      <button onClick={onClick}>Click me</button>
    </div>
  )
}

export default App
```

- input field에 텍스트를 입력하면 onChange 이벤트 핸들러에 의해 onChange 함수가 실행되고, keyword state가 갱신됨
- useEffect hook으로 키워드가 적어도 빈 문자열이아니고, keyword의 length가 6이상일 때 `console.log("SEARCH FOR");`가 실행됨
  - [] 빈 배열과 달리 정의한 두번째 인자(deps = dependency list)에 `keyword state`로 제한을 둠
    - 두번째 인자에 `[keyword, counter]`처럼 하나의 이상의 state를 감지할 수 있음
  - 해당 조건을 충족하는 keyword의 상태에서만 실행됨

> useEffect는 컴포넌트가 생성될때 혹은 갱신될때 코드 실행 조건을 제어할 수 있음\
> React는 effect가 수행되는 시점에 이미 DOM이 업데이트되었음을 보장함

## Cleanup function

```javascript
import { useState, useEffect } from 'react'

function Hello() {
  useEffect(() => {
    console.log('created :)')
    return () => console.log('destroyed :(')
  }, [])
  return <h1>Hello</h1>
}

function App() {
  const [showing, setShowing] = useState(false)
  const onClick = () => setShowing((prev) => !prev)
  return (
    <div>
      {showing ? <Hello /> : null}
      <button onClick={onClick}>{showing ? 'Hide' : 'Show'}</button>
    </div>
  )
}

export default App
```

- showing boolean 값에 따라서 Hello 컴포넌트를 생성하거나 삭제함
- `Show` 버튼을 클릭했을 때, prev(이전 값)를 반전시켜 showing의 state 값을 변경
  - 바뀐 state에 따라 컴포넌트가 다시 렌더링되고 Hello 컴포넌트가 생성
- useEffect 첫번째 인자의 함수를 반환해서, 컴포넌트 소멸 시점에 함수를 실행할 수 있음

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
