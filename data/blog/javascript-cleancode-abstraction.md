---
title: 추상화하기
date: '2023-06-07'
tags: ['javascript']
draft: false
summary: Magic Number, 네이밍 컨벤션, DOM API 접근 추상화
layout: PostSimple
authors: ['default']
---

# 추상화하기

## Magic Number

### Case 1. 상수로 추상화

```jsx
// 공통적인 상수는 util.ts, constants.ts 파일로 따로 관리(export 키워드를 사용)
const COMMON_DELAY_MS = 3 * 60 * 1000

setTimeout(() => {
  scrollToTop()
}, COMMON_DELAY_MS)
```

- 일정시간이 지난 후 최상위 스크롤로 이동하는 로직
- 공통적인 지연 시간은 상수로 할당해서 `SnakeCase`와 대문자의 조합으로 변하지 않는 값을 명시

### Case 2. Numeric Operator 활용

```jsx
// Numeric Operator
const PRICE = {
  MIN: 1_000_000, // 1백만원
  MAX: 100_000_000, // 1억
}

console.log(PRICE) // { MIN: 1000000, MAX: 100000000 }
```

- `Numeric Operator`를 활용해 금액과 같은 상수값이 숫자인 경우 특정 숫자의 단위가 직관적으로 보기 힘든 경우 `UnderScore`(\_)를 사용
  - 연산 시, 언더스코어도 원본 값에 영향을 받지 않고 연산이 가능함

### Case 3. 하드코딩된 숫자 따로 관리하기

```jsx
const CAR_NAME_LEN = Object.freeze({
  MIN: 1,
  MAX: 5,
})

function isValidName(name) {
  return carName.length >= CAR_NAME_LEN.MIN && carName.length <= CAR_NAME_LEN.MAX
}

function notiValidName(value) {
  if (!isArrayItemLengthRange(names, CAR_NAME_LEN.MIN, CAR_NAME_LEN.MAX)) {
    alert(`자동차 이름은 ${CAR_NAME_LEN.MIN}자에서 ${CAR_NAME_LEN.MAX}자까지 입력할 수 있습니다.`)
  }
}
```

- `Case 1, 2`와 마찬가지로 하드코딩된 숫자의 정책이 바뀌는 경우를 고려해서 `Object.freeze` 메서드를 사용해 해당 객체를 `readonly` 상태로 만듦

## 네이밍 컨벤션

- 저장소, 폴더, 파일, 함수, 변수, 상수, 깃 브랜치, 커밋 등 프로그래밍 전반적으로 이름을 네이밍을 위한 규칙이나 관습을 만드는 것
- 팀이나 개인의 차원에 따라 다를 수 있으며 특히 개인적인 견해와 해석에 따라 다르고, 기준을 정할때 기본적인 논리와 이유가 있어야 함

### 대표적인 케이스

- `camelCase`
  - 일반적으로 자바스크립트에서 빈도수 높게 사용
- `PascalCase`
  - 함수 생성자나 클래스 혹은 컴포넌트, ENUM 명에 사용
- `kebab-case`
  - NPM 패키지나, 저장소명, 파일기반의 라우팅을 사용하는 NextJS, Nuxt, REMIX…
- `SNAKE_CASE`
  - 상수 사용

### 접두사, 접미사

- `prefix-*`, `*-suffix`
  - `data` 식별자 속성에서 사용(`prefix`)
    - `data-id`, `data-name`, `data-value`
  - `Container`나 `Component` 작명 사용(`suffix`)
    - `AppContainer`, `BoxContainer`, `ListComponent`, `ItemComponent`
  - `ICar`, `TCar`(`prefix`)
    - `interface`, `type alias` 명시적 표기 사용
- `동사-*`
  - 함수는 동사로 시작함
  - `_`, `#`

### 연속적인 규칙

```text
for (let i = 0; i < 10; i++) {
  for (let j = 0; j < 10; j++) {}
}

function func<T, U, V>(name: T, value: U)
```

### 자료형 표현

```text
const inputNumber = 10
const someArr = []
const strToNum = 'some code'
```

### 이벤트 표현

```text
function on-*
function handle-*
function *-Action
function *-Event
function take-*
function *-Query
function *-All
```

### CRUD

```text
function generator-*
function gen-*
function make-*
function get
function set
function remove
function create
function delete
```

### Flag

```text
const isSubmit
const isDiasbled
const isString
const isNumber
```

### ETC

```text
function selectById(id)
function selectrAll
```

> 자바스크립트에서 지정한 키워드 혹은 예약어와 중복되지 않게 작성해야 함(⭐⭐⭐)

## DOM API 접근 추상화

```jsx
const createLoader = () => {
  const el = document.createElement('div')
  const el2 = document.createElement('div')
  const el3 = document.createElement('div')

  return {
    el,
    el2,
    el3,
  }
}

const createLoaderStyle = ({ el, el2, el3 }) => {
  el.setAttribute('class', 'loading d-flex justify-center mt-3')
  el2.setAttribute('class', 'relative spinner-container')
  el3.setAttribute('class', 'material spinner')

  return {
    newEl: el,
    newEl2: el2,
    newEl3: el3,
  }
}

const loader = () => {
  const { el, el2, el3 } = createLoader()
  const { newEl, newEl2, newEl3 } = createLoaderStyle({ el, el2, el3 })

  newEl.append(newEl2)
  newEl2.append(newEl3)

  return newEl
}

const changeColor = (element) => {
  element.style.backgroundColor = 'black'
}

const openModal = (element) => {
  element.classList.add('--open')
}

const closeModal = (element) => {
  element.classList.remove('--open')
}

const myModal = () => {
  /**
		modal 생성 코드
	*/
  return document.querySelector('#modal')
}

openModal(myModal)
changeColor(myModal)
closeModal(myModal)
```

- 바닐라 자바스크립트로 구현할 때에는 `HTML`, `CSS`, `Javascript`가 각기 다르기 때문에 역할과 책임에 따라 함수를 추상화하고, 재활용하는 코드를 작성해야 함

> **Referenced**

- [클린코드 자바스크립트](https://www.udemy.com/course/clean-code-js/)
