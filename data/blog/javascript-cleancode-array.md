---
title: 배열 다루기
date: '2023-04-23'
tags: ['javascript']
draft: false
summary: Javascript의 배열은 객체다, Array.length, 배열 요소에 접근하기, 유사 배열 객체, 불변성, for 문 배열 고차 함수로 리팩터링, 배열 메서드 체이닝 활용하기
layout: PostSimple
authors: ['default']
---

# 배열 다루기

## Javascript의 배열은 객체다

### Case 1. 배열의 값 삽입

```jsx
const arr = [1, 2, 3]

arr[3] = 'test'
arr['property'] = 'string value'
arr['obj'] = {}
arr[{}] = [1, 2, 3]
arr['func'] = function () {
  return 'hello'
}

/* 반환값
[ 1,
  2,
  3,
  'test',
  property: 'string value',
  obj: {},
  '[object Object]': [ 1, 2, 3 ],
  func: [λ] ]
*/
```

- 배열임을 예상하고 배열의 키에 값을 할당했음에도 불구하고, 반환값이 객체처럼 key와 value의 한쌍으로 출력됨
- 자바스크립트의 배열 내부 동작은 객체와 유사하게 동작함

### Case 2. 배열의 값 타입 확인하기

```jsx
const arr = [1, 2, 3]
// const arr = '[1, 2, 3]';

console.log(Array.isArray(arr))
/*
if (arr.length) {
	console.log('배열 확인');
}

if (arr instanceof Array) {
	console.log('배열 확인');
}
*/
```

- 문자열에도 `length` 프로퍼티가 있으므로, `Array.isArray`와 같은 내장 메서드 객체를 사용해 배열검증을 해야 함

> 자바스크립트의 배열은 객체와 같이 동작할 수 있으므로, 주의가 필요함

## Array.length

### Case 1. 구멍 배열

- `React`에서도 흔히 쓰이는 `length` 프로퍼티는 배열의 길이를 주로 측정하고 `&&` 연산자로 해당 조건이 참일 때, 컴포넌트를 반환하는 식의 코드를 무심코 작성하는 경우가 있음

```jsx
const arr = [1, 2, 3]

console.log(arr.length) // 3

arr.length = 10

console.log(arr.length) // 10

// arr -> [ 1, 2, 3, , , , , , ,  ]
```

- `arr.length`에 무심코 값을 할당하면 할당될 자리에 구멍처럼 빈값이 할당되는 경우가 있음
- 자바스크립트 `Array.length`는 배열의 길이보다는 마지막 인덱스를 추출하는 경우 사용하는 것을 권장
  - 배열의 길이 보장 X

### Case 2. 프로토타입 조작

```jsx
Array.prototype.clear = function () {
  this.length = 0
}

function clearArray(array) {
  array.length = 0

  return array
}

const arr = [1, 2, 3]

console.log(clearArray(arr))
```

- 프로토타입을 재정의해서 배열의 길이를 `0`으로 바꾸면 원본 배열에 영향이 감
- 이처럼 외부의 요인에 의해 값이 조작될 위험이 있기 때문에 위와 같은 위험성을 인지하고 유의해서 작성해야 함

## 배열 요소에 접근하기

### Case 1. 배열의 특정 인덱스 접근

```jsx
function operateTime(inputs, operators, is) {
  inputs[0].split('').forEach((num) => {
    cy.get('.digit').contains(num).click()
  })

  inputs[1].split('').forEach((num) => {
    cy.get('.digit').contains(num).click()
  })
}
```

- cypress testing 도구에서 inputs 배열의 특정 숫자로 배열 요소의 n번째에 접근하는 소스코드를 보면 해당 숫자가 무슨 의미인지 알기 어려운 경우가 있음

```jsx
function operateTime(inputs, operators, is) {
  const [firstInput, secondInput] = inputs

  firstInput.split('').forEach((num) => {
    cy.get('.digit').contains(num).click()
  })

  secondInput.split('').forEach((num) => {
    cy.get('.digit').contains(num).click()
  })
}
```

```jsx
function operateTime([firstInput, secondInput], operators, is) {
  firstInput.split('').forEach((num) => {
    cy.get('.digit').contains(num).click()
  })

  secondInput.split('').forEach((num) => {
    cy.get('.digit').contains(num).click()
  })
}
```

- 배열을 구조분해 할당해서 의미있는 변수로 명시하여 의미를 분명하게 함
- 혹은 매개변수에서 구조 분해 할당으로 값을 취득하는 방법을 권장

### Case 2. 구조 분해 할당 활용

```jsx
function formatDate(targetDate) {
  const date = targetDate.toISOString().split('T')[0]
  const [year, month, day] = date.split('-')

  return `${year}년 ${month}월 ${day}일`
}
```

- `data`에 할당될 배열의 첫번째 요소는 위의 예시처럼 0의 인덱스 값을 받아 첫번째 요소를 할당함
- 해당 예시 역시 0을 삭제하기 위해서 아래와 같은 방법으로 진행할 수 있음

  - `const date = targetDate.toISOString().split('T')[0];`

    - 하나의 요소인 경우에도 구조 분해 할당이 가능함
      - `const [date] = targetDate.toISOString().split('T');`
    - 유틸 함수를 만들어서 처리

      ```jsx
      function head(arr) {
        return arr[0] ?? ''
      }

      function formatDate(targetDate) {
        const date = head(targetDate.toISOString().split('T'))
        const [year, month, day] = date.split('-')

        return `${year}년 ${month}월 ${day}일`
      }
      ```

      - `head`라는 유틸함수르 전달받은 매개변수의 첫번째 요소(`0`)이 없으면 빈문자열을 반환하는 유틸함수를 만들고 `formatDate` 함수에 인자를 전달

## 유사 배열 객체

### Case 1. Array.from 활용

```jsx
const arrayLikeObject = {
  0: 'HEllO',
  1: 'WORLD',
  length: 2,
}

const arr = Array.from(arrayLikeObject)

console.log(Array.isArray(arrayLikeObject)) // false

console.log(Array.isArray(arr)) // true

console.log(arr) // ['HELLO', 'WORLD']
```

- `Array.from`에 객체를 전달하면 `array`와 유사한 배열 객체를 생성할 수 있음
- `isArray`
  - 배열인지 아닌지 `Boolean`으로 반환하는 메서드
- `Web API` 중 `nodeList`도 요소를 배열로 반환하기 때문에 Array.from과 유사하게 동작

### Case 2. 유사 배열 객체(arguments)

```jsx
function generatePriceList() {
  console.log(Array.isArray(arguments)) // false

  for (let index = 0; index < arguments.length; index++) {
    const element = arguments[index]

    console.log(element) // 100, 200, 300, 400, 500, 600
  }

  // return Array.from(arguments).map((arg) => arg + "원");
}

generatePriceList(100, 200, 300, 400, 500, 600)
```

- `generatePriceList`의 함수의 전달된 매개변수가 없음에도 불구하고, `arguments`가 마치 배열처럼 동작하고 있음
- `arguments`는 유사 배열 객체로써, 순회가 가능함
  - 하지만, 자바스크립트의 내장 함수 중에 `Map`, `forEach`, `reduce`, `filter`, `every` 등과 같은 고차 함수에서는 사용이 불가능함
  - 유사 배열 객체로써 동작하므로 `Array.from`을 사용하면 고차 함수를 사용할 수 있음
- `proto` 를 확인해보면 map과 같은 내장 메서드가 존재하지 않음

## 불변성

```jsx
const originArray = ['123', '456', '789']

const newAray = [...originArray]

originArray.push(10)
originArray.push(11)
originArray.push(12)
originArray.unshift(0)
ㅇ
```

- 말그대로, 변하지 말하야 하는 값을 의미
  - 원본 배열은 변하지 않고 새로운 배열을 변환한다는 의미에서 흔히 사용되는 규칙
- 불변성을 지키기 위해서는 배열을 복사하고, 새로운 배열을 반환하는 메서드들을 활용해야 함
  - 커스텀 함수 활용
  - 반환 값이 새로운 배열을 반환하는 고차 배열 함수 사용(`map`, `filter`, `slice` …)

## for 문 배열 고차 함수로 리팩터링

### Case 1. 원화 표기

```jsx
/** Before - for문
 * 1. 원화 표기
 */
const price = ['2000', '1000', '3000', '5000', '4000']

function getWonPrice(priceList) {
  let temp = []

  for (let i = 0; i < priceList.length; i++) {
    temp.push(priceList[i] + '원')
  }

  return temp
}
```

```jsx
// After - map 고차 함수 사용
const price = ['2000', '1000', '3000', '5000', '4000']
const suffixWon = (price) => price + '원'

function getWonPrice(priceList) {
  return priceList.map(suffixWon)
}

const result = getWonPrice(price)
```

- for문에서 불필요하게 사용되는 임시 변수와 i와 같은 의미 없는 변수값을 map을 사용해 위와 같이 개선

### Case 2. 원화 표기 조건식 추가

```jsx
/** Before - for문
 * 1. 원화 표기
 * 2. 1000원 초과 리스트만 출력
 */

const price = ['2000', '1000', '3000', '5000', '4000']

function getWonPrice(priceList) {
  let temp = []

  for (let i = 0; i < priceList.length; i++) {
    if (priceList[i] > 1000) {
      temp.push(priceList[i] + '원')
    }
  }

  return temp
}
```

```jsx
// After - filter 고차 함수 사용
const price = ['2000', '1000', '3000', '5000', '4000']

const suffixWon = (price) => price + '원'
const isOverOneThousand = (price) => Number(price) > 1000

function getWonPrice(priceList) {
  const isOverList = priceList.filter(isOverOneThousand)

  return isOverList.map(suffixWon)
}

const result = getWonPrice(price)
```

- `if` 조건문을 `filter` 메서드를 활용해 인자로 받은 `price`값을 숫자형태로 캐스팅한 후 조건에 부합하는 새로운 배열로 반환한 후, 해당 배열을 `map`으로 순회하여 개선

### Case 3. 가격 순 정렬 추가

```jsx
/** Before - for문
 * 1. 원화 표기
 * 2. 1000원 초과 리스트만 출력
 * 3. 가격 순 정렬
 */

const price = ['2000', '1000', '3000', '5000', '4000']

function getWonPrice(priceList, orderType) {
  let temp = []

  for (let i = 0; i < priceList.length; i++) {
    if (priceList[i] > 1000) {
      temp.push(priceList[i] + '원')
    }
  }

  if (orderType === 'ASCENDING') {
    someAscendingSortFunc(temp)
  }

  if (orderType === 'DESCENDING') {
    someDescendingSortFunc(temp)
  }

  return temp
}
```

```jsx
// After - accendingList 커스텀 함수 사용
const price = ['2000', '1000', '3000', '5000', '4000']

const suffixWon = (price) => price + '원'
const isOverOneThousand = (price) => Number(price) > 1000
const ascendingList = (a, b) => a - b

function getWonPrice(priceList) {
  const isOverList = priceList.filter(isOverOneThousand)
  const sortList = isOverList.sort(ascendingList)

  return sortList.map(suffixWon)
}

const result = getWonPrice(price)
```

- `for` 문의 늘어난 조건문으로 인해서, 결과값을 예측하기 어려울 뿐만아니라, 유연한 대처를 할 수 없음
- 개선을 한 코드 마찬가지로, 늘어난 정렬 조건 만큼 새로 생성해야할 함수와 변수는 근본적인 문제를 해결할 수 없으므로 아래 메서드 체이닝을 활용해야 함

## 배열 메서드 체이닝 활용하기

### Before

```jsx
const price = ['2000', '1000', '3000', '5000', '4000']

function getWonPrice(priceList, orderType) {
  let temp = []

  for (let i = 0; i < priceList.length; i++) {
    if (priceList[i] > 1000) {
      temp.push(priceList[i] + '원')
    }
  }

  if (orderType === 'ASCENDING') {
    someAscendingSortFunc(temp)
  }

  if (orderType === 'DESCENDING') {
    someDescendingSortFunc(temp)
  }

  return temp
}
```

### After

```jsx
const price = ['2000', '1000', '3000', '5000', '4000']

const suffixWon = (price) => price + '원'
const isOverOneThousand = (price) => Number(price) > 1000
const ascendingList = (a, b) => a - b

function getWonPrice(priceList) {
  return priceList
    .filter(isOverOneThousand) // filte 원하는 조건에 맞는 배열 리스트 만듦
    .sort(ascendingList) // sort 정렬
    .map(suffixWon) // 배열 요소들을 다시 정리
}

const result = getWonPrice(price)
```

- 각 배열의 요소들을 `filter`, `sort`, `map` 함수 순서대로 체이닝하여 반환
- 자바스크립트에서 제공하는 내장 메서드를 통해 코드와 의미를 보다 명확하게 구현할 수 있음

> **Referenced**

- [클린코드 자바스크립트](https://www.udemy.com/course/clean-code-js/)
