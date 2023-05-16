---
title: 객체 다루기
date: '2023-05-16'
tags: ['javascript']
draft: false
summary: Shorthand Properties, Computed Property Name, Lookup Table, Object Destructuring, Object.freeze, Prototype 조작 지양하기, hasOwnProperty, 직접 접근 지양하기, Optional Chaning
layout: PostSimple
authors: ['default']
---

# 객체 다루기

## Shorthand Properties

### Case 1. CSS 단축 표현

```css
.before {
  background-color: #000;
  background-image: url(images/bg.gif);
  background-repeat: no-repeat;
  background-position: left top;

  margin-top: 10px;
  margin-right: 5px;
  margin-bottom: 10px;
  margin-left: 5px;
}

.after {
  background: #000 url(images/bg.gif) no-repeat left top;

  margin: 10px 5px 10px 5px;
}
```

- `background` suffix를 기준으로 `color`, `image`, `repeat`, `position`
- `margin` suffix를 기준으로 `top`, `right`, `bottom`, `left`
- 위의 속성을 `.after`에서 단축 표현으로 사용하고 있음

### Case 2. Redux + 객체 리터럴

```jsx
/**
 * Shorthand Properties
 * Reducer
 */
const counterApp = combineReducers({
  counter,
  extra,
})
```

```jsx
/**
 * Shorthand Properties
 * Concise Method
 * ES2015+
 */
const firstName = 'poco'
const lastName = 'jang'

/*
const person = {
  firstName: "poco",
  lastName: "jang",
  getFullName: function () {
    return this.firstName + " " + this.lastName;
  },
};
*/

const person = {
  firstName,
  lastName,
  getFullName() {
    return this.firstName + ' ' + this.lastName
  },
}
```

- `Reducer`의 `combinReducers`와 같이 키 + 값의 조합이 아닌 단축 속성 및 단축 메서드를 사용하는 경우 왜 값이 없는데에도 동작하는가를 짚고 넘어가는 것이 좋음
  - `key`와 `value`의 형태가 동일한 경우에 간결하게 표현할 수 있음
- `Shorthand Properties`와 `Concise Method` 덕분에 가독성이 더 좋아졌고, 특정 속성들을 축약해서 사용할 수 있게 됨
  - 모던 자바스크립트의 형태로 인지

## Computed Property Name(계산된 속성 이름)

### Case 1. Event 객체 활용

```jsx
import React, { useState } from 'react'

function SomeComponent() {
  const [state, setState] = useState({
    id: '',
    password: '',
  })

  const handleChange = (e) => {
    setState({
      [e.target.name]: e.target.value,
    })
  }

  return (
    <React.Fragment>
      <input value={state.id} onChange={handleChange} name="name" />
      <input value={state.password} onChange={handleChange} name="password" />
    </React.Fragment>
  )
}

export default SomeComponent
```

- `[e.target.name]: e.target.value,`
- 속성 이름을 동적으로 받아서 유연하게 처리할 수 있음
  1. `onChange` 함수에 `handleChange` 함수를 호출함
  2. `handleChange`에서 `argument`로 `event` 객체를 전달받음
  3. `event`의 속성 → `target.name`과 `target.value`를 동적으로 할당

### Case 2. Redux

```jsx
const noop = createAction('INCREMENT')

const reducer = handleActions(
  {
    [noop]: (state, action) => ({
      counter: state.counter + action.payload,
    }),
  },
  { counter: 0 }
)
```

- `[noop]` 객체의 키에 동적으로 값을 넘겨받아서 키로써 할당하는 예시

### Case 3. 동적으로 표현식 처리

```jsx
const funcName0 = 'func0'
const funcName1 = 'func1'
const funcName2 = 'func2'

const obj = {
  [funcName0]() {
    return 'func0'
  },
  [funcName1]() {
    return 'func1'
  },
  [funcName2]() {
    return 'func2'
  },
}

for (let i = 0; i < 3; i++) {
  console.log(obj[`func${i}`]())
}
```

- `obj`의 `함수명[]`, `매개변수()`, `함수몸체{}`로 구분된 구조로써, for문을 순회하면서 각 함수의 이름을 동적으로 받아 실행

> `Redux`, `React`와 같은 서드파티 라이브러리 혹은 프레임워크에서도 `Javascript`의 `Computed Properties Name`을 사용해 동적으로 로직을 구현할 수 있음

## Lookup Table

- `JSON`과 같은 형태의 `Key`와 `Value`로 구분된 테이블

### Case 1, 2. `if ~ else if` 조건문과 `switch`로 보는 문제점

```jsx
/**
 * Object Lookup Table
 */
function getUserType(type) {
  if (type === 'ADMIN') {
    return '관리자'
  } else if (type === 'INSTRUCTOR') {
    return '강사'
  } else if (type === 'STUDENT') {
    return '수강생'
  } else {
    return '해당 없음'
  }
}

/* Switch문 */
function getUserType(type) {
  switch (key) {
    case 'ADMIN':
      return '관리자'
    case 'INSTRUCTOR':
      return '강사'
    case 'STUDENT':
      return '수강생'
    default:
      return '해당 없음'
  }
}
```

- 늘어나는 `type` 혹은 `key`를 `else if` 조건문이나 `switch`문을 연달아 쓰게되면 유지보수가 어려워지고 코드의 결과를 예측하기 어려움

### Case 3. Object Lookup Table 함수 1

```jsx
function getUserType(type) {
  const USER_TYPE = {
    ADMIN: "관리자",
    INSTRUCTOR: "강사",
    STUDENT: "수강생",
		UNDEFINED: "해당 없음".
  };

  return USER_TYPE[type] ?? USER_TYPE[UNDEFINED];
}
```

- `type`이라는 매개변수를 받아서 `USER_TYPE`의 `key`값으로 접근
- 단축 평가로 `||` 해당 객체가 존재하지 않으면 `defaultValue`인 해당 없음을 반환
- 혹은 `??(Nullish Operator)`를 사용

## Object Destructuring

### Case 1, 2. 객체 구조 분해 할당을 이용한 매개변수 전달

```jsx
/* 개선 전 */
function Person(name, age, location) {
  this.name = name
  this.age = age
  this.location = location
}

const poco = new Person('poco', 30, 'korea')
```

- 매개 변수의 순서는 항상 `name`, `age`, `location` 순임
- 전달하지 않거나, 필요 없는 매개 변수의 경우 `undefined`로 할당

```jsx
/* 개선 후 */
function Person({ name, age, location }) {
  this.name = name
  this.age = age
  this.location = location
}

const poco = new Person({
  name: 'poco',
  age: 30,
  location: 'korea',
})
```

- 순서를 지키지 않아도 되고, 인스턴스에서 `name`, `age`, `location` 키값으로 구조 분해 할당을 할 수 있음
- 주로 인자가 3개인 경우 사용

### Case 3. 배열 및 객체 구조 분해 할당

```jsx
const orders = ['First', 'Second', 'Third']

const st = orders[0]
const rd = orders[2]

const [st, , rd] = orders
// const { 0: st, 2: rd} = orders;
```

- `orders` 배열을 구조분해 할당하면 순서를 지켜야 하므로, 중간에 빈 값이 발생할 수 있지만, `object` 형태로 구조분해 할당을 하면, `index`와 `value`로 보다 명시적으로 할당할 수 있음

## Object.freeze

```jsx
const STATUS = Object.freeze({
  PENDING: 'PENDING',
  SUCCESS: 'SUCCESS',
  FAIL: 'FAIL',
  OPTIONS: {
    GREEN: 'GREEN',
    RED: 'RED',
  },
})
```

- 객체의 조작을 예방하기 위해 `Object.freeze({})`를 사용해 불변하는 객체를 만듦
  - `Object.isFrozen()` 메서드를 활용하여 해당 객체가 `freezing` 상태인지 `Boolean` 반환 메서드 제공
  - 하지만, `deep freezing`이 지원되지 않으므로 중첩구조의 객체(`깊은 Depth`)는 `freezing`이 되지 않음
- 깊은 `Freezing` 구현

  1. 대중 유틸 라이브러리 사용(`lodash`, `underscore`)
  2. 직접 유틸 함수 생성

     ```jsx
     function deepFreeze(targetObj) {
       Object.keys(targetObj).forEach(key => {
         if (/** 객체가 맞다면 */) {
           deepFreeze(targetObj[key]);
         }
       })

       return Object.freeze(targetObj);
     }
     ```

  3. 객체를 순회
  4. 값이 객체인지 확인
  5. 객체이면 재귀
  6. 그렇지 않으면 `Object.freeze` 그대로 사용
  7. stackoverflow 자문
  8. TypeScript ⇒ readonly

## Prototype 조작 지양하기

```jsx
/**
 * Prototype 조작 지양하기
 *
 * 1. 이미 JS는 많이 발전했다.
 *   1-1. 직접 만들어서 모듈화
 *   1-2. 직접 만들어서 모듈화 => 배포 => NPM
 * 2. JS 빌트인 객체를 건들지말자
 */
class Car {
  constructor(name, brand) {
    this.name = name
    this.brand = brand
  }

  sayName() {
    return this.brand + '-' + this.name
  }
}

const casper = new Car('캐스퍼', '현대')
```

- 자바스크립트는 몽키페치 언어이므로, 런타임에 동작되는 것들을 모두 조작할 수 있음
- 과거에는 빌트인 객체의 프로토타입을 직접 조작하는 경우가 많았지만, 예기치 못한 오류 발생 시 오류 추적이 어려움

## hasOwnProperty

- `Property`를 가지고 있는지 확인하는 메서드 제공
  - `Object.prototype.hasOwnProperty.call(targetObj, targetProp);`
- 자바스크립트에서는 `Object.prototype`에서 확장되기 떄문에, `hasOwnProperty`의 명칭은 외부 `hasOwnProperty`와 겹칠 수 있으므로 올바른 결과들을 얻기 위해서는 외부 `hasOwnProperty`를 권장함

```jsx
/**
 * hasOwnProperty
 */
function hasOwnProp(targetObj, targetProp) {
  return Object.prototype.hasOwnProperty.call(targetObj, targetProp)
}

const person = {
  name: 'hyeonseok',
}

hasOwnProp(person, 'name')

const foo = {
  hasOwnProperty: function () {
    return 'hasOwnProperty'
  },
  bar: 'string',
}

hasOwnProp(foo, 'hasOwnProperty')
```

- `foo` 객체는 문자열을 반환하는 `hasOwnProperty` 메서드를 담고 있음
  - `foo.hasOwnProperty('bar')`를 실행하면 `Object.hasOwnProperty` 메서드가 실행되지 않음

## 직접 접근 지양하기

```jsx
/**
 * 직접 접근 지양하기
 * 예측 가능한 코드를 작성해서 동작이 예측 가능한 앱
 */
// 직접 접근 지양
const model = {
  isLogin: false,
  isValidToken: false,
}

// model에 대신 접근
function setLogin(bool) {
  model.isLogin = bool
  serverAPI.log(model.isLogin)
}

// model에 대신 접근
function setValidToken(bool) {
  model.isValidToken = bool
  serverAPI.log(model.isValidToken)
}

// model에 직접 접근 X
function login() {
  setLogin(true)
  setValidToken(true)
}

// model에 직접 접근 X
function logout() {
  setLogin(false)
  setValidToken(false)
}

someElement.addEventListener('click', login)
```

- 객체를 직접 건드는 영역(레이어)을 분리한 후 호출

## Optional Chaning

```jsx
const response = {
	data: {
		userList: [
			name: 'Jang',
			info: {
				tel: '010',
				email: 'jang@gmail.com'
			}
		]
	}
}

function getUserEmailByIndex(userIndex) {
	if (response?.data?.userList?.[userIndex]?.info?.email){
		return response.data.userList[userIndex].info.email;
	}
	return '알 수 없는 에러가 발생';
}

getUserEmailByIndex(0);
```

- 특정 객체에 접근하기 위해서는 점표기법으로 좌에서 우로 접근해야함
  - 만약, `response`의 데이터에서 `userList`가 없는 경우 런타임 오류가 발생할 수 있음
  - 위와 같은 문제가 발생하지 않기 위해서는, 조건문을 사용할 수 있지만 중첩된 데이터 객체의 구조에서는 유연하게 대처하지 못하는 경우가 발생
- 결과적으로 따져봤을 때, 옵셔널 체이닝(`?`)을 사용해서 유실될 우려가 있는 상황을 예방할 수 있음

> **Referenced**

- [클린코드 자바스크립트](https://www.udemy.com/course/clean-code-js/)
