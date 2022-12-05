---
title: 경계 다루기
date: '2022-12-05'
tags: ['javascript']
draft: false
summary: min - max, begin - end, first - last, prefix - suffix, 매개변수의 순서가 경계다
layout: PostSimple
authors: ['default']
---

# 경계 다루기

## min - max

```jsx
function genRandomNumber(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min
}

// 상수
const MIN_NUMBER = 1
const MAX_NUMBER = 45

genRandomNumber(MIN_NUMBER, MAX_NUMBER)
```

- 함수의 구현부를 보지 않아도 예측 가능한 함수명을 정의
- 넘겨주는 인자의 변수명 또한 상수로 선언하여 어떤 매개변수가 전달되는지 예측 가능함
- 팀마다 `MIN`, `MAX`의 경계가 애매해질 수 있음
- 최소값, 최대값 (포함되는지 vs 안되는지)
- 이상, 초과 vs 이하, 미만
- 컨벤션이 모호해줄 수 있어, 최소값과 최대값 포함 여부를 결정하거나 혹은 네이밍에 최소값과 최대값 포함 여부를 포함함
- ex) MIN*`IN`\_NUMBER(이하), MIN_NUMBER*`LIMIT`(초과)

## begin - end

- `DatePicker`와 같은 시작 날짜와 끝 날짜 함수를 정의할 때, 함수명과 매개변수명을 예측가능한 네이밍 컨벤션을 지정

```jsx
/**
 * begin - end
 *
 * 경계를 포함하지만 제외하는 경우
 */

function reservationDate(beginDate, endDate) {
  // ...some code
}

reservationDate('YYYY-MM-DD', 'YYYY-MM-DD')
```

- 매개변수가 왼쪽 → 오른쪽으로 자연스럽게 유추할 수 있음

## first - last

```jsx
/**
 * first - last
 *
 * 포함된 양 끝을 의미한다.
 * 부터 ~~~ 까지
 */

const students = ['jikgoo', '어글리', '직구']

function getStudents(first, last) {
  // ...some code
}

getStudents('jikgoo', '직구')
```

- `first`와 `last` 매개 변수명을 통해, 특정 배열에 시작 - 끝점을 의미하는 네이밍 컨벤션을 지정하고 보다 의미를 유추하기 쉽게 할 수 있음

## prefix - suffix

- 접두사(`prefix`), 접미사(`suffix`)로 사용되는 네이밍 컨벤션은 암묵적은 규칙으로써, 의미를 유추할 수 있음
- `Javascript`의 `getter`, `setter` 예약어는 `prefix`로 쓰임
- `React`의 `hook`은 주로 `useState`, `useEffect` 등으로 사용되며 `prefix`로 사용됨
- `redux`의 상태값을 나타내는 네트워크의 `aciton` 상태 값은 `suffix`로 쓰임
- `export const STARRED_REQUEST = 'STARRED_REQUEST'`
- `export const STARRED_SUCCESS = 'STARRED_SUCCESS'`
- `export const STARRED_FAILURE = 'STARRED_FAILURE'`

## 매개변수의 순서가 경계다

```jsx
/**
* 매개변수의 순서가 경계다
*
* 호출하는 함수의 네이밍과 인자의 순서의 연관성을 고려한다.
*/

// ES2015+
function someFunc(someArg1, someArg2, someArg3, someArg4) {}

function getFunc(someArg1, someArg3) {
  someFunc(someArg1, undefined, someArg3);
}
```

- 함수를 호출할때, 네이밍과 인자의 순서 연관성을 고려해야 함
- 유지보수에 취약함 함수를 만들지 않기 위해서 아래 4가지가 지켜져야 함
  1. 매개변수를 2개가 넘지 않도록 만든다.
  2. 규칙적이지 않은 매개변수가 전달된다면 `arguments`, `rest parameter(...)`사용
  3. 매개변수를 객체에 담아서 넘긴다.
  4. 랩핑하는 함수

> **Referenced**

- [클린코드 자바스크립트](https://www.udemy.com/course/clean-code-js/)
````
