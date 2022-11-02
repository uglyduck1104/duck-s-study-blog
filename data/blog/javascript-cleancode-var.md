---
title: 변수 다루기
date: '2022-11-02'
tags: ['javascript']
draft: false
summary: var를 지양하자, function scope & block scope, 전역 공간 사용 최소화, 임시변수 제거하기
layout: PostSimple
authors: ['default']
---

# 변수 다루기

## var를 지양하자

- `let`, `const`는 ECMA Script 2015 버전부터 생김
  - 이전에는 `var` 예약어로만 변수를 선언함

### 스코프의 구분

- `var` → 함수 스코프
- `let` & `const` → 블록 단위 스코프
  - `Temporal Dead Zone(TDZ)` 속성을 가지므로 보다 안전한 코드를 작성할 수 있음

### 중복 선언, 중복 할당 - var

```javascript
var name = '이름'
var name = '이름2'

console.log(name) // 이름2
```

- 동일한 이름으로 변수에 값을 할당함
  - 값은 다른데 변수명은 일치함에도 불구하고 선언과 할당을 허용함
  - 가장 마지막에 할당된 값이 변수의 값

```javascript
console.log(name) // undefined

var name = '이름3'
```

- 오류가 발생해야하는 상황임에도 오류가 발생하지 않고 중복 할당, 중복 선언을 허용함
  - 늘어난 코드 라인에 비해, 할당된 값을 추적하기 어려워 혼란이 야기됨

### 중복 선언, 중복 할당 - const & let

```javascript
let name = '이름3'
let name = '이름3'
let name = '이름3'

// SyntaxError: Identifier 'name' has already been declared
```

- `let` 예약어로 이미 선언한 변수명은 런타임 에러를 출력함

```javascript
let name

name = '이름'
name = '이름2'
name = '이름3'

console.log(name) // 이름3

const anotherName = '이름'
anotherName = '이름2'

// Assignment to constant variable.
```

- `const` 예약어로 값을 할당하면 재할당이 불가능함
  - 조금 더 안전하게 변수를 선언하고 할당할 수 있음

## function scope & block scope

### Function scope - var

```javascript
var global = '전역'

if (global === '전역') {
  var global = '지역'

  console.log(global) // 지역
}

console.log(global) // 지역
```

- `if` 블록에서 할당한 `지역` 문자열 값이 전역 스코프까지 오염됨
- `var`는 함수 단위 스코프이기 때문에, 블럭 단위 스코프로 바꾸지 않는 이상 오염의 위험성이 있음

### Block scope - let

```javascript
let global = '전역'

if (global === '전역') {
  let global = '지역'

  console.log(global) // 지역
}

console.log(global) // 전역
```

- 중괄호(`블럭 단위`) 안에서는 전역 변수 값이 오염이 되지 않고, 명확한 값을 보여주고 있음
- 굳이 `if`문을 작성하지 않아도 블럭 단위(`{}`)로 코드를 작성해도 동일한 결과를 얻을 수 있음

### Block scope - const

```javascript
const person = {
  name: 'jang',
  age: '30',
}

/*
person = {
	name: 'jang',
	age: '30',
}
*/

person.name = 'lee'
person.age = '31'

console.log(person) // { name: 'lee', age: '22' }

const person2 = [
  {
    name: 'jang',
    age: '30',
  },
]

person2.push({
  name: 'lee',
  age: '31',
})

console.log(person2) // [ { name: 'jang', age: '30'}, { name: 'lee', age: '31' } ]
```

- 선언과 동시에 할당한 `person` 객체는 확실히 재할당이 불가능하지만 점 표기법으로 속성값을 직접 변경할 수 있음
  - `person`을 재할당하지 않고 객체 내부의 값만 바꾸었으므로 가능함
- 배열도 객체와 마찬가지로 원본 배열에 값을 추가할 수 있음

> 재할당만 금지되고, 객체, 배열 같은 레퍼런스 객체들을 조작할 때는 이상이 없음

## 전역 공간 사용 최소화

- 전역(`최상위`) 공간의 구분
  - `Global`이라고도 하고 `Window`로 나뉨
  - 브라우저 환경 → `Window`
    - 브라우저에서 간단하게 사용할 수 있는 인터페이스
  - NodeJS → `Global`
    - NodeJS 환경이라서 브라우저에서는 사용하지 못함
- `var` 예약어를 통해 전역 변수로 선언한 값은 window 객체에서 접근이 가능하므로, 파일을 나누어도 코드의 스코프은 공유됨
  - 예약어를 변수로 선언했을 때, 기존의 예약어는 동작하지 않고 에러 조차 볼 수 없음

> `IIFE`, `Module`, `Closure`, `const`, `let`과 같은 개념으로 전역 공간 사용을 최소화 하는 편이 나음

## 임시변수 제거하기

- 임시변수 → 특정 공간 `Scope` 안에서 전역 변수처럼 동작
- 함수를 최대한 쪼개서 값을 반환해야 함

### 예시 1

```javascript
function getElements() {
  const result = {} // 임시 객체

  result.title = document.querySelector('.title')
  result.text = document.querySelector('.text')
  result.value = document.querySelector('.value')

  return result
}
```

- 블록 스코프에서 사용한 임시 변수는 전역 공간이나 다름 없는 상황이 나올 수 있음
- 임시 객체가 생기는 순간 객체에 계속 접근할 수 있음
- 아래와 같이 개선할 수 있음

  ```javascript
  function getElements() {
  	return {
  		title: document.querySelector('.title');
  		text: document.querySelector('.text');
  		value: document.querySelector('.value');
  	};
  }
  ```

  - 임시 변수를 선언할 필요 없이, 자체적으로 값을 반환

### 예시 2

```javascript
function geDateTime(targetDate) {
  let month = targetDate.getMonth()
  let day = targetDate.getDate()
  let hour = targetDate.Hours()

  month = month >= 10 ? month : '0' + month
  day = day >= 10 ? day : '0' + day
  hour = hour >= 10 ? hour : '0' + hour

  return {
    month,
    day,
    hour,
  }
}
```

- `getDateTime` 함수에서 추가적인 스펙이 생길 경우, 요구 사항을 위해서 함수를 추가하느냐 기존 함수를 유지 보수하는 지의 관점

  - 유지보수의 경우 함수를 제대로 보수하지 못하면 많은 부작용이 발생하므로 바로 값을 리턴하는 형태로 개선

    ```javascript
    function getDateTime(targetDate) {
    	const month = targetDate.getMonth();
    	const day = targetDate.getDate();
    	const hour = targetDate.Hours();

    	return {
    		month: month >= 10 ? month : '0' + month;
    		day: day >= 10 ? day : '0' + day;
    		hour: hour >= 10 ? hour : '0' + hour;
    	};
    }

    function getDateTime() {
    	const currentDateTime =	getDateTime(new Date());

    	return {
    		month: currentDateTime.month + '분 전'
    		day: currentDateTime.day + '일 전'
    		hour: currentDateTime.hour + '시 전';
    	}
    }
    ```

  - 함수를 한 번 더 추상화해 재활용 할 수 있고, 함수를 명확한 역할로 구분할 수 있음

> **Referenced**

- [클린코드 자바스크립트](https://www.udemy.com/course/clean-code-js/)
