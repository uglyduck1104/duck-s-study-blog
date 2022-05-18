---
title: 함수와 일급 객체
date: '2022-05-18'
tags: ['javascript']
draft: false
summary: 함수는 값을 사용할 수 있는 곳(변수 할당문, 객체의 프로퍼티 값, 배열의 요소, 함수 호출의 인수, 함수 반환문)이라면 어디서든지 리터럴로 정의할 수 있으며 런타임에 함수 객체로 평가됨
layout: PostSimple
authors: ['default']
---

# 함수와 일급 객체

## 일급 객체

> 아래의 조건에 부합하는 객체를 `일급 객체`라 함

1. 무명의 리터럴로 생성 가능
   - 런타임에 생성 가능
2. 변수나 자료구조(객체, 배열 등)에 저장 가능
3. 함수의 매개변수로 전달 가능
4. 함수의 반환값으로 사용 가능

```javascript
// 1. 함수는 무명의 리터럴로 생성 가능
// 2. 함수는 변수에 저장 가능
// 런타임(할당 단계)에 함수 리터럴 평가되어 함수 객체 생성 및 변수에 할당
const increase = function (num) {
  return ++num;
};

const decrease = function (num) {
  return --num;
};

// 2. 함수는 객체에 저장
const auxs = {increase, decrease};

// 3. 함수의 매개변수로 전달 가능
// 4. 함수의 반환값으로 사용 가능
function makeCounter(aux) {
  let num = 0;

  return function () {
    num = aux(num);
    return num;
  };
}

// 3. 함수는 매개변수에게 함수를 전달할 수 있다.
const increaser = makeCounter(auxs.increase);
console.log(increaser()); // 1
console.log(increaser()); // 2

// 3. 함수는 매개변수에게 함수를 전달할 수 있다.
const decreaser = makeCounter(auxs.decrease);
console.log(decrease()); // -1
console.log(decrease()); // -2
```

- 함수는 값을 사용할 수 있는 곳(`변수 할당문`, `객체의 프로퍼티 값`, `배열의 요소`, `함수 호출의 인수`, `함수 반환문`)이라면 어디서든지 리터럴로 정의할 수 있으며 런타임에 함수 객체로 평가됨

## 함수 객체의 프로퍼티

- 함수는 객체이므로 `프로퍼티를 가질 수 있음`

```javascript
function square(number) {
  return number * number;
}

console.log(Object.getOwnPropertyDescriptors(square));
/*
{
	length: {value: 1, writable: false, enumerable: false, configurable: true},
	name: {value: "square", writable: false, enumerable: false, configurable: true},
	arguments: {value: null, writable: false, enumerable: false, configurable: false},
	caller: {value: null, writable: false, enumerable: false, configurable: false},
	prototype: {value: {...}, writable: true, enumerable: false, configurable: false},
*/

// __proto__는 square 함수의 프로퍼티가 아님
console.log(Object.getOwnPropertyDescriptor(square, '__proto__')); // undefined

// __proto__는 Object.prototype 객체의 접근자 프로퍼티
// square 함수는 Object.prototype 객체로부터 __proto__ 접근자 프로퍼티를 상속받음
console.log(Object.getOwnPropertyDescriptor(Object.prototype, '__proto__'));
// {get: f, set: f, enumerable: false, configurable: true}
```

- `arguments`, `caller`, `length`, `name`, `prototype` 프로퍼티는 모두 함수 객체의 데이터 프로퍼티

### arguments 프로퍼티

> 함수 호출 시 전달된 인수들의 정보를 담고 있는 `순회 가능한 유사 배열 객체`


- `length 프로퍼티`를 가진 객체로 `for 문으로 순회`할 수 있는 객체
- 함수 외부에서는 참조할 수 없음
  - ES3부터 표준에서 폐지됨
- 함수 내부에서 지역 변수처럼 사용할 수 있는 `arguments 객체 참조`

```javascript
function multiply(x, y) {
  console.log(arguments);
  return x * y;
}

console.log(multiply());         // NaN
console.log(multiply(1));        // NaN
console.log(multiply(1, 2));     // 2
console.log(multiply(1, 2, 3));  // 2
```

- `매개변수`와 `인수의 개수`가 일치하는지 `확인하지 않음`
  - 매개변수 개수만큼 인수를 전달하지 않아도 `에러가 발생하지 않음`
- 함수가 호출되면 함수 몸체 내에서 암묵적으로 `매개변수가 선언`되고 `undefined로 초기화`된 이후 `인수 할당`
- 인수를 프로퍼티 값으로 소유하며 `프로퍼티 키`는 `인수의 순서`를 나타냄
- callee 프로퍼티
  - arguments 객체를 생성한 함수(자기 자신)
- length 프로퍼티
  - 인수의 개수

> 매개변수 개수를 확정할 수 없는 `가변 인자 함수를 구현`할 때 유용함

### caller 프로퍼티

> ECMAScript 사양에는 포함되지 않는 비표준 프로퍼티로서 `함수 자신을 호출한 함수`를 가리킴

### length 프로퍼티

```javascript
function foo() {
}

console.log(foo.length); // 0

function bar(x) {
  return x;
}

console.log(bar.length); // 1

function baz(x, y) {
  return x * y;
}

console.log(baz.length); // 2
```

> 선언한 `매개변수의 개수`를 가리킴

- `arguments` 객체의 `length` 프로퍼티
  - 인자의 개수
- `함수` 객체의 `length` 프로퍼티
  - 매개변수의 개수

### name 프로퍼티

> 함수 객체의 name 프로퍼티는 `함수 이름`을 나타냄

- `ES6`부터 정식 표준으로 채택된 프로퍼티
- ES5에서는 `빈 문자열`을 값으로 갖지만 ES6에서부터 함수 객체를 가리키는 `식별자 값`으로 변경
  - 동작을 달리하므로 주의해야 함

```javascript
// 기명 함수 표현식
var nameFunc = function foo() {
};
console.log(namedFunc.name); // foo

// 익명 함수 표현식
var anonymousFunc = function () {
};
// ES5: name 프로퍼티는 빈 문자열을 값으로 갖는다
// ES6: name 프로퍼티는 함수 객체를 가리키는 변수 이름을 값으로 갖음
console.log(anonymousFunc.name); // anonymousFunc

// 함수 선언문(Function declaration)
function bar() {
}

console.log(bar.name); // bar
```

### __proto__ 접근자 프로퍼티

> [[Prototype]] 내부 슬롯이 가리키는 프로토타입 객체에 접근하기 위해 사용하는 `접근자 프로퍼티`

```javascript
const obj = {a: 1};

// 객체 리터럴 방식으로 생성한 객체의 프로토타입 객체는 Object.prototype임
console.log(obj.__proto__ === Object.prototype); // true

// 객체 리터럴 방식으로 생성한 객체는 프로토타입 객체인 Object.prototype의 프로퍼티를 상속받음
// hasOwnProperty 메서드는 Object.prototype의 메서드
console.log(obj.hasOwnProperty('a')); // true
console.log(obj.hasOwnProperty('__proto__')); // false
```

`hasOwnProperty` 메서드

- 인수로 전달받은 프로퍼티 키가 `객체 고유의 프로퍼티 키`인 경우에만 `true`를 반환하고 `상속받은 프로토타입의 프로퍼티 키`인 경우 `false`를 반환

### prototype 프로퍼티

> 생성자 함수로 호출할 수 있는 함수 객체, 즉 `constructor만 소유하는 프로퍼티`

```javascript
// 함수 객체는 prototype 프로퍼티를 소유함
(function () {
}).hasOwnProperty('prototype'); // -> true

// 일반 객체는 prototype 프로퍼티를 소유하지 않음
({}).hasOwnProperty('prototype'); // -> false
```

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 234 ~ 248p
