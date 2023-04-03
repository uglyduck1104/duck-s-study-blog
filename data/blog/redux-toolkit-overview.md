---
title: RTK(Redux Toolkit) Overview
date: '2023-04-03'
tags: ['redux']
draft: false
summary: Redux Toolkit과 Redux의 관계 및 개요, 간단한 예제
layout: PostSimple
authors: ['default']
---

# RTK(Redux Toolkit) Overview

## Redux & React Redux & Redux toolkit

### Redux

- 상태관리자로써 프로그램이 동작하는 상태를 체계적으로 관리해줌
- 리덕스 자체는 리액트와는 무관하게 자바스크립트 앱을 위한 예측 가능한 상태 컨테이너로써 서로 다른 환경(서버, 클라이언트, 네이티브)에서 작동하고 테스트하기 쉬운 앱을 작성하도록 도와줌
- 리덕스는 매우 작지만(2kb) 사용 가능한 애드온은 매우 많음

### React Redux

- 리액트에서 사용하기 쉽게 만들기 위해 해결해준 도구로써, useHook을 사용해 dispatch 및 selector를 사용할 수 있음

### Redux Toolkit

- Redux를 적용하기 위해 기존의 복잡한 설정(특히 미들웨어)과 보일러 플레이트 등의 코드를 단순 간결하게 작성할 수 있도록 많은 유틸리티 기능을 제공하여 이를 통해 Redux 앱을 더 빠르고 쉽게 작성할 수 있음
- 불변성 유지하는 라이브러리(`Immer`)가 기본적으로 내장되어 있음
- RTK(Redux Toolkit)에 포함되어있는 애드온의 확장성
- Redux Toolkit을 사용하면 아래와 같은 기능을 활용할 수 있습니다.
  - `configureStore` - Redux store를 생성하는 유틸리티 함수
  - `createSlice` - reducer와 action을 생성하는 유틸리티 함수
  - `createAsyncThunk` - 비동기 작업을 처리하는 유틸리티 함수
  - `createEntityAdapter` - 일반적으로 백엔드 API의 데이터를 처리하는 유틸리티 함수
  - 또한, `Redux Toolkit`은 `Immer` 패키지를 내부적으로 사용하기 때문에, 불변성을 유지하면서 쉽게 상태를 업데이트할 수 있습니다.

## 설치

Redux Toolkit은 모듈 번들러 또는 노드 애플리케이션과 함께 사용하기 위해 NPM에서 패키지로 제공됨

```bash
# NPM
npm install @reduxjs/toolkit

# Yarn
yarn add @reduxjs/toolkit
```

## Chrome Extension - Redux DevTools

- `Redux DevTools`를 이용해 상태가 변하는 스냅샷을 확인 가능하고, `RTK Query` 디버거 툴 등 지원하는 기능이 많음

[Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=ko)

## 예제

### `counterSlice.js`

```jsx
import { createSlice } from '@reduxjs/toolkit'
const counterSlice = createSlice({
  name: 'counterSlice',
  initialState: { value: 0 },
  reducers: {
    up: (state, action) => {
      state.value = state.value + action.payload
    },
  },
})
export default counterSlice
export const { up } = counterSlice.actions
```

- `redux toolkit`에서 제공하는 `createSlice` 함수를 `import` 한 후, `counterSlice` 객체를 생성
  - 해당 객체는 `Redux`의 `createStore`와 같은 역할을 함
- `name` 속성을 사용하여 이름을 정의하고 `initialState`로 값의 초기 상태를 정의
- `reducers`의 속성은 슬라이스에서 처리할 수 있는 액션 타입 및 리듀서 함수를 정의
  - `up` 액션 타입을 정의하고, `up` 액션을 처리하는 리듀서 함수를 정의
  - `up` 함수는 불변성을 유지하기 위해 `state` 객체의 값을 수정하지 않고 `action.payload`의 값을 이용해 새로운 `value` 값을 계산 한 후 반환
- 해당 함수를 `export` 키워드로 내보낸 후, `dispatch` 함수로 스토어에 보내면 상태를 변경할 수 있음

### `store.js`

```jsx
import { configureStore } from '@reduxjs/toolkit'
import counterSlice from './counterSlice'
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
  },
})
export default store
```

- `configureStore` 함수를 사용해서 스토어를 생성하고, 위에서 작성한 `counterSlice`를 `reducer` 속성을 사용하여 스토어의 리듀서 함수를 지정
- `counterSlice` 모듈에서 정의한 액션 타입과 리듀서 함수를 처리할 수 있음

### `App.js`

```jsx
import React from 'react'
import { createStore } from 'redux'
import { Provider, useSelector, useDispatch } from 'react-redux'
import store from './store'
import { up } from './counterSlice'

function Counter() {
  const dispatch = useDispatch()
  const count = useSelector((state) => {
    return state.counter.value
  })
  return (
    <div>
      <button
        onClick={() => {
          dispatch(up(2))
        }}
      >
        +
      </button>{' '}
      {count}
    </div>
  )
}
export default function App() {
  return (
    <Provider store={store}>
      <div>
        <Counter></Counter>
      </div>
    </Provider>
  )
}
```

- `Counter` 컴포넌트는 함수를 사용해 스토어의 상태를 조회(`useSelector`)하거나, 액션 생성자 함수를 호출하여 액션 객체를 생성한 후 `dispatch` 함수를 이용해 액션 객체를 스토어에 보냄(`useDispatch`)
- `Provider`, `useSelector`, `useDispatch` 함수를 `import` 한 후, `App` 컴포넌트에 `Provider`로 `store` 객체를 전달

> **Referenced**

- https://ko.redux.js.org/
- https://www.youtube.com/watch?v=9wrHxqI6zuM
- https://redux.js.org/redux-toolkit/overview
