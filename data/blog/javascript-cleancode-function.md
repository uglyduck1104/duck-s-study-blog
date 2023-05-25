---
title: 함수 다루기
date: '2023-05-25'
tags: ['javascript']
draft: false
summary: 함수, 메서드, 생성자, argument & parameter, 복잡한 인자 관리하기, Default Value, Rest Parameters, void & return, 화살표 함수, Callback Function, 순수 함수
layout: PostSimple
authors: ['default']
---

# 함수 다루기

## 함수, 메서드, 생성자

```jsx
/**
 * 함수, 메서드, 생성자
 */
// 함수
function func() {
  return this
}

// 객체의 메서드
const obj = {
  method() {
    return this
  },
  conciseMethod() {
    return this
  },
}

// 생성자 함수 (Class)
function Func() {
  return this
}

/**
 * 함수
 * 1급 객체
 * - 변수나, 데이터에 담길 수 있다
 * - 매개변수로 전달 가능 (콜백 함수)
 * - 함수가 함수를 반환 (고차 함수)
 */
func()

// 메서드 => 객체에 의존성이 있는 함수, OOP 행동을 의미
obj.method()

// 생성자 함수 => 인스턴스를 생성하는 역할 => Class
const instance = new Func()
```

- 함수의 `this` → global(전역 객체)을 바라봄
  - 서브 루틴, 서브 프로그램이라고 할 정도로 핵심적으로 사용
- 메서드 `this` → 호출된 객체를 바라봄
  - 객체의 의존성이 있음
  - 내장 빌트인 객체(`.`) 표기법 앞의 타입을 따름
  - 기존의 콜론(`:`)을 사용하는 방식에서 `Concise` 메서드를 활용해 간결하게 사용할 수 있게됨
- 생성자 함수 `this` → 생성될 인스턴스를 가리킴
- 함수, 메서드, 생성자 함수를 언제 사용하고, 언제 구별해서 분리할지 판단이 됨

## argument & parameter

```jsx
/**
 * Parameter (Formal Parameter)
 *
 * 형식을 갖춘, 매개변수
 */
function axios(url) {
  // some code
}

/**
 * Argument (Actual Parameter)
 *
 * 실제로 사용되는, 인자 or 인수
 */
axios('https://github.com')
```

- 함수(`function`)의 정의에서 사용되는 곳을 `parameter`라고 함
- 실제 함수를 호출하고 있는 곳에서는 `argument`라고 함(`MDN`에서는 `real values`라고 표현)

> Referenced Link  
> [Parameter - MDN Web Docs Glossary: Definitions of Web-related terms | MDN](https://developer.mozilla.org/en-US/docs/Glossary/Parameter)

## 복잡한 인자 관리하기

### Case 1. 객체 구조 분해 할당 관점

```jsx
function createCar(name, brand, color, type) {
  return {
    name,
    brand,
    color,
    type,
  }
}
```

- 늘어나는 `parameter`에서 무엇이 `name`, `brand`, `color`, `type`인지 뜻이 모호해짐(순서를 지켜야 함)

  - 현재 모던 브라우저는 객체 구조 분해 할당이 가능하지만 `ES6` 이전의 코드에서는 `options`라는 `parameter`를 순회해서 값을 꺼내야 했음

    ```jsx
    function createCar(options) {
      return {
        name: options.name,
        brand: options.brand,
        color: options.color,
        type: options.type,
      }
    }
    ```

```jsx
function createCar({ name, brand, color, type }) {
  return {
    name,
    brand,
    color,
    type,
  }
}
```

- `ES6`부터 등장한 구조 분해 할당 개념으로 복잡한 `parameter`를 관리할 때 순서를 지킬 필요가 없음
- 구조분해 할당을 구분해서 활용할 수 있음

  ```jsx
  function createCar(name, { brand, color, type }) {
    return {
      name,
      brand,
      color,
      type,
    }
  }

  createCar('차량 이름', {
    type: '타입',
  })
  ```

  - 첫번쨰 인자는 명시적으로 중요하다고 인지 시킬 수 있음
  - 나머지 인자는 `options` 값으로 접근할 수 있음

### Case 2. 에러 케이스 처리

```jsx
/**
 * 복잡한 인자 관리하기
 */
function createCar({ name, brand, color, type }) {
  if (!name) {
    throw new Error('name is a required')
  }

  if (!brand) {
    throw new Error('brand is a required')
  }
}

createCar({ name: 'CAR', type: 'SUV' })
```

- `parameter`의 값을 사용해 `Error` 객체를 출력하면 같이 협업하는 팀원들이 실수하지 않도록 사전에 방지하고 알려줄 수 있도록 활용할 수 있음(명시적으로 노출)

## Default Value

### Case 1. ES5 기본값 처리

```jsx
/**
 * default value
 */
function createCarousel(options) {
  options = options || {}
  var margin = options.margin || 0
  var center = options.center || false
  var navElement = options.navElement || 'div'

  // ..some code
  return {
    margin,
    center,
    navElement,
  }
}

createCarousel()
```

- `options parameter`를 사용해 넘겨받은 객체의 값이 `undefined`일 경우를 기본값 할당을 통해 예외로 처리할 수 있음(방어코드 로직 추가)
  - `||` OR 연산자로 처리해서 `defaultValue`를 할당 → 모던 브라우저에서는 ??(`Nullish Operator`)를 대체로 사용할 수 있음

### Case 2. 객체 구조 분해로 기본값 처리

```jsx
/**
 * default value / default parameter
 */
function createCarousel({ margin = 0, center = false, navElement = 'div' } = {}) {
  // ..some code
  return {
    margin,
    center,
    navElement,
  }
}

createCarousel(undefined)
```

- `ES2015`의 문법으로서, 함수의 인자가 `undefined`인 경우에 `default parameter`를 설정해서 오류가 발생하지 않게 할 수 있음

### Case 3. deafult parameter 예외 처리

```jsx
/**
 * default value
 */
const required = (argName) => {
  throw new Error('required is ' + argName)
}

function createCarousel({
  items = required('items'),
  margin = 0,
  center = false,
  navElement = 'div',
} = {}) {
  // ... some code

  return {
    margin,
    center,
    navElement,
  }
}

console.log(
  createCarousel({
    margin: 10,
    center: true,
    navElement: 'span',
  })
)
```

- `items = required('items')`
  - `required` 함수의 `argument`에 `argName`을 `parameter`로 받아 해당하는 속성 값이 존재하지 않을 경우에 `throw new Error`로 예외 메시지를 출력
  - 명시적이고 안전하게 코드 작성이 가능

## Rest Parameters

### Case 1. arguments 객체

```jsx
/**
 * Rest Parameters
 */
function sumTotal() {
  Array.isArray(arguments)

  return Array.from(arguments).reduce((acc, curr) => acc + curr)
}

sumTotal(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```

- `sumTotal` → 가변 인자들을 받아서, 모두 더하는 함수
  - 화살표 함수를 사용하지 않았을 때를 가정해, 내부 `arguments` 객체를 순회하여 `parameter`로 받은 값들을 가함
- 이후에 추가되는 `parameter`를 유연하게 처리하지 못함
- `arguments`는 유사 배열(`객체`)이기 때문에, `Array.from`을 사용해야 함

### Case 2. Rest parameters 활용

```jsx
/**
 * Rest parameters
 */
function sumTotal(initValue, bonusValue, ...args) {
  return args.reduce((acc, curr) => acc + curr, initValue)
}

sumTotal(100, 99, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```

- 나머지 매개변수는 항상 가장 마지막에 위치해야 함(`Spread Operator`와 다름)
- `arguments` 객체와 다르게 순수한 배열로 순회할 수 있음

## void & return

### Case 1. void case

```jsx
/**
 * void & return
 */
function handleClick() {
  return setState(false)
}

function showAlert(message) {
  return alert(message)
}

function test(sum1, sum2) {
  const result = sum1 + sum2
}

function testVoidFunc() {
  return test(1, 2)
}

testVoidFunc()
```

- 코드의 흐름에 따라 `void`(반환값이 없는) 함수인 경우에도 습관적으로 `return`을 사용하는 경우가 있음
- `setState`나 `alert`의 경우에는 반환값이 없으므로 `return` 키워드를 사용하지 않아도 됨
- 자바스크립트는 기본적으로 반환값이 없을 때, `undefined`를 반환
- 내가 사용하는 `return` 값이 있는 지, 없는 지 확인해야 함
  > Referenced Link  
  > [void operator - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void)

### Case 2. return case

```jsx
/**
 * void & return
 */
function isAdult(age) {
  return age > 19
}

function getUserName(name) {
  return '유저 ' + name
}

const isFlag = isAdult(10)
```

- `return` 값이 명확할 때, 값 그 자체로 함수를 변수에 할당할 수 있음

## 화살표 함수

### Case 1. 화살표 함수를 잘못 사용하는 경우(`this` 키워드)

```jsx
/**
 * Arrow Function
 */
const user = {
  name: 'Poco',
  getName: () => {
    return this.name
  },
}

user.getName()
```

- `getName method`에서 `this` 키워드는 `Poco`를 가리키는걸 기대하지만, 실제로 그러지 못함
  - `getName: () => {...}`(Bad 🤦🏻‍♂️) → `getName() {...}`(Good 🙆🏻‍♂️)
    - 화살표 함수에서의 this는 렉시컬 스코프를 따르기 때문에, 화살표 함수를 메서드에 사용하는 경우 `this`는 호출된 객체를 바라보지 못함

### Case 2. 화살표 함수를 잘못 사용하는 경우(arguments 객체)

```jsx
/**
 * Arrow Function
 */
const user = {
  name: 'Poco',
  getName: () => {
    return this.name
  },
  newFriends: () => {
    const newFriendList = Array.from(arguments)

    return this.name + newFriendList
  },
}

// user.newFriends('Jang', '장')
```

- `arguments` 객체를 사용하지 못하는 것 뿐 아니라, `call`, `apply`, `bind`과 같은 메서드를 사용하지 못함
- `arguments` 객체 대신 `rest parameter`를 사용할 수 있음

### Case 3. 화살표 함수를 잘못 사용하는 경우(생성자 함수)

```jsx
const Person = (name, city) => {
  this.name = name
  this.city = city
}

const person = new Person('poco', 'korea')
```

- 화살표 함수를 사용하는 경우 생성자 함수를 사용하지 못함
- 최근엔 `class` 키워드를 사용할 수 있으므로, 화살표 함수를 사용할 이유가 없음
- `new` 연산자를 사용하는 경우 선언부에서 화살표 함수를 사용하면 오류 발생

### Case 4. 화살표 함수를 잘못 사용하는 경우(생성자 함수)

```jsx
class Parent {
  parentMethod() {
    console.log('parentMethod')
  }

  parentMethodArrow = () => {
    console.log('parentMethodArrow')
  }

  overrideMethod = () => {
    return 'Parent'
  }
}

class Child extends Parent {
  childMethod() {
    super.parentMethod()
  }

  overrideMethod() {
    return 'Child'
  }
}

new Child().childMethod()
new Child().overrideMethod()
```

- 화살표 함수는 `class`에서 구현을 했을 때, 생성자 함수 내부에서 초기화되는 문제가 발생
- 동일한 이름의 메서드를 부모에서도 만들고, 상속받는 자식에서도 오버라이드하면 부모의 메서드가 호출됨
  - 부모 클래스에서 정의한 메서드를 다형성의 관점에서 자식 클래스에서 오버라이드하여 재사용하려고 하기를 기대했지만, 결과는 다르게 나옴
  - 만약에 동일한 메서드를 오버라이드해서 사용하고 싶은 경우에 부모 클래스의 화살표 함수를 일반 함수로 만들면 해결됨

## Callback Function

- 대부분 콜백함수는 `async`/`await` 혹은 `promise`와 같이 비동기를 제어하는 함수라고 생각함
  - 함수의 실행권을 다른 함수에 위임하는 관점으로 해석할 수 있음
  - ex) `someElement.addEventListener('click', function(e){...});`

### Case 1. 개선 전

```jsx
/**
 * Callback Function
 */
function register() {
  const isConfirm = confirm('회원가입에 성공했습니다.')

  if (isConfirm) {
    redirectUserInfoPage()
  }
}

function login() {
  const isConfirm = confirm('로그인에 성공했습니다.')

  if (isConfirm) {
    redirectIndexPage()
  }
}
```

- 동일한 결과에 따라서, 동작이 다르므로 서로 다른 동작을 하는 메서드를 생성함

### Case 2. 개선 후

```jsx
/**
 * Callback Function
 */
function confirmModal(message, cbFunc) {
  const isConfirm = confirm(message)

  if (isConfirm && cbFunc) {
    cbFunc()
  }
}

function register() {
  confirmModal('회원가입에 성공했습니다.', redirectUserInfoPage)
}

function login() {
  confirmModal('로그인에 성공했습니다.', redirectIndexPage)
}
```

- 동일하게 동작하는 함수를 추상화하고, 추상화한 함수 인자에 `message`와 다르게 동작하는 `callback` 함수를 전달해서 동적으로 처리

## 순수 함수

- `Redux`의 공식 문서를 보면, 순수 함수란 아래 항목과 같이 `Side Effect`를 일으키지 않는 함수를 말함
  1. `console`의 값을 로깅
  2. 파일 저장
  3. 비동기 타이머 설정
  4. `AJAX HTTP` 요청을 만듦
  5. 함수의 바깥에서 상태를 변경하거나 인수의 변이가 일어남
  6. 랜던됨 숫자나 고유한 랜덤 ID를 생성하는 유틸리티 성 함수(`Math.random()`, `Date.now()`)

### Case 1. Primitive Value

```jsx
/**
 * Pure Function
 */
let num1 = 10
let num2 = 20

function impureSum1() {
  return num1 + num2
}

function impureSum2(newNum) {
  return num1 + newNum
}

function pureSum(num1, num2) {
  return num1 + num2
}

pureSum(10, 20)
pureSum(10, 20)
pureSum(10, 20)
pureSum(30, 100)
pureSum(30, 100)
```

- 동일한 인자를 받고 동일한 결과를 출력해야 함
- 외부에서 값이 변경되는 경우, 함수 내부의 상태가 변경되면 안됨
  1. 값의 일관성이 꺠짐
  2. 값을 예측할 수 없음

### Case 2. Referenced Value

```jsx
/**
 * Pure Function
 */

function changeValue(num) {
  num++

  return num
}

////////////////////////////////

const obj = { one: 1 }

function changeObj(targetObj) {
  targetObj.one = 100

  return targetObj
}

changeObj(obj)

obj // { one: 100 }
```

- `changeValue` 함수는 원시 값을 다루므로 정상적인 결과를 반환
- `changeObj` 함수는 객체이므로, 원시 값을 다루지 않기때문에 원본 객체가 변이됨

  ```jsx
  // 객체, 배열 => 새롭게 만들어서 반환
  function changeObj(targetObj) {
    return { ...targetObj, one: 100 }
  }

  changeObj(obj) // { one: 100 }

  obj // { one: 1 }
  ```

> **Referenced**

- [클린코드 자바스크립트](https://www.udemy.com/course/clean-code-js/)
