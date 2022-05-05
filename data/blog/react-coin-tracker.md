---
title: React Simple Coin Tracker Example
date: '2022-05-04'
tags: ['react']
draft: false
summary: ToDos 예제로 알아보는 Effect 개념
layout: PostSimple
authors: ['default']
---

# Simple Coin Tracker

- 페이지나 앱을 들어왔을 때 API를 호출 상태에 따른 로딩 메시지를 노출하고 coin 정보의 목록을 출력

```javascript
import { useState } from 'react'

import { useEffect, useState } from 'react'

function App() {
  const [loading, setLoading] = useState(true)
  const [coins, setCoins] = useState([])
  useEffect(() => {
    fetch('https://api.coinpaprika.com/v1/ticker')
      .then((response) => response.json())
      .then((json) => {
        setCoins(json)
        setLoading(false)
      })
  }, [])
  return (
    <div>
      <h1>The Coins! {loading ? '' : `(${coins.length})`}</h1>
      {loading ? (
        <strong>Loading...</strong>
      ) : (
        <select>
          {coins.map((coin) => (
            <option key={coin.id}>
              {coin.name} ({coin.symbol}): {coin.price_usd} USD
            </option>
          ))}
        </select>
      )}
    </div>
  )
}

export default App
```

## Execute Sequences

1. 컴포넌트가 가장 처음 render 될 때, useEffect로 api를 호출
   ```javascript
   useEffect(() => {
     // Todo
   }, [])
   ```

- 두번째 인자가 빈 배열이면 코드는 최초 한 번만 실행
- URL을 fetch 요청하고 응답받은 데이터로부터 json을 추출
- 추출한 json 데이터를 `coins state` 배열에 갱신 및 저장
- `true`값으로 초기화된 `loading state`를 `false` 처리

2. 변환된 state 값에 따라서 component에 노출

- `loading state`
  ```javascript
  <h1>The Coins! {loading ? '' : `(${coins.length})`}</h1>
  ```
  - loading(true)
    - loading state가 true(기본값)일 경우, 빈 string 값 노출
  - loading(false)
    - 1번 과정에서 loading state를 `false`로 처리되었다는 건, status code가 200(success)일 경우를 암시
    - `coins state` 배열의 `length(배열의 길이) 값`을 노출
      - coins가 배열로 기본값을 설정한 이유는 적어도 비어있는 array로 두어서 `undefined` 되지 않기 위해서임
  ```javascript
  {
    loading ? (
      <strong>Loading...</strong>
    ) : (
      <select>
        {coins.map((coin) => (
          <option>
            {coin.name} ({coin.symbol}): {coin.price_usd} USD
          </option>
        ))}
      </select>
    )
  }
  ```
  - loading(true)
    - loading state가 true(기본값)일 경우, "Loading..." 텍스트를 노출
  - loading(false)
    - map 고차함수를 이용해 `coins state`의 배열을 순회하면서 배열안에 있는 각각의 `coin` 값을 반환

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
