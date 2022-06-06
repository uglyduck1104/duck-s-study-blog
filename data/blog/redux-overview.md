---
title: Redux
date: '2022-06-06'
tags: ['redux']
draft: false
summary: Redux는 React, Angular, Vue, Ember 및 바닐라 JS를 포함한 모든 UI 계층 또는 프레임워크와 함께 사용할 수 있는 독립 실행형 라이브러리
layout: PostSimple
authors: ['default']
---

# Redux

## 소개

<img alt="redux_overview" src="/static/images/redux_overview.png" width="600"/>

- 국내외 redux를 사용하는 사이트는 6월 6일 기준 `57,410`곳에 사용중임
- React용 공식 Redux `UI 바인딩 라이브러리`
  - 다른 UI 레이어와 함께 사용할 수 있지만 원래는 `React와 함께 사용하도록 설계`되었음
- Redux 자체는 React, Angular, Vue, Ember 및 바닐라 JS를 포함한 모든 UI 계층 또는 프레임워크와 함께 사용할 수 있는 `독립 실행형 라이브러리`
  - 주로 React와 함께 사용되나 상호 `독립적`임
  - UI 코드에서 직접 상호 작용하는 것이 아닌 UI 바인딩 라이브러리 사용을 통해 UI 프레임워크와 연결

### 실행 로직 순서

1. Redux 스토어 생성
2. 업데이트 구독
3. 구독 콜백 내부:
   1. 현재 스토어 상태 가져오기
   2. UI에 필요한 데이터 추출
   3. 데이터로 UI 상태 업데이트
4. 필요한 경우 초기 상태로 UI 렌더링
5. Redux 작업을 통해 UI 입력에 응답

> 위와 같은 순서를 매번 반복해서 처리하려면 복잡한 로직이 필요하므로, 해당 프로세스를 재사용 가능하게 만들기 위해 등장함

### 등장 배경

<img alt="redux_overview2" src="/static/images/redux_overview_2.png" width="700"/>

- 부모에서 자식 구성 요소로 상태를 전달하거나 prop으로 상태를 전달하는 경우, depth가 깊어지면 깊어질수록 거쳐야되는 횟수가 많아지는데(`Component Drilling`) 이런 문제점을 `store`라는
  저장소를 통해 쉽게 가져올 수 있음
  - 즉 모든 애플리케이션 `상태를 한 곳(Store)에 저장`하고 `특정 시점의 상태`를 알 수 있음
  - 또 React를 구성하는 요소 트리에서 필요한 저장소 상태의 특정 데이터 조각을 추출하여 `불필요한 구성 요소를 다시 렌더링하는 것을 예방`함

### 리덕스 원칙

- 스토어는 직접 변경할 수 없고 항상 액션에 의해서 `새로운 상태를 반환`함
- 상태 변경은 순수 함수인 Reducer에 의해 처리되고 Reducer는 상태를 업데이트 함
  - Reducer는 `현재 상태와 액션`을 받아 `새로운 상태를 반환하는 함수`
  - 하나 이상의 Reducer에 의해서 각 작업은 `단일 저장소`에 업데이트 함

## Action

- 상태 업데이트는 핵심 작업 중 하나이므로 모든 상태 업데이트는 dispatch action에 의해 트리거 됨
- 액션은 `자바스크립트 객체`로서, `액션 이벤트`에 대한 정보를 담고 있음
  - `속성`(필수)과 `페이로드`(선택)를 가지고 있음

```javascript
const action = {
  type: 'LOGIN'
};

// 액션 생성자
const actionCreator = () => {
  return action;
}

// 다음과 동일하게 동작
/*
const actionCreator = () => {
  return { type: 'LOGIN'}
}
*/
```

- 작업을 생성한 후 상태를 업데이트할 수 있도록 Redux Store에 Action 전달
- `액션 생성자`는 단순히 액션을 반환하는 JavaScript 함수

### Dispatch Method를 이용해 Action Event 전달하기

- dispatch 메서드는 Redux Store에 전달하기 위한 메서드로서, `store.dispatch()`를 호출하고 action 생성자로부터 반환된 값을 store로 전달함

```javascript
const store = Redux.createStore(
  (state = {login: false}) => state
);
const loginAction = () => {
  return {type: 'LOGIN'}
};

// Dispatch the action 
store.dispatch(loginAction());
```

- state값에 login을 false 값으로 초기화 한 후, 'LOGIN' type를 반환하는 함수인 `loginAction`을 store에 전달

## Reducers

- Reduce 함수는 단순히 `상태`와 `동작`을 가진, `새로운 상태만 반환`하는 순수 함수

```javascript
/*
function myReducer(state = initialState, action){
    전달된 작업을 기반으로 새로운 상태를 반환함
}
 */

// Counter 상태값을 증가
function myReducer(state, action) {
  switch (action.type) {
    case "INCREMENT_COUNTER":
      return {...state, counter: state.counter + 1};
    default:
      return state;
  }
}
```

- 기존의 state 값을 `직접 변경시키지 않고`, `새로운 객체를 생성`하여 카운터를 증가
  - `새로 반환된 객체`의 업데이트 된 상태값을 사용하여 저장소를 업데이트

```javascript
const LOGIN = 'LOGIN';
const LOGOUT = 'LOGOUT';
const defaultState = {
  authenticated: false
};
const authReducer = (state = defaultState, action) => {
  switch (action.type) {
    case LOGIN:
      return {...state, authenticated: true}
    case LOGOUT:
      return {...state, authenticated: false}
    default:
      return state
  }
};
const store = Redux.createStore(authReducer);
const loginUser = () => {
  return {type: LOGIN}
};
const logoutUser = () => {
  return {type: LOGOUT}
};
```

- switch 문을 사용해 `action type`에 해당하는 case 문을 정의하고 여러 작업을 처리함
- loginUser, logoutUser의 타입을 `상수 타입의 문자열`로 선언하여 휴먼 에러(오타)를 방지

## Store

```javascript
const reducer = (state = 5) => {
  return state;
}

let store = Redux.createStore(reducer);
```

- 애플리케이션 `상태를 유지`하고 `관리`하는 객체
  - Redux 저장소를 생성하는 데 사용하는 `createStore()`라는 메서드를 사용
  - `reducer` 함수는 `필수 인자 값`
- Store 객체는 상호 작용할 수 있는 메서드 제공함
  - `store.dispatch(action)`
    - reducer를 호출하고 상태를 저장하여 리스너를 실행
  - `store.subscribe(action)`
    - 저장소에 대해 `작업이 디스패치될 때마다` 호출되는 리스너 함수를 저장소에 구독
  - `store.getState()`
    - Redux store의 `현재 상태를 취득`

> **Referenced**

- https://trends.builtwith.com
- https://react-redux.js.org
- https://medium.com/javascript-in-plain-english/redux-with-react-is-simple-3e3480a83432