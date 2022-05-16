---
title: let, const 키워드와 블록 레벨 스코프
date: '2022-05-16'
tags: ['javascript']
draft: false
summary: var 키워드로 선언한 변수의 문제점, let, const 키워드와 비교 
layout: PostSimple
authors: ['default']
---

# let, const 키워드와 블록 레벨 스코프

## var 키워드로 선언한 변수의 문제점

### 변수의 중복 선언 허용

- 같은 스코프 내에서 중복 선언을 허용하여 `먼저 선언된 변수 값이 변경`되는 부작용 발생

### 함수 레벨 스코프

- 함수의 코드 블록만을 지역 스코프로 인정하기 때문에 함수 외부에서 var 키워드로 선언한 변수는 코드 블록 내에서 선언해도 `모두 전역 변수`가됨
- 의도치 않은 `전역 변수의 중복 선언 문제`가 발생

### 변수 호이스팅

- 변수 호이스팅에 의해 변수 선언문이 스코프의 선두로 끌어 올려진 것처럼 동작함
- 변수 선언문 이전에 참조할 수 있음

```javascript
// 변수 호이스팅에 의해 foo 변수 선언(1. 선언)
// 변수 foo -> undefined로 초기화(2. 초기화)
console.log(foo); // undefined

// 변수에 값 할당(3. 할당)
foo = 123;

console.log(foo); // 123

// 변수 선언은 런타임 이전에 자바스크립트 엔진에 의해 암묵적으로 실행
var foo;
```

> 변수 호이스팅에 의해 에러는 발생하지 않지만, 가도성을 떨어뜨리고 잠재적인 오류를 발생시킬 여지를 남김

## let 키워드

- ES6에 새로운 변수 선언 키워드가 도입됨(let, const)

### 변수 중복 선언 금지

- var 키워드와는 달리 let 키워드로 이름이 같은 변수를 중복 선언하면 문법 에러가 발생함

```javascript
// var 키워드 -> 중복 선언 허용
var foo = 123;
var foo = 456;

// let 키워드 -> 중복 선언 금지
let bar = 123;
let bar = 456; // SyntaxError: Identifier 'bar' has already been declared
```

### 블록 레벨 스코프

- var 키워드로 선언한 변수는 함수 레벨 스코프를 따르므로 발생하는 문제점을 let 키워드의 `블록 레벨 스코프`로 해결할 수 있음

```javascript
let foo = 1; // 전역 변수
{
	let foo = 2; // 지역 변수
	let bar = 3; // 지역 변수
}

console.log(foo); // 1
console.log(bar); // ReferenceError: bar is not defined
```

- 전역에서 선언된 foo 변수와 코드 블록 내에서 선언된 foo 변수는 다른 별개의 변수임
- bar 변수도 블록 레벨 스코프를 갖는 지역 변수이므로 전역에서는 bar 변수를 참조할 수 없음

<img alt="js_block_level1" src="/static/images/js_block_level1.png" width="700"/>

### 변수 호이스팅(`var` vs `let`)

`var` keyword

```javascript
// var 키워드로 선언한 변수는 런타임 이전에 선언 단계와 초기화 단계가 실행됨
// 변수 선언문 이전에 변수를 참조할 수 있음
console.log(foo); // undefined

var foo;
console.log(foo); // undefined

foo = 1; // 할당문에서 할당 단계가 실행됨
console.log(foo); // 1
```

<img alt="js_block_level2" src="/static/images/js_block_level2.png" width="500"/>

1. 선언 단계에서 스코프에 변수 식별자를 등록해 자바스크립트 엔진에 변수의 존재를 알림
2. 즉시 초기화 단계에서 undefined로 변수를 초기화

> 변수 선언문 이전에 접근해도 스코프에 변수가 존재하기 때문에 에러는 발생하지 않지만 undefined를 반환

`let` keyword

```javascript
// 런타임 이전에 선언 단계 실행, 아직 변수 초기화 X
// 초기화 이전의 일시적 사각지대에서는 변수 참조할 수 없음
console.log(foo); // ReferenceError: foo is not defined

let foo; // 변수 선언문에서 초기화 단계 실행
console.log(foo); // undefined

foo = 1; // 할당문에서 할당 단계가 실행됨
console.log(foo); // 1
```

<img alt="js_block_level3" src="/static/images/js_block_level3.png" width="500"/>

- `선언 단계`와 `초기화 단계`가 분리되어 진행
- 스코프의 시작 지점부터 초기화 단계 시작 지점까지 변수를 참조할 수 없음(`일시적 사각지대 - TDZ`)
- let 키워드로 선언했다고해서 변수 호이스팅이 발생하지 않는 것은 아님

> ES6에서 도입된 let, const를 포함해서 모든 선언(var, let, const, function, function*, class 등)을 호이스팅함
단, ES6에서 도입된 `let`, `const`, `class`를 사용한 선언문은 `호이스팅이 발생하지 않는 것처럼` 동작

## const 키워드

- 주로 상수를 선언하기 위해 사용하는 키워드로서 let 키워드와 대부분 동일함

### 선언과 초기화

- const 키워드로 선언한 변수는 반드시 `선언`과 동시에 `초기화`해야 함!
  - 그렇지 않을 경우 `SyntaxError: Missing initializer in const declaration` 문법 오류를 출력함
- let 키워드와 마찬가지로 블록 레벨 스코프를 가지며, 변수 호이스팅이 발생하지 않는 것처럼 동작

### 상수

- var, let 키워드와는 달리 const 키워드로 선언한 변수는 `재할당이 금지`됨
- 원시 값을 할당할 경우 변수 값을 변경할 수 없기 때문에 const 키워드를 `상수를 표현하는 데 사용`함
  - 원시 값은 `immutable value` 즉, 변경할 수 없는 값을 const 키워드에 의해 재할당이 금지되므로 할당된 값을 변경할 수 있는 방법은 없음
- 주로 상수의 이름은 대문자로 선언해 상수임을 명확히 나타냄(`_`스네이크 케이스)
  - ex) TAX_RATE
- 재할당을 금지할 뿐 `불변`을 의미하지는 않음
  - const 키워드로 선언된 변수에 `객체를 할당한 경우`에는 `값을 변경`할 수 있음(변수에 할당된 `참조 값은 변경되지 않음`)

## 정리

- ES6에서는 var 키워드를 사용하지 않는 것이 좋음
- 재할당이 필요한 경우 let 키워드를 사용하나, 스코프는 최대한 좁게 만듦
- 변경이 발생하지 않는 원시 값과 객체는 const 키워드를 사용

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 208 ~ 218p
