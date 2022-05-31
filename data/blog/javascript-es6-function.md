---
title: ES6 함수의 추가 기능
date: '2022-05-31'
tags: ['javascript']
draft: false
summary: ES6 이전의 자바스크립트의 함수는 별다른 구분 없이 다양한 목적으로 사용됨
layout: PostSimple
authors: ['default']
---

# ES6 함수의 추가 기능

## 함수의 구분

- ES6 이전의 자바스크립트의 함수는 별다른 구분 없이 다양한 목적으로 사용됨
  - 편리할 것 같지만 실수를 유발할 수 있으며, 성능 면에서도 손해인 경우 발생
  - 생성자 함수로 호출되지 않아도 프로토 타입 객체를 생성하므로 혼란 유발
- 사용 목적에 따라 명확히 구분되지 않는 ES6 이전의 모든 함수는 `일반 함수로서 호출`할 수 있는 것은 물론 `생성자 함수로서 호출`할 수 있었음
  - 호출할 수 있는 함수 객체인 `callable`이면서 인스턴스를 생성할 수 있는 함수 객체인 `constructor`임

| ES6 함수의 구분    | constructor | prototype | super | arguments |
|---------------|-------------|-----------|-------|-----------|
| 일반 함수(Normal) | O           | O         | X     | O         |
| 메서드(Method)   | X           | X         | O     | O         |
| 화살표 함수(Arrow) | X           | X         | X     | X         |

## 메서드

- ES6 이전 사양에는 메서드에 대한 명확한 정의가 없었음
- ES6 사양에서 메서드는 `메서드 축약 표현`으로 `정의된 함수`만을 의미함

```javascript
const obj = {
  x: 1,
  // foo는 메서드
  foo() {
    return this.x;
  },
  // bar에 바인딩된 함수는 메서드가 아닌 일반 함수
  bar: function () {
    return this.x;
  }
};

console.log(obj.foo()); // 1
console.log(obj.bar()); // 1
```

- `ES6` 사양에서 정의한 메서드는 `인스턴스를 생성할 수 없는 non-constructor`

    ```javascript
    new obj.foo(); // -> TypeError: obj.foo is not a constructor
    new obj.bar(); // -> bar {}
    ```

  - 생성자 함수로서 호출할 수 없음


- 인스턴스도 생성 불가하므로 prototype 프로퍼티가 없고, 프로토타입도 생성하지 않음

    ```javascript
    obj.foo.hasOwnProperty('prototype'); // -> false
    obj.bar.hasOwnProperty('prototype'); // -> true
    ```

- 표준 빌트인 객체가 제공하는 `프로토타입 메서드`와 `정적 메서드`는 모두 `non-constructor`임
  - `String.prototype.toUpperCase.prototype`
  - `String.fromCharCode.prototype`
  - `Number.prototype.toFixed.prototype`
  - `Number.isFinite.prototype`
  - `Array.prototype.map.prototype`
  - `Array.from.prototype`


- ES6 메서드는 자신을 바인딩한 객체를 가리키는 내부 슬롯 [[HomeObject]]를 갖음
  - 내부 슬롯 [[HomeObject]]를 갖는 ES6 메서드는 `super 키워드`를 사용할 수 있음

## 화살표 함수

- function 키워드 대신 `화살표(⇒)를 사용`하여 기존 함수 정의 방식보다 `간략하게 함수 정의`할 수 있음
  - `this가 전역 객체를 가리키는 문제를 해결`하기 위한 대안으로 유용함

### 화살표 함수 정의

#### `함수 정의`

```javascript
const multiply = (x, y) => x * y;
multiply(2, 3); // -> 6
```

#### `매개 변수 선언`

```javascript
// 매개 변수 여러개
const arrow = (x, y) => { ... };

// 매개 변수 한개
const arrow = x => { ... };

// 매개 변수 없을 때
const arrow = () => { ... };
```

#### `함수 몸체 정의`

- 함수 표현식

    ```javascript
    // concise body
    const power = x => x ** 2;
    power(2); // -> 4
    
    // block body
    const power = x => { return x ** 2; };
    ```

  - `하나의 문`으로 구성된다면 `중괄호 {}를 생략`할 수 있고,
  - 내부의 문이 `값으로 평가`될 수 있는 표현식인 문이라면 `암묵적으로 반환`


- 객체 리터럴 반환

    ```javascript
    const create = (id, content) => ({ id, content});
    create(1, 'JavaScript'); // -> {id: 1, content: "Javascript"}
    
    // 위와 동일함
    const create = (id, content) => {return { id, content }; };
    ```

  - 객체 리터럴을 반환하는 경우 `소괄호 ()`로 감싸줘야 함


- 즉시 실행 함수(IIFE) 선언 예제

    ```javascript
    const person = (name => ({
      sayHi() { return `Hi? My name is ${name}.`; }
    }))('Lee');
    
    console.log(person.sayHi()); // Hi? My name is Lee.
    ```

  - 화살표 함수도 일급 객체이므로 map, filter, reduce와 같은 고차 함수에 인수로 전달할 수 있음

### 화살표 함수와 일반 함수 차이

1. 화살표 함수는 인스턴스를 생성할 수 없는 `non-constructor`임

    ```javascript
    const Foo = () => {};
    // 생성자 함수로서 호출될 수 없음
    new Foo(); // TypeError: Foo is not a constructor
    Foo.hasOwnProperty('prototype'); // -> false
    ```
   - prototype 프로퍼티도 없고, 프로토타입을 생성하지 않음


2. 중복된 매개변수 이름을 선언할 수 없음

     ```javascript
     const arrow = (a, a) => a + a;
     // SyntaxError: Duplicate parameter name not allowed in this context
     ```
   - 일반 함수(strict mode 제외)와 달리 중복된 매개변수 이름을 선언하면 에러가 발생함


3. 화살표 함수는 자체의 `this`, `arguments`, `super`, `new.target` 바인딩을 갖지 않음(⭐⭐⭐)

   - 함수 내부에서 스코프 체인을 통해 `상위 스코프`의 `this`, `arguments`, `super`, `new.target`을 참조함
   - `화살표 함수 중첩` 시, 스코프 체인 상에서 가장 가까운 함수 중에서 `화살표가 아닌 함수`의 `this`, `arguments`, `super`, `new.target`을 참조

### this

- 가장 구별되는 큰 특징중에 하나인 `this 바인딩`
- 일반 함수와 다르게 동작함
  - 콜백 함수 내부의 this가 외부 함수의 this와 다르기 때문에 발생하는 `문제를 해결하기 위해 설계`됨

#### `문제 인식`

```javascript
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix;
  }

  add(arr) {
    // add 메서드는 인수로 전달된 배열 arr을 순회하며 배여의 모든 요소에 prefix를 추가함
    // (1)
    return arr.map(function (item) {
      return this.prefix + item; // (2)
    })
  }
}

const prefixer = new Prefixer('-webkit-');
console.log(prefixer.add(['transition', 'user-select']));
```

- 예제의 결과 값을 우리는 `['-webkit-transition', '-webkit-user-select']`를 기대하지만 결과는 그렇지 않음
  - `TypeError: Cannot read property 'prefix' of undefined`
- 상위 코드 `(1)`에서 this는 메서드 호출한 객체(`prefixer`)를 가리킴
- 하지만 `(2)`에서 this는 `undefined`를 가리킴
  - `일반 함수`로서 `호출`되는 모든 함수 내부의 this는 `전역 객체를 가리킴`
  - `클래스 내부`의 모든 코드에는 `strict mode가 암묵적으로 적용`되므로, 콜백 함수에서도 적용되고 strict mode에서 `일반 함수로서 호출된 모든 함수 내부의 this`에는 전역 객체가
    아니라 `undefined`가 `반영됨`

> `Array.prototype.map` 메서드가 콜백 함수를 `일반 함수로서 호출`하기 때문

#### `문제 해결 - ES6 이전`

1. add 메서드를 호출한 prefixer 객체를 가리키는 `this를 일단 회피`시킨 후 `콜백 함수 내부`에서 사용

    ```javascript
    // ...
    add(arr) {
      // this 회피
      const that = this;
      return arr.map(function (item){
        // this 대신 that 참조
        return that.prefix + ' ' + item;
      });
    }
    // ...
    ```

2. Array.prototype.map의 두 번째 인수로 add 메서드를 호출한 `prefixer 객체를 가리키는 this 전달`

    ```javascript
    // ...
    add(arr) {
      return arr.map(function (item){
        return that.prefix + ' ' + item;
      }, this); 
    }
    // ...
    ```

   - this에 바인딩된 값이 콜백 함수 내부의 this에 바인딩됨

3. Function.prototype.bind 메서드 사용

    ```javascript
    // ...
    add(arr) {
      return arr.map(function (item){
        return that.prefix + ' ' + item;
      }, bind(this)); 
    }
    // ...
    ```

#### `문제 해결 - ES6`

```javascript
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix;
  }

  add(arr) {
    return arr.map(item => this.prefix + item);
  }
}

const prefixer = new Prefixer('-webkit-');
console.log(prefixer.add(['transition', 'user-select']));
// ['-webkit-transition', '-webkit-user-select']
```

- 화살표 함수는 함수 자체의 this 바인딩을 갖지 않음(⭐⭐⭐)
  - 함수 내부의 this를 참조하면 `상위 스코프의 this를 그대로 참조함`(`lexical this`)
  - 렉시컬 스코프와 같이 화살표의 `this는 함수가 정의된 위치에 결정된다는 것을 의미`

#### `일반 메서드 정의`

- 메서드를 화살표 함수로 정의하는 것은 피해야 함(일반적인 의미의 메서드)

```javascript
// Bad Case ❌
const person = {
  name: 'Lee',
  sayHi: () => console.log(`Hi ${this.name}`)
};

// sayHi 프로퍼티에 할당된 화살표 함수 내부의 this는 상위 스코프인 전역의 this가 가리키는 
// 전역 객체를 가리키므로 이 예제를 브라우저에서 실행하면 this.name은 빈 문자열을 갖는 window.name과 같음
// 전역 객체 window에는 빌트인 프로퍼티 name이 존재함
person.sayHi(); // Hi

// Good Case 🙋‍♂️
const person = {
  name: 'Lee',
  sayHi() {
    console.log(`Hi ${this.name}`);
  }
};

person.sayHi(); // Hi Lee
```

- 메서드를 정의할 때에는 `ES6 메서드 축약 표현`으로 사용하는 것이 좋음

#### `프로토타입 동적 추가`

- 일반 함수가 아닌 ES6 메서드를 동적 추가하고 싶으면 `객체 리터럴을 바인딩`하고 프로토타입의 `constructor 프로퍼티와 생성자 함수 간의 연결을 재설정함`

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype = {
  // constructor 프로퍼티와 생성자 함수 간의 연결을 재설정
  constructor: Person,
  sayHi() { console.log(`Hi ${this.name}`); }
};

const person = new Person('Lee');
person.sayHi(); // Hi Lee
```

### super

- super는 `[[HomeObject]]`를 갖는 `ES6 메서드 내에서만 사용`할 수 있는 키워드임

```javascript
class Base {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    return `Hi! ${this.name}`;
  }
}

class Derived extends Base {
  // 화살표 함수의 super는 상위 스코프인 constructor의 super를 가리킴
  sayHi = () => `${super.sayHi()} how are you doing?`;
}

const derived = new Derived('Lee');
console.log(derived.sayHi()); // Hi! Lee how are you doing?
```

- 화살표 함수는 함수 자체의 `super 바인딩을 갖지 않음`
  - 화살표 함수 내부에서 super를 참조하면 this와 마찬가지로 `상위 스코프의 super를 참조`함

### arguments

- 함수 자체의 `arguments 바인딩을 갖지 않음`
  - 함수 내부에서 arguments를 참조하면 this와 마찬가지로 `상위 스코프의 arguments를 참조`

```javascript
(function () {
  // 화살표 함수 foo의 arguments는 상위 스코프인 즉시 실행 함수의 arguments를 가리킴
  const foo = () => console.log(arguments); // [Arguments] { '0': 1, '1': 2 }
  foo(3, 4);
}(1, 2));

// 화살표 함수 foo의 arguments는 상위 스코프인 전역의 arguments를 가리킴
// 하지만 전역에는 arguments 객체가 존재하지 않음. arguments 객체는 함수 내부에서만 유효함
const foo = () => console.log(arguments);
foo(1, 2); // ReferenceError: arguments is not defined
```

- 화살표 함수에는 arguments 객체를 사용할 수 없음
- 가변 인자 함수를 구현할 때는 반드시 `Rest 파라미터를 사용`해야 함

## Rest 파라미터

### 기본 문법

- Rest 파라미터는 매개변수 이름 앞에 세 개의 점 … 을 붙여서 정의한 매개변수를 의미함
  - 함수에 전달된 인수들의 목록을 `배열로 전달 받음`

```javascript
function bar(param1, param2, ...rest) {
  console.log(param1); // 1
  console.log(param1); // 2
  console.log(rest); // [ 3, 4, 5 ]
}

bar(1, 2, 3, 4, 5);
```

- 선언된 매개변수에 할당된 인수를 제외한 나머지 인수들로 구성된 배열이 할당됨
- Rest 파라미터는 `반드시 마지막 파라미터`어야 함
- Rest 파라미터는 `단 하나만` 선언할 수 있음
- 함수 정의 시 선언한 매개변수 개수를 나타내는 함수 객체의 `length 프로퍼티에 영향을 주지 않음`

```javascript
function foo(...rest) {}

console.log(foo.length); // 0

function bar(x, ...rest) {}

console.log(bar.length); // 1

function bar(x, y, ...rest) {}
console.log(baz.length); // 2
```

### Rest 파라미터와 arguments 객체

- ES5에서는 함수 호출 시 인수들의 정보를 담고 있는 순회 가능한 `유사 배열 객체`를 함수 내부에 `지역 변수`처럼 사용할 수 있지만 call, apply 메서드를 사용해 배열로 변환해야 하는 번거로움이 있었음
- ES6에서는 `rest 파라미터`를 사용해 가변 인자 함수의 인수 목록을 `배열로 직접 전달 받을 수 있음`

    ```javascript
    function sum(...args){
    	// Rest 파라미터 args에는 배열 [1, 2, 3, 4, 5]가 할당됨
    	return arg.reduce((pre, cur) => pre + cur, 0);
    }
    console.log(sum(1, 2, 3, 4, 5)); // 15
    ```

  - 함수와 ES6 메서드는 `Rest 파라미터`와 `arguments 객체`를 모두 사용할 수 있음
    - `화살표 함수`는 arguments 객체를 사용하지 못하므로, `Rest 파라미터를 사용`해야 함

## 매개변수 기본값

- `자바스크립트 엔진`은 매개변수의 `개수`와 `인수의 개수를 체크하지 않음`
- 인수가 전달되지 않은 매개변수의 값은 undefined이므로 `의도치 않은 결과를 방지`하기 위해 기본값을 할당할 필요가 있음

### ES5

```javascript
function sum(x, y) {
  // 인수가 전달되지 않은 경우를 생각해서 기본값을 할당함
  x = x || 0;
  y = y || 0;

  return x + y;
}

console.log(sum(1, 2)); // 3
console.log(sum(1)); // 1
```

### ES6

- ES6에서 도입된 매개변수 기본값을 사용하면 함수 내에서 수행하던 `인수 체크` 및 `초기화를 간소화`할 수 있음

```javascript
function sum(x = 0, y = 0) {
  return x + y;
}

console.log(sum(1, 2)); // 3
console.log(sum(1));    // 1
```

- 매개변수에 `인수를 전달하지 않은 경우`와 `undefined를 전달한 경우`에만 유효함
- Rest 파라미터와 마찬가지로 함수 객체의 length 프로퍼티와 arguments 객체에 아무런 영향을 주지 않음

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 469 ~ 491p