---
title: 생성자 함수에 의한 객체 생성
date: '2022-05-17'
tags: ['javascript']
draft: false
summary: 객체 리터럴에 의한 객체 생성 방식과 달리 new 연산자와 함께 Object 생성자 함수를 호출하면 빈 객체를 생성하여 반환함
layout: PostSimple
authors: ['default']
---

# 생성자 함수에 의한 객체 생성

## Object 생성자 함수

- 객체 리터럴에 의한 객체 생성 방식과 달리 new 연산자와 함께 Object 생성자 함수를 호출하면 빈 객체를 생성하여 반환함
  - `생성자 함수`
    - new 연산자와 함께 호출하여 객체를 생성하는 함수
  - `인스턴스`
    - 생성자 함수에 의해 생성된 객체

```javascript
// 빈 객체의 생성
const person = new Object();

// 프로퍼티 추가
person.name = 'Lee';
person.sayHello = function () {
  console.log("Hi! My name is ' + this.name);
};

console.log(person); // {name: "Lee", sayHello: f}
person.sayHello(); // Hi! My name is Lee
```

- Object 생성자 함수 이외에 String, Number, Boolean, Function, Array, Date, RegExp, Promise 등의 빌트인 생성자 함수를 제공함

## 생성자 함수

### 객체 리터럴 vs 생성자 함수

`객체 리터럴`

- 객체 리터럴에 의한 객체 생성 방식은 `단 하나의 객체`만 생성함
- 둘 이상의 객체를 생성할 경우 비효율적임

`생성자 함수`

- 객체를 생성하기 위한 템플릿(클래스)처럼 생성자 함수를 사용하여 프로퍼티 구조가 동일한 객체 여러 개를 간편하게 생성할 수 있음

```javascript
// 생성자 함수
function Circle(radius) {
  // 1. 암묵적으로 빈 객체가 생성되고 this에 바인딩됨 -> Circle {}

  // 2. this에 바인딩되어 있는 인스턴스를 초기화함
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };

  // 3. 완성된 인스턴스가 바인딩된 this가 암묵적으로 반환
  // 명시적 반환 방법
  // -> return {};
}

// 인스턴스 생성
const circle1 = new Circle(5); // 반지름이 5인 Circle 객체 생성

console.log(circle); // Circle {radius: 5, getDiameter: f}
```

- 생성자 함수는 일반 함수와 동일한 방법으로 정의하고 `new 연산자`와 함께 호출하면 생성자 함수로 동작함

> 자바스크립트 엔진은 아래와 같은 과정을 거쳐 암묵적으로 인스턴스를 `생성`하고 `초기화`한 후 `반환`함

**1. 생성자 함수의 인스턴스 생성과 this 바인딩**

- 암묵적으로 빈 객체(인스턴스)가 생성되면 `this에 바인딩`됨
  - 생성자 함수 내부의 `this`가 생성자 함수가 생성할 인스턴스를 가리킴
  - 함수 몸체의 코드가 한 줄씩 실행되는 `런타임 이전`에 실행

**2. 인스턴스 초기화**

- this에 바인딩되어 있는 인스턴스에 프로퍼티나 메서드를 추가하고 생성자 함수가 인수로 전달받은 초기값을 인스턴스 프로퍼티에 할당하여 초기화하거나 고정값을 할당함

**3. 인스턴스 반환**

- this가 아닌 다른 객체를 명시적으로 반환하면 this가 반환되지 못하고 return 문에 명시한 객체가 반환됨
  - this가 아닌 다른 값 반환 시 생성자 함수 기본 동작 훼손을 야기하므로 `return문을 생략`

### 내부 메서드 [[Call]]과 [[Construct]]

```javascript
// 함수는 객체
function foo() {
}

// 함수는 객체이므로 프로퍼티를 소유할 수 있음
foo.prop = 10;

// 함수는 객체이므로 메서드를 소유할 수 있음
foo.method = function () {
  console.log(this.prop);
};

foo.method(); // 10
```

- 함수는 객체이지만 일반 객체와는 다름
  - 일반 객체는 호출할 수 없지만 `함수는 호출`할 수 있음
- 함수 객체는 [[Environment]], [[FormalParameters]] 등의 내부 슬롯과 [[Call]], [[Construct]] 같은 내부 메서드를 추가로 가짐
- 함수가 호출되는 형태에 따라 내부 메서드 호출 방식이 달라짐

    ```javascript
    function foo() {}
    
    // 일반적인 함수로서 호출: [[Call]]이 호출
    foo();
    
    // 생성자 함수로서 호출: [[Construct]]가 호출
    new foo();
    ```

  - 일반 함수 → 내부메서드 `[[Call]]` 호출
    - callable(`함수 객체`)
  - 생성자 함수 → 내부메서드 `[[Construct]]` 호출
    - constructor(`함수 객체`)
      - 생성자 함수로서 호출할 수 있는 함수
    - non-constructor(`함수 객체`)
      - 생성자 함수로서 호출할 수 없는 함수

<img alt="js_construct_function1" src="/static/images/js_construct_function1.png" width="700"/>

> 모든 함수 객체는 호출(`callable`)할 수 있지만 모든 함수 객체를 생성자 함수(`contructor`)로서 호출할 수 있는 것은 아님

### constructor / non-constructor 구분

- 자바스크립트 엔진은 함수 정의를 평가하여 함수 정의 방식에 따라 구분함
  - constructor
    - 함수 선언문, 함수 표현식, 클래스

      ```javascript
      // 일반 함수 정의: 함수 선언문, 함수 표현식
      function foo() {}
      const bar = function () {};
        
      new foo(); // -> foo{}
      new bar(); // -> bar{}
      ```

  - non-constructor
    - 메서드(ES6 메서드 축약 표현), 화살표 함수

      ```javascript
      // 화살표 함수 정의
      const arrow = () => {};
        
      new arrow(); // TypeError: arrow is not a constructor
        
      // 메서드 정의: ES6의 메서드 축약 표현만 메서드로 인정
      const obj = {
        x() {}
      };
        
      new obj.x(); // TypeError: obj.x is not a constructor
      ```

> 생성자 함수로서 호출될 것을 기대하고 정의하지 않은 일반 함수(callable이면서 constructor)에 `new 연산자`를 붙여 호출하면 `생성자 함수`처럼 동작할 수 있음

### new 연산자

- 함수 객체의 내부 메서드 [[Call]]이 호출되는 것이 아니라 `[[Construct]]`가 호출됨
  - new 연산자와 함께 호출하는 함수는 `constructor`이어야 함

### new.target

- new 연산자 없이 호출되는 것을 방지하기 위해 파스칼 케이스 컨벤션을 사용함
- this와 유사하게 constructor인 모든 함수 내부에서 암묵적인 지역 변수와 같이 사용됨

> new 연산자와 함께 생성자 함수로서 호출되면 함수 내부의 new.target은 함수 자신을 가리킴
  
```javascript
// 생성자 함수
function Circle(radius) {
  // 이 함수가 new 연산자와 함께 호출되지 않았다면 undefined 반환
  if (!new.target) {
    // new 연산자와 함께 생성자 함수를 재귀 호출 -> 인스턴스 반환
    return new Circle(radius);
  }
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

// new 연산자 없이 생성자 함수를 호출하여도 new.target을 통해 생성자 함수로서 호출됨
const circle = Circle(5);
console.log(circle.getDiameter());
```

### 빌트인 생성자 함수

- Object와 Function 생성자 함수는 new 연산자 없이 호출해도 동일하게 동작함
- String, Number, Boolean 생성자 함수는 new 연산자와 함께 호출했을 때와 그렇지 않은 경우 다르게 반환함

```javascript
const str = String(123);
console.log(str, typeof str); // 123 string

const num = Number('123');
console.log(num, typeof num); // 123 number

const bool = Boolean('true');
console.log(bool, typeof bool); // true boolean
```

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 234 ~ 248p
