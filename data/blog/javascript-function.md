---
title: 함수
date: '2022-05-13'
tags: ['javascript']
draft: false
summary: 함수는 코드의 재사용이라는 측면에서 유용하며 `유지보수의 편의성`을 높이고, 실수(human error)를 줄여 코드의 신뢰성과 가독성을 향상시키는데 목적이 있음
layout: PostSimple
authors: ['default']
---

# 함수

> 함수는 코드의 재사용이라는 측면에서 유용하며 `유지보수의 편의성`을 높이고, 실수(human error)를 줄여 `코드의 신뢰성과 가독성을 향상`시키는데 목적이 있음

## 함수의 리터럴의 구분

`함수 이름`
- 함수 이름은 식별자이므로 식별자 네이밍 규칙을 준수해야함
- 함수 이름은 함수 몸체 내에서만 참조할 수 있음 
- 함수 이름은 함수 몸체 내에서만 참조할 수 있음 
- 함수 이름은 생략할 수 있으며, 이름 유무에 따라 기명함수, 익명함수로 나뉨

`매개 변수 목록`
- 0개 이상의 매개변수를 소괄호(`(..)`)로 감싸고 쉼표로 구분 
- 각 매개변수에는 함수를 호출할 때 지정한 인수가 순서대로 할당 
- 함수 몸체 내에서 변수와 동일하게 취급(식별자 네이밍 규칙 준수)

`함수 몸체`
- 함수가 호출될때 일괄적으로 실행될 문(실행 단위의 코드블록) 
- 함수 몸체는 함수 호출에 의해 실행

### 리터럴

- 사람이 이해할 수 있는 문자 또는 약속된 기호로 사용해 값을 생성하는 표기 방식
- 값을 생성하기 위한 표기법

### 함수 객체와 일반 객체의 차이

- 일반 객체는 호출할 수 없지만 함수는 호출 가능
- 함수 객체에는 고유한 프로퍼티를 가지고 있고, `함수가 객체`라는 사실은 타 프로그래밍 언어와 구별되는 중요한 특징

## 함수 정의표

| 함수 정의 방식        | 예시                                                |
|-----------------|---------------------------------------------------|
| 함수 선언문          | `function add(x, y){return x + y;}`               |
| 함수 표현식          | `var add = function(x, y){return x + y;}`         |
| Function 생성자 함수 | `var add = new Function('x', 'y', 'return + y');` |
| 화살표 함수(ES6)     | `var add = (x, y) ⇒ x + y;`                       |

### 함수 선언문

```javascript
// 함수 선언문
function add(x, y) {
  return
  x + y;
}

// 함수 참조
console.dir(add); // f add(x,y)

// 함수 호출
console.log(add(2, 5)); // 7
```

- 함수 선언문은 이름(add)을 생략할 수 없음
- 표현식이 아닌 `문에는 변수에 할당할 수 없음`

`기명 함수 리터럴`

```javascript
var add = function add(x, y) {
  return x + y;
};

// 함수 호출
console.log(add(2, 5)); // 7
```

- 표현식이 아닌 문은 변수에 할당되지 않는데, 해당 예시는 변수에 할당되고 있음
  - 자바스크립트 엔진이 코드의 문맥에 따라 다르게 해석함
    - `기명 함수 리터럴`도 중의적인 코드이므로 코드의 문맥에 따라 해석이 달라짐

```javascript
function foo() {
  console.log('foo');
}

foo(); // foo

(function bar() {
  console.log('bar');
})
bar(); // ?
```

- 단독으로 사용된 함수 리터럴(foo)는 `함수 선언문`으로 해석
- 그룹 연산자(`()`)를 사용하면 함수 `리터럴 표현식`으로 해석하므로 `ReferenceError: bar is not defined`를 오류를 냄
- 표현식이 아닌 문인 함수 선언문은 피연산자로 사용할 수 없음(검증)

> 자바스크립트 엔진은 생성된 함수를 호출하기 위해 함수 이름과 `동일한 이름의 식별자를 암묵적으로 생성`

<img alt="js_function1" src="/static/images/js_function1.png" width="500"/>

- 함수는 함수 이름으로 호출하는 것이 아닌 함수 객체를 가리키는 식별자로 호출함

### 함수 표현식

- 값처럼 변수에 할당할 수도 있고 프로퍼티 값이 될 수도 있으며 배열의 요소가 될 수 있음
- 즉, 자바스크립트의 함수는 `일급 객체`(⭐⭐⭐⭐⭐)

```javascript
var add = function (x, y) {
  return x + y;
}

console.log(add(2, 5)); // 7
```

- 함수 선언문으로 정의한 add 함수를 함수 표현식으로 바꿔 정의할 수 있음
- 함수 표현식의 이름을 생략하는 것이 일반적임

### 함수 생성 시점과 함수 호이스팅

```javascript
// 함수 참조
console.dir(add); // f add(x,y)
console.dir(sub); // undefined

// 함수 호출
console.log(add(2, 5)); // 7
console.log(sub(2, 5)); // TypeError: sub is not a function

// 함수 선언문
function add(x, y) {
  return x + y;
}

// 함수 표현식
var sub = function (x, y) {
  return x - y;
}
```

- 함수 선언문으로 정의한 함수와 함수 표현식으로 정의한 함수의 `생성 시점이 다름`
  - `런타임 이전`에 객체가 먼저 생성
    - 함수 이름과 동일한 이름의 식별자를 `암묵적으로 생성`하고 `함수 객체 할당`
- `함수 호이스팅 vs 변수 호이스팅`
  - 함수 호이스팅
    - 함수 선언문이 코드의 선두로 끌어 올려진 것처럼 동작하는 자바스크립트 고유의 특징
  - 변수 호이스팅
    - 함수 호이스팅과 동일하게 런타임 이전에 먼저 실행되어 `undefined로 초기화`함
    - 변수 할당문의 값은 할당문이 실행되는 시점`(런타임)에 평가됨`

> `함수 표현식`으로 함수를 정의하면 함수 호이스팅이 발생하는 것이 아니라, `변수 호이스팅이 발생`

![js_function2](/static/images/js_function2.png)

- 함수 표현식으로 정의한 함수는 반드시 `함수 표현식 이후에 참조 또는 호출 해야함`(⭐⭐⭐⭐⭐)
- 다만, `함수 호이스팅`은 함수를 호출하기 전에 반드시 함수를 선언해야 한다는 `당연한 규칙을 무시`함
  - 그러므로!! 함수 선언문 대신 `함수 표현식 사용을 권장`

### 화살표 함수

- ES6에 도입된 함수로서, function 키워드 대신 `=>` 키워드를 사용해 선언을 단축할 수 있음
- 항상 익명 함수로 정의

```javascript
// 화살표 함수
const add = (x, y) => x + y;
console.log(add(2, 5)) // 7
```

- 생성자 함수로 사용할 수 없으며, 기존 함수와 `this 바인딩 방식이 다름`
- prototype 프로퍼티가 없으므로 `argument 객체를 생성하지 않음`

## 함수 호출

### 매개변수의 최대 개수

- ECMAScript 사양에는 매개변수의 최대 개수에 대해 명시적으로 제한하고 있지 않음
- 하지만, `매개변수가 많아지면` 그만큼 인수의 순서를 고려해야하므로 코드 전체가 영향을 받고 `유지보수성이 나빠짐`
- 이상적인 함수는 `한 가지 일만` 해야 하며 `가급적 작게` 만들어야 함

## 다양한 함수의 형태

### 즉시 실행 함수

- 함수 정의와동시에 즉시 호출되는 함수(`IIFE` - Immediately invoked Function Expression)
- 단 한 번만 호출되고 다시 호출할 수 없음

### 익명 즉시 실행 함수

```javascript
(function () {
  var a = 3;
  var b = 5;
  return a * b;
}());
```

- 함수 이름이 없는 익명 함수를 사용하는 것이 일반적임
- `()` 그룹 연산자 내의 기명 함수는 함수 리터럴로 평가됨
- 즉시 실행 함수는 반드시 그룹 연산자`(...)`로 감싸야 함
  - 함수 리터럴을 먼저 평가해서 함수 객체를 생성하기 위해서임

```javascript
// 일반 함수처럼 값을 반환할 수 있음
var res = (function () {
  var a = 3;
  var b = 5;
  return
  a + b;
}());

console.log(res); // 15

// 즉시 실행 함수에도 일반 함수처럼 인수를 전달할 수 있음
res = (function (a, b) {
  return a * b;
}(3, 5));
```

- 왜 쓰이는가?
  - 혹시 있을 수 있는 변수나 함수 이름의 충돌을 방지할 수 있음

### 콜백 함수

- 함수 매개변수를 통해 다른 함수의 내부로 전달되는 함수

```javascript
// 외부에서 전달받은 f를 n만큼 반복 호출
function repeat(n, f) {
  for (var i = 0; i < n; i++) {
    f(i); // i를 전달하면서 f를 호출
  }
}

var logAll = function (i) {
  console.log(i);
};

// 반복 호출할 함수를 인수로 전달
repeat(5, logAll); // 0 1 2 3 4

var logOdds = function (i) {
  if (i % 2) console.log(i);
};

// 반복 호출할 함수를 인수로 전달
repeat(5, logOdds); // 1 3
```

- repeat 함수는 내부 로직에 의존하지 않고 외부에서 로직의 일부분을 함수로 전달받아 수행
- 매개 변수를 통해 함수의 `외부에서 콜백 함수를 전달받은 함수`를 `고차 함수`라고 함
  - 고차 함수는 콜백 함수의 호출 시점을 결정해서 호출함

> 콜백 함수는 `고차 함수에 의해 호출`되며, 고차 함수는 필요에 따라 `콜백 함수에 인수를 전달`할 수 있음

```javascript
// 익명 함수 리터럴을 콜백 함수로 고차 함수에 전달
repeat(5, function (i) {
  if (i % 2) console.log(i);
}); // 1 3
```

- 익명 함수 리터럴로 정의하면서 곧바로 고차 함수에 전달하는 것이 일반적인 케이스

### 순수 함수와 비순수 함수

`순수 함수`

- 외부 상태에 의존하지 않고 변경하지 않는 함수
  - 함수 내부 상태에만 의존한다 해도 그 내부 상태가 호출될 때마다 변화하는 값이라면 순수 함수가 아님
  - 함수의 외부 상태도 변경하지 않음
- `동일한 인수`가 전달되면 언제나 `동일한 값`을 반환

```javascript
var count = 0;

function increase(n) {
  return ++n;
}

count = increase(count);
console.log(count); // 1
```

- 순수 함수 increase는 동일한 인수가 전달되면 언제나 동일한 값을 반환
- 순수 함수가 반환한 결과값을 변수에 재할당해서 상태를 변경

`비순수 함수`

- 외부 상태에 의존하거나 외부 상태를 변경하는 함수
  - 부수 효과(side effect)가 발생

```javascript
var count = 0;

function increase() {
  return ++count;
}

increase();
// console.log(count); // 1
```

- increase 함수는 외부 상태에 의존하며 외부 상태를 변경함

  > 함수가 외부 상태를 변경하면 상태 변화 추적에 어려우므로 `순수 함수 사용`을 권장

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 154 ~ 188p
