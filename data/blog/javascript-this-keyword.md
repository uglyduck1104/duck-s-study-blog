---
title: this
date: '2022-05-26'
tags: ['javascript']
draft: false
summary: this를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있음
layout: PostSimple
authors: ['default']
---

# this

## this 키워드

- 객체는 `상태`를 나타내는 `프로퍼티`와 `동작`을 나타내는 `메서드`를 하나의 논리적인 단위로 묶은 복합적인 자료구조
- 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 `자기 참조 변수`
  - this를 통해 자신이 속한 `객체` 또는 `자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조`할 수 있음
  - 자바스크립트 엔진에 의해 `암묵적으로 생성`, 코드 `어디서든 참조`할 수 있음

> this가 가리키는 값, 즉 this 바인딩은 `함수 호출 방식`에 의해 `동적으로 결정`됨 (⭐⭐⭐⭐⭐)

### 동작 예제

```javascript
// 객체 리터럴
const circle = {
  radius: 5,
  getDiameter() {
    // this는 메서드를 호출한 객체를 가리킴
    return 2 * this.radius;
  }
};

console.log(circle.getDiameter()); // 10
```

- `객체 리터럴`의 메서드 내부에서는 this는 `메서드를 호출한 객체`를 가리킴

```javascript
// 생성자 함수
function Circle(radius) {
  // this는 생성자 함수가 생성할 인스턴스를 가리킴
  this.radius = radius;
}

Circle.prototype.getDiameter = function () {
  // this는 생성자 함수가 생성할 인스턴스를 가리킴
  return 2 * this.radius;
};

// 인스턴스 생성
const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

- 생성자 함수 내부의 this는 `생성자 함수가 생성할 인스턴스를 가리킴`

> 자바스크립트의 this는 함수가 호출되는 방식에 따라 this에 바인딩될 값, 즉 this 바인딩이 `동적으로 결정됨`

- 이와 달리, 자바나 C++ 같은 클래스 기반 언어에서는 언제나 this는 `클래스가 생성하는 인스턴스`를 가리킴
- 객체의 프로퍼티나 메서드를 참조하기 위한 `자기 참조 변수`이므로 `객체의 메서드 내부`, `생성자 함수 내부`에서만 의미가 있음

## 함수 호출 방식과 this 바인딩

- 앞서 살펴본 바와 같이 this 바인딩은 `함수 호출 방식에 동적으로 결정`된다고 했음
- 주의해야 할 점은 `동일한 함수도 다양한 방식으로 호출`할 수 있음

### 일반 함수 호출

- 기본적으로 this에는 `전역 객체`가 바인딩됨

```javascript
function foo() {
  console.log("foo's this: ", this); // window
  function bar() {
    console.log("bar's this: ".this); // window
  }

  bar();
}

// foo 함수를 일반적인 방식으로 호출
// foo 함수 내부의 this는 전역 객체 window를 가리킴
foo(); // window
```

- 전역 함수는 물론이고 어떤 함수라도 `일반 함수로 호출`되면 함수 내부의 this에는 `전역 객체`가 바인딩됨
- strict mode가 적용된 일반 함수 내부의 this에는 `undefined`가 바인딩됨

### 메서드 호출

```javascript
const person = {
  name: 'Lee',
  getName() {
    // 메서드 내부의 this는 메서드를 호출한 객체에 바인딩됨
    return this.name;
  }
};

// 메서드 getName을 호출한 객체는 person
console.log(person.getName()); // Lee
```

- person 객체의 getName 프로퍼티가 가리키는 함수 객체는 person 객체에 포함된 것이 아니라 `독립적으로 존재하는 별도의 객체`

<img alt="javascript_this_keyword1" src="/static/images/javascript_this_keyword1.png" width="700"/>

- getName 프로퍼티가 가리키는 함수 객체, 즉 `getName 메서드`는 `다른 객체의 프로퍼티에 할당`하는 것으로 `다른 객체의 메서드`가 될 수도 있고 일반 변수에
  할당하여 `일반 함수로 호출될 수도 있음`
- 프로퍼티로 메서드를 가리키고 있는 객체와는 관계가 없고, `호출한 객체에 바인딩`

```javascript
const anotherPerson = {
  name: 'Kim';
};

// getName 메서드를 anotherPerson 객체의 메서드로 할당
anotherPerson.getName = person.getName;

// getName 메서드를 호출한 객체는 anotherPerson임
console.log(anotherPerson.getName()); // Kim

// getName 메서드를 변수에 할당
const getName = person.getName

// getName 메서드를 일반 함수로 호출
console.log(getName()); // ''
// 일반 함수로 호출된 getName 함수 내부의 this.name은 브라우저 환경에서 window.name과 같음
// 브라우저 환경에서 window.name은 브라우저 창의 이름을 나타내는 빌트인 프로퍼티로서 기본값은 ''
// Node.js 환경에서 this.name은 undefined임
```

<img alt="javascript_this_keyword2" src="/static/images/javascript_this_keyword2.png" width="750"/>

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.getName = function () {
  return this.name;
};

const me = new Person('Lee');

// getName 메서드를 호출한 객체는 me
console.log(me.getName()); // 1. Lee

Person.prototype.name = 'Kim';

// getName 메서드를 호출한 객체는 Person.prototype
console.log(Person.prototype.getName()); // 2. Kim
```

- `1`의 경우 getName 메서드 호출한 객체는 `me`
  - 따라서 getName 메서드 내부의 this는 me를 가리키고 this.name은 `‘Lee’`
- `2`의 경우 getName 메서드를 호출한 객체는 `Person.prototype`
  - Person.prototype도 객체이므로 직접 메서드를 호출할 수 있음
  - 결과적으로 Person.prototype을 가리키며 this.name은 `‘Kim’`

![javascript_this_keyword3](/static/images/javascript_this_keyword3.png)

### 생성자 함수 호출

```javascript
// 생성자 함수
function Circle(radius) {
  // 생성자 함수 내부의 this는 생성자 함수가 (미래에) 생성할 인스턴스를 가리킴
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

// 반지름이 5인 Circle 객체를 생성
const circle1 = new Circle(5);

console.log(circle1.getDiameter()); // 10
```

- 생성자 함수는 이름 그대로 객체를 생성하는 함수이므로 `new 연산자와 함께 호출`하면 해당 함수는 `생성자 함수로 동작`함
  - new 연산자와 함께 생성자 함수를 호출하지 않으면 생성자 함수가 아니라 일반 함수로 동작함

### `Function.prototype.apply`/`call`/`bind` 메서드에 의한 간접 호출

- apply, call, bind 메서드는 `Function.prototype`의 메서드

<img alt="javascript_this_keyword4" src="/static/images/javascript_this_keyword4.png" width="700"/>

```text
/**
 * 주어진 this 바인딩과 인수 리스트 배열을 사용하여 함수를 호출
 * @param thisArg - this로 사용할 객체
 * @param argsArray - 함수에게 전달할 인수 리스트의 배열 또는 유사 배열 객체
 * @returns 호출된 함수의 반환값
 */
Function.prototype.apply(thisArg[, argsArray])

/**
 * 주어진 this 바인딩과 ,로 구분된 인수 리스트를 사용하여 함수를 호출
 * @param thisArg - this로 사용할 객체
 * @param arg1, arg2, ... - 함수에게 전달할 인수 리스트
 * @returns 호출된 함수의 반환값
 */
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

- apply와 call 메서드의 본질적인 기능은 `함수 호출`
  - 첫번째 인수로 전달한 특정 객체를 호출한 함수의 this에 바인딩
  - 호출할 함수에 인수전달 방식만 다를 뿐 `동일하게 동작함`
  - `apply` 메서드
    - 호출할 함수의 인수를 배열로 묶어 전달
  - `call` 메서드
    - 호출할 함수의 인수를 쉼표로 구분한 리스트 형식으로 전달
- `arguments` 객체와 같은 `유사 배열 객체`에 배열 메서드를 사용하는 경우에 주로 많이 쓰임
  - `Array.prototype.slice`와 같은 배열의 메서드를 사용할 수 있게 해줌

    ```javascript
    function converterArgsToArray() {
    	console.log(arguments);
    
    	// arguments 객체를 배열로 변환
    	// Array.prototype.slice를 인수 없이 호출하면 배열의 복사본 생성
    	const arr = Array.prototype.slice.call(arguments);
    	//const arr = Array.prototype.slice.apply(arguments);
    	console.log(arr);
    
    	return arr;
    }
    
    convertArgsToArray(1, 2, 3); // [1, 2, 3]
    ```

- bind 메서드는 apply, call 메서드와 달리 `함수를 호출하지 않지만` 첫번째 인수로 전달한 값으로 this 바인딩이 교체된 함수를 새롭게 생성해 반환함

    ```javascript
    function getThisBinding() {
    	return this;
    }
    
    // this로 사용할 객체
    const thisArg = { a: 1 };
    
    // bind 메서드는 첫 번째 인수로 전달한 thisArg로 this 바인딩이 교체된
    // getThisBinding 함수를 새롭게 생성해 반환함
    console.log(getThisBinding.bind(thisArg)); // getThisBinding
    // bind 메서드는 함수를 호출하지는 않으므로 명시적으로 호출해야 함
    console.log(getThisBinding.bind(thisArg)()); // { a: 1 }
    ```

    ```javascript
    const person = {
    	name: 'Lee',
    	foo(callback) {
    		// bind 메서드로 callback 함수 내부의 this 바인딩을 전달
    		setTimeout(callback.bind(this), 100);
    	}
    };
    
    person.foo(function() {
    	console.log(`Hi! My name is ${this.name}.`); // HI! my name is Lee.
    });
    ```

## 정리

| 함수 호출 방식                                         | this 바인딩                                               |
|--------------------------------------------------|--------------------------------------------------------|
| 일반 함수 호출                                         | 전역 객체                                                  |
| 메서드 호출                                           | 메서드를 호출한 객체                                            |
| 생성자 함수 호출                                        | 생성자 함수가 (미래에) 생성할 인스턴스                                 |
| Function.prototype.apply/call/bind 메서드에 의한 간접 호출 | Function.prototype.apply/call/bind 메서드에 첫번째 인수로 전달한 객체 |

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 342 ~ 358p