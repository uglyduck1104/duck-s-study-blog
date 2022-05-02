---
title: 인간 JS 엔진 되기 - Part1
date: '2022-05-03'
tags: ['javascript']
draft: false
summary: 고차함수, 호출 스택 분석, 스코프체인, 호이스팅, this
layout: PostSimple
authors: ['default']
---

## 함수와 함수의 호출, 고차함수

`hello.js`

```jsx
const add = (a, b) => a + b

function calculator(func, a, b) {
  return func(a, b)
}

add(3, 5) // 8
calculator(add(), 3, 5) // 8

document.querySelect('#header').addEventListener('click', add())
// 함수 호출의 잘못된 예
// => 함수의 리턴 값을 생각하자!!

const onClick = () => () => {
  console.log('hello')
}
/*
const onClick = () => {
	return () => {console.log('hello')};
}
*/

document.querySelect('#header').addEventListener('click', onClick())
/*
document.querySelect('#header').addEventListener('click', () => {
	console.log('hello');
});
*/

// 해당문은 정상적으로 동작함
// 고차함수의 경우도 리턴값을 생각해서 치환해보자
```

`react.js`

```jsx
import { useCallback } from 'react';

export const App = () => {
	const onClick = useCallback((e) => {
		console.log(e.target);
	}, []);

	return (
		<div onClick={onClick()}></div> // 잘못된 호출
		<div onClick={onClick}></div> // 정상 호출
	)
}
```

- 함수의 호출 값이 헷갈리면, 리턴 값을 치환해서 직접 확인해보자
- 함수 호출인지, 선언인지 정확히 구분해서 사용

## 호출 스택 분석

### 개요

- 실행하지 않고 결과를 추측할 수 있어야 함
- 일반적으로 위에서 아래로 실행, 왼쪽에서 오른쪽으로
  - 1차원적인 사고 방식에서 벗어나야 함
  - 직접 그리는 연습이 필요함
    - stack - FILO, LIFO
    - queue - FIFO
  - 특정 선언문 혹은 함수에 debugger 걸고 개발자 도구에 Call stack 확인하기

### 호출 스택 분석

`callstack.js`

```jsx
// 변수 선언
const x = 'x'

// 함수 선언
function c() {
  const y = 'y'
  console.log('c')
}

function a() {
  const x = 'x'
  console.log('a')
  function b() {
    const z = 'z'
    console.log('b')
    c()
  }
  b()
}

a() // a, b, c
c() // c
```

- 함수가 호출되면, 선언부가 실행
  - 함수의 중괄호 부터 중괄호까지
  - 함수의 호출도 stack에 쌓임
  - 중괄호가 끝나면 stack에서 빠져나감

## 스코프체인

### 개요

```jsx
// 자바스크립트 내장 함수
const console = {
  log() {
    // 콘솔에 글자 적는 기능
  },
}
```

- 해당 코드처럼 자바스크립트에서 제공하는 라이브러리, 내장 함수가 돌아가는 것을 상상해보기

`callstack.js`

```jsx
const x = 'x'
function c() {
  const y = 'y'
  console.log('c')
}

function a() {
  const x = 'x'
  console.log('a')
  function b() {
    const z = 'z'
    console.log('b')
    c()
  }
  b()
}

a() // a, b, c
c() // c
```

- 스코프체인
  - 함수에서 어떤 값에 접근이 가능 / 불가능한가 확인
  - 함수의 선언부만 보고, 내 부모의 함수가 무엇인지 파악하기
  - lexical scope로 접근해서 분석하기
  - 선언 지도 그려보기
- ES6로 넘어가면서 함수의 블록의 개념 중요

## 호이스팅

### 개요

- 함수 선언 부분을 최상단에 써서 호이스팅이 발생하는 오류를 만들지마라
- eslint 설정을 통해 발생되는 코드를 방지해라

`callstack.js`

```jsx
var y

function c() {
  const y = 'y'
  console.log('c')
  function b() {
    const z = 'z'
    console.log('b')
    c()
  }
}

function a() {
  console.log('a')
  b()
  console.log(x) // TDZ temporal dead zone
  const x = 'x2'
}

const x = 'x'
console.log(z) // Error
y = 'hehe'
const z = () => {}

a() // a, b, c
c() // c
```

- variable keyword는 호이스팅, 변수 재 할당 등의 문제로 쓰지 않을 것
  - 함수의 호이스팅을 방지하기 위해서 선언부는 최상단에 둠
- const, let 선언보다 호출을 상단에 올리지 말 것

```jsx
function a() {
  console.log(z)
}

// 정상 동작
const z = 'z1'
a()

// 다만 순서를 바꾸면 에러 발생
a()
const z = 'z1'
```

- TDZ는 함수 내부에까지 영향을 끼치지 않음

## this

### 개요

- 요즘에는 `globalThis`으로 통합
  - 자바스크립트에서는 window 반환
  - Node.js에서는 global 반환

```jsx
function a() {
  'use strict'
  console.log(this)
}
```

- strict 모드일 경우에는 `undefined` 반환
  - ES2015 모듈에서는 strict 모드 자동 적용

```jsx
const obj = {
	name: 'he';
	sayName() { // TIP: sayName: function() { 과 동작 방식이 다름
		console.log(this.name);
	}
}
```

- this는 함수가 호출될 때 결정됨
  - 함수를 호출하지 않을 경우
    - Window 객체 반환
  - 함수를 호출할 경우
    - 함수 앞에 객체가 붙을 경우 객체의 속성이 호출됨

### 공식

```jsx
function Human(name) {
  this.name = name
}

// new Human('person')
// > Human {name: 'person'}
//	     name:'person'
```

this가 바뀌는 경우

1. new 키워드를 이용해 객체를 생성한 경우
2. 함수 앞에 객체가 붙은 경우(`obj.sayName()`)

- Arrow function으로 호출했을 때는 예외

### this 바꾸기

```jsx
function sayName() {
  console.log(this.name)
}
```

- window 객체의 name을 반환
- apply, bind, call로 함수의 this를 바꿀 수 있음
  - bind
    - `sayname.bind({ name: 'person'})()`
      - 새로운 함수를 반환 후 함수를 직접 호출해야함
      - 반환함과 동시에 호출하려면 `()` 삽입해야 함
  - apply
    - `sayname.apply(null, [3,5])`
    - 첫번째 인자는 this argument 임
    - 매개변수를 `배열`로 넘김
  - call
    - `sayname.call(null, 3, 5)`
    - 첫번째 인자는 this argument 임
    - 매개변수를 `순서대로` 넣어줌

### 예제로 알아보는 코드

```jsx
const obj = {
  name: 'person',
  sayName() {
    console.log(this.name)
    function inner() {
      console.log(this.name)
    }
    inner()
  },
}
```

- inner의 lexical scope 체인
  - innner → sayName → Anonymous(root)
- `obj.sayName()`은 `person`을 가리키지만, `inner()의` 함수 호출은 `window.name`을 가리킴
- 함수 호출을 할때 this를 바꿔줬는지 안했는지 공식처럼 생각

```jsx
const obj = {
  name: 'person',
  sayName() {
    console.log(this.name)
    const inner = () => {
      console.log(this.name)
    }
    inner()
  },
}
```

- `Arrow function () ⇒`
  - 부모의 함수의 this를 따라감
- this 다음과 같이 호출될 때 결정됨

  ```jsx
  > obj.sayName()
  person
  person
  ```

  - 호출되기전에는 알 수 없음

### this를 분석할 수 없는 케이스

```jsx
const header = document.querySelector('.El')
header.addEventListener('click', function () {
  console.log(this) // header
})
```

- addEventListener내의 `function()`은 선언이지 호출이 아님
  - 호출하는 부분이 보이지 않음

### 추측 코드(정확하지 않음)

```jsx
const header = {
  addEventListener: function (eventName, callback) {
    callback.call(this) // this가 header
    // callback.call(header);
    // callback();
  },
}
```

- addEventListener 함수가 어떻게 구성되어있는지는 알 수 없으나, 동작 방식을 추측하는 연습을 통해 this가 어떻게 동작하는지 깊게 이해해 볼 수 있음

### Arrow function을 이용할 경우

```jsx
const header = document.querySelector('.El')
header.addEventListener('click', () => {
  console.log(this) // header
})
```

- arrow function의 경우 최상위 부모의 window를 가리킴

> **Referenced**

- [Zerocho - 인간 JS 엔진 되기](https://www.youtube.com/channel/UCp-vBtwvBmDiGqjvLjChaJw)
