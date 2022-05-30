---
title: 클로저
date: '2022-05-30'
tags: ['javascript']
draft: false
summary: 클로저는 함수와 그 함수가 선언된 렉시컬 환경과의 조합이다
layout: PostSimple
authors: ['default']
---

# 클로저

- 클로저는 자바스크립트 고유의 개념이 아님
  - 함수형 프로그래밍 언어(하스켈, 리스프, 얼랭, 스칼라 등)에서 사용되는 특성 중 하나
  - 고유 개념이 아니므로 ECMAScript 사양에 등장하지 않음

> 클로저는 `함수`와 그 함수가 선언된 `렉시컬 환경`과의 `조합`이다\
> **\<MDN\>**

## 렉시컬 스코프

- 함수를 어디서 호출했는지가 아니라 `함수를 어디에 정의`했는지에 따라 `상위 스코프를 결정`함
  - 함수를 어디서 호출하는지는 상위 스코프 결정에 어떤 영향도 주지 못함
  - 상위 스코프 결정은 `외부 렉시컬 환경`에 대한 참조에 저장할 `참조값을 결정`
- 상위 스코프에 대한 참조는 함수 정의가 평가되는 시점에 `함수가 정의한 환경에 의해 결정`

## 함수 객체의 내부 슬롯 `[[Environment]]`

- 함수는 자신의 내부 슬롯 `[[Environment]]`에 자신이 정의된 환경, `상위 스코프의 참조를 저장`함
  - `[[Environment]]`에 저장된 상위 스코프의 참조는 `현재 실행 중인 실행 컨텍스트의 렉시컬 환경`을 가리킴

### 예제

```javascript
const x = 1;

function foo() {
  const x = 10;
  // 상위 스코프는 함수 정의 환경에 따라 결정됨
  // 함수 호출 위치와 스코프는 아무런 관계가 없음
  bar();
}

// 함수 bar는 자신의 상위 스코프, 전역 렉시컬 환경을 [[Environment]]에 저장하여 기억함
function bar() {
  console.log(x);
}

foo(); //?
bar(); //?
```

### 실행 컨텍스트

![javascript_closures1](/static/images/javascript_closures1.png)

- foo, bar 함수 모두 전역에서 `함수 선언문으로 정의`됨
  - 전역 코드가 평가되는 시점에 함수 객체르 생성하고 전역 객체 window의 메서드가 됨
  - `[[Environment]]` 내부 슬롯에는 실행 중인 실행 컨텍스트의 렉시컬 환경인 `전역 렉시컬 환경의 참조`가 저장됨(그림에서의 1번)

#### `함수 코드 평가 순서`

1. 함수 실행 컨텍스트 생성
2. 함수 렉시컬 환경 생성 
   - 2.1 함수 환경 레코드 생성
   - 2.2 this 바인딩
   - 2.3 외부 렉시컬 환경에 대한 참조 결정

> 외부 렉시컬 환경에 대한 참조에는 함수 객체의 내부 슬롯에 저장된 `렉시컬 환경의 참조가 할당`됨(그림에서 2,3번)

## 클로저와 렉시컬 환경

```javascript
const x = 1;

// (1)
function outer() {
  const x = 10;
  const inner = function () {
    console.log(x);
  } // (2)
  return inner;
}

// outer 함수를 호출하면 중첩 함수 inner를 반환
// outer 함수의 실행 컨텍스트는 실행 컨텍스트 스택에 팝되어 제거됨
const innerFunc = outer(); // (3)
innerFunc() // (4) 10
```

- 생명 주기가 종료되어 실행 컨택스트 스택에서 제거된 outer 함수의 지역 변수 x가 정상적으로 동작되는 듯해보임

> 외부 함수보다 `중첩 함수가 더 오래 유지되는 경우` 중첩 함수는 이미 생명 주기가 종료된 `외부 함수의 변수를 참조`할 수 있음

- 자바스크립트의 모든 함수는 `자신의 상위 스코프를 기억`하므로 어디서 호출하든 상관없이 유지됨

### 실행 순서와 실행 컨텍스트 스택

1. Outer 함수 평가 및 함수 객체 생성

   ![javascript_closures2](/static/images/javascript_closures2.png)

   - 전역 렉시컬 환경을 outer 함수 객체의 [[Environment]] 내부 슬롯에 `상위 스코프`로 저장

2. Outer 함수 호출

   ![javascript_closures3](/static/images/javascript_closures3.png)

   - Outer 함수의 `렉시컬 환경이 생성`됨
     - [[Environment]] 내부 슬롯에 저장된 렉시컬 환경을 outer 함수 `외부 렉시컬 환경에 대한 참조`에 할당함
   - 중첩 함수(inner) 평가
     - `함수 표현식`으로 정의했으므로 `런타임에 평가`
     - `outer 함수의 렉시컬 환경`을 `상위 스코프`로 저장

3. Outer 함수 종료

   ![javascript_closures4](/static/images/javascript_closures4.png)

   - outer 함수는 실행 컨텍스트 `스택에서 제거`
     - 실행 컨텍스트 스택은 제거되었으나, `outer 함수의 렉시컬 환경까지 소멸하는 것은 아님`(⭐⭐⭐⭐)
     - `outer 함수`는 inner 함수의 [[Environment]] `내부 슬롯에 의해 참조`되어 있고, `inner 함수` 역시 `전역 변수 innerFunc에 의해 참조`되므로 가비지 컬렉션의 대상이 되지
       않음
       - 누군가 참조하고 있는 메모리 공간을 함부로 해제하지 않음
   - `inner 함수 반환` 및 outer 함수의 생명 주기 종료

4. Inner 함수 호출

   ![javascript_closures5](/static/images/javascript_closures5.png)

   - inner 함수의 실행 컨텍스트 `생성` 및 `스택 푸시`
   - 외부 렉시컬 환경에 대한 참조
     - `inner 함수 객체`의 [[Environment]] 내부 슬롯에 저장되어 있는 `참조 값 할당`
   - 외부 함수보다 더 오래 생존한 중첩 함수는 외부 함수의 생존 여부와 상관없이 `자신이 정의된 위치에 의해 결정된 상위 스코프를 기억함`
     - inner의 내부에서는 상위 스코프를 참조할 수 있으므로 `상위 스코프의 식별자`를 `참조`할 수 있고 `값`을 `변경`할 수도 있음

### 클로저가 아닌 경우

- `상위 스코프`의 어떤 식별자도 `참조하지 않는 함수`
- 외부 함수보다 `중첩 함수의 생명주기가 짧은 경우`
  - 외부 함수보다 일찍 소멸되므로 `생명 주기가 종료된 외부 함수의 식별자를 참조`할 수 있다는 클로저의 본집에 부합하지 않음

### 클로저인 경우

```javascript
function foo() {
  const x = 1;
  const y = 2;

  // 클로저
  // 중첩 함수 bar는 외부 함수보다 더 오래 유지되며 상위 스코프의 식별자를 참조
  function bar() {
    console.log(x);
  }

  return bar;
}

const bar = foo();
bar();
```

- 상위 스코프의 식별자를 참조하고 있음
- 외부 함수의 외부로 반환되어 외부 함수보다 더 오래 살아남음

> 중첩 함수가 상위 스코프의 식별자를 참조하고 있고 `중첩 함수가 외부 함수보다 더 오래 유지되는 경우`에 한정함

## 클로저의 활용

- `상태를 안전하게 변경`하고 `유지`하기 위해 사용
- 의도치 않게 변경되지 않도록 `상태를 안전하게 은닉`하고 `특정 함수에게만 상태 변경을 허용`함

### 카운트 상태 변경 함수 - IIFE

```javascript
// 카운트 상태 변경함수
const increase = (function () {
  // 카운트 상태 변수
  let num = 0;

  // 클로저인 메서드를 갖는 객체 반환
  // 객체 리터럴은 스코프를 만들지 않음
  // 아래 메서드들의 상위 스코프는 즉시 실행 함수의 렉시컬 환경임
  return {
    // num: 0, // 프로퍼티는 public하므로 은닉되지 않음
    increase() {
      return ++num;
    },
    decrease() {
      return num > 0 ? --num : 0;
    }

  };
}());

console.log(counter.increase()); // 1
console.log(counter.increase()); // 2

console.log(counter.decrease()); // 1
console.log(counter.decrease()); // 0
```

- `즉시 실행 함수가 반환한 클로저`는 카운트 상태를 유지하기 위한 `자유 변수 num을 언제 어디서 호출하든지 참조하고 변경`할 수 있음
- num 변수는 외부에서 직접 접근할 수 없는 `은닉된 private 변수`이므로 안정적인 프로그래밍 가능

### 카운트 상태 변경 함수 - 함수형 프로그래밍

- `변수 값`은 누군가에 의해 `언제든지 변경`될 수 있으므로, `오류 발생의 근본적 원인`이 됨
  - `외부 상태 변경`이나 `가변 데이터를 피하고 불변성을 지향`하는 함수형 프로그래밍에서 부수 효과를 최대한 억제하여 오류를 피하고 프로그래밍의 안전성을 높이기 위해 클로저는 적극적으로 사용됨

```javascript
// 함수를 반환하는 고차 함수
// 카운트 상태를 유지하기 위한 자유 변수 counter를 기억하는 클로저 반환
const counter = (function () {
  // 카운트 상태를 유지하기 위한 자유 변수
  let counter = 0;

  // 함수를 인수로 전달받는 클로저를 반환
  return function (aux) {
    // 인수로 전달받은 보조 함수에 상태 변경을 위임함
    counter = aux(counter);
    return counter;
  };
}());

// 보조 함수
function increase(n) {
  return ++n;
}

// 보조 함수
function decrease(n) {
  return --n;
}

// 보조 함수를 전달하여 호출
console.log(counter(increase)); // 1
console.log(counter(increase)); // 2

// 자유 변수를 공유
console.log(counter(increase)); // 1
console.log(counter(increase)); // 0
```

- 카운트를 유지하기 위한 `자유 변수를 공유`하여 `카운터의 증감`을 연동

> `렉시컬 환경을 공유`하는 클로저를 만들어야 함


> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 388 ~ 408p