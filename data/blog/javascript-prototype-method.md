---
title: 프로토타입 교체와 메서드
date: '2022-05-24'
tags: ['javascript']
draft: false
summary: 프로토타입 교체, 존재 확인, 상속과 열거
layout: PostSimple
authors: ['default']
---

# 프로토타입 교체와 메서드

## 프로토타입의 교체

- 프로토타입은 임의의 다른 객체로 변경 가능
  - 부모 객체인 프로토타입을 동적으로 변경할 수 있음

### `생성자 함수`에 의한 교체

```javascript
const Person = (function () {
  function Person(name) {
    this.name = name;
  }

  // 1. 생성자 함수의 prototype 프로퍼티를 통해 프로토 타입을 교체
  Person.prototype = {
    sayHello() {
      console.log(`Hi! My name is ${this.name}`);
    }
  };

  return Person;
}());

const me = new Person('Lee');
```

<img alt="javascript_prototype_method_1" src="/static/images/javascript_prototype_method_1.png" width="750"/>

- 프로토타입으로 교체한 객체 리터럴에는 `constructor 프로퍼티가 없음`
  - 자바스크립트 엔진이 프로토타입을 생성할 때 암묵적으로 추가하므로, `me 객체의 생성자 함수를 검색`하면 Person이 아닌 `Object`가 나옴

  ```javascript
  console.log(me.constructor === Object); // true
  ```

> `프로토타입을 교체`하면 constructor 프로퍼티와 생성자 함수 간의 `연결이 파괴`됨

```javascript
const Person = (function () {
  function Person(name) {
    this.name = name;
  }

  // 생성자 함수의 prototype 프로퍼티를 통해 프로토타입을 교체
  Person.prototype = {
    // constructor 프로퍼티와 생성자 함수 간의 연결 설정
    constuctor: Person,
    sayHello() {
      console.log(`Hi! My name is ${this.name}`);
    }
  };

  return Person;
}());

const me = new Person('Lee');
console.log(me.constructor === Person); // true
```

### `인스턴스`에 의한 프로토타입의 교체

- 인스턴스의 `__proto__` 접근자 프로퍼티를 통해 교체가 가능함

```javascript
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

// 프로토타입으로 교체할 객체
const parent = {
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  }
};

// 1. me 객체의 프로토타입을 parent 객체로 교체
Object.setPrototypeOf(me, parent);
// me.__proto__ = parent; 와 동일하게 동작함

me.sayHello();
```

<img alt="javascript_prototype_method_2" src="/static/images/javascript_prototype_method_2.png" width="750"/>

- 생성자 함수와 마찬가지고 프로토타입의 교체로 인해서 `constructor 프로퍼티`와 `생성자 함수` 간의 `연결이 파괴`됨
- 생성자 함수에 의한 프로토타입 교체와 별다른 차이가 없어보이지만 아래 그림에서 처럼 다른 차이점이 있음

<img alt="javascript_prototype_method_3" src="/static/images/javascript_prototype_method_3.png" width="750"/>

## instanceof 연산자

- `좌변`에 `객체`를 가리키는 `식별자`, `우변`에 `생성자 함수`를 가리키는 식별자를 `피연산자(반드시 함수)`로 받음

```
객체 instanceof 생성자 함수
```

> 우변의 생성자 함수의 prototype에 바인딩된 객체가 `좌변의 객체의 프로토타입 체인 상에 존재`하면 `true`로 평가, 그렇지 않으면 `false`로 평가

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

console.log(me instanceof Person); // true
```

- Person.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 `true`로 평가
  - 프로토타입이 교체되어 연결이 파괴된 경우에는 `false`로 평가

> 생성자 함수의 `prototype에 바인딩된 객체`가 프로토타입 체인 상에 `존재`하는지 확인

<img alt="javascript_prototype_method_4" src="/static/images/javascript_prototype_method_4.png" width="750"/>

- `me instanceof Person`
  - me 객체의 프로토타입 체인 상에 `Person.prototype에 바인딩된 객체의 존재 확인`
- `me instanceof Object`
  - me 객체의 프로토타입 체인 상에 `Object.prototype에 바인딩된 객체의 존재 확인`
- `instanceof` 함수 구현체

    ```javascript
    function isInstanceOf(instance, constructor) {
      // 1. 프로토타입 취득
      const prototype = Object.getPrototypeOf(instance);
      // 2. 재귀 탈출 조건 설정
      // prototype이 null이면 프로토타입 체인의 종점에 다다름
      if(prototype === null) return false;
    
      // 프로토타입이 생성자 함수의 prototype 프로퍼티에 바인딩된 객체라면 true 반환
      // 그렇지 않다면 재귀 호출로 프로토타입 체인 상의 상위 프로토타입으로 이동하여 확인
      return prototype === constructor.prototype || isInstanceOf(prototype, constructor);
    }
    
    console.log(isInstanceof(me, Person)); // true
    console.log(isInstanceof(me, Object)); // true
    console.log(isInstanceof(me, Array)); // false
    ```

## 직접 상속

### `Object.create`에 의한 직접 상속

- 명시적으로 프로토타입을 지정하여 새로운 객체를 생성하는 방법 제공
  - `첫 번째 매개변수` - 생성할 객체의 프로토타입으로 지정할 객체 전달
  - `두 번째 매개변수`(선택) - 생성할 객체의 프로퍼티 키와 프로퍼티 디스크립터 객체로 이뤄진 객체 전달

```text
/**
 * 지정된 프로토타입 및 프로퍼티를 갖는 새로운 객체를 생성하여 반환
 * @param {Object} prototype - 생성할 객체의 프로토타입으로 지정할 객체
 * @param {Object} [propertiesObject] - 생성할 객체의 프로퍼티를 갖는 객체
 * @returns {Object} 지정된 프로토타입 및 프로퍼티를 갖는 새로운 객체
 */
Object.create(prototype[, propertiesObject])
```

- `new 연산자 없이도` 객체 생성 가능
- `프로토타입을 지정`하면서 객체 생성 가능
- 객체 리터럴에 의해 생성된 `객체 상속 가능`

### 객체 리터럴 내부에서 __proto__에 의한 직접 상속

- ES6에서는 `객체 리터럴 내부`에서 접근자 프로퍼티를 사용하여 `직접 상속을 구현`할 수 있음

```javascript
const myProto = {x: 10};

// 객체 리터럴에 의해 객체를 생성하면서 프로토타입을 지정하여 직접 상속받을 수 있음
const obj = {
  y: 20,
  // obj -> myProto -> Object.prototype -> null
  __proto__: myProto
};

console.log(obj.x, obj.y); // 10, 20
console.log(Object.getPrototypeOf(obj) === myProto); // true
```

## 정적 프로퍼티 / 메서드

- 생성자 함수로 인스턴스를 생성하지 않아도 참조/호출할 수 있는 프로퍼티/메서드

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

// 정적 프로퍼티
Person.staticProp = `static prop`;

// 정적 메서드
Person.staticMethod = function () {
  console.log('staticMethod');
};

const me = new Person('Lee');

// 생성자 함수에 추가한 정적 프로퍼티/메서드는 생성자 함수로 참조/호출함
Person.staticMethod(); // staticMethod

// 정적 프로퍼티/메서드는 생성자 함수가 생성한 인스턴스로 참조, 호출할 수 없음
// 인스턴스로 참조, 호출할 수 있는 프로퍼티, 메서드는 프로토타입 체인 상에 존재해야  함
me.staticMethod(); // TypeError: me.staticMethod is not a function
```

<img alt="javascript_prototype_method_5" src="/static/images/javascript_prototype_method_5.png" width="800"/>

- `생성자 함수가 생성한 인스턴스`는 자신의 프로토타입 체인에 속한 `객체의 프로퍼티/메서드에 접근할 수 있음`
- `정적 프로퍼티/메서드`는 인스턴스의 프로토타입 체인에 속한 객체의 프로퍼티/메서드가 아니므로 `인스턴스로 접근할 수 없음`
- 참고
  - MDN 문서에는 프로토타입 프로퍼티/메서드를 표기할 때 prototype을 `#`으로 표기하는 경우도 있음

## 프로퍼티 존재 확인

### in 연산자

- 객체 내에 특정 프로퍼티가 존재하는지 여부 확인

```javascript
/**
 * key: 프로퍼티 키를 나타내는 문자열
 * Object: 객체로 평가되는 표현식
 */
key in object

// ===============================================================

const person = {
  name: 'Lee',
  address: 'Seoul';
};

console.log('name' in person); // true
console.log('address' in person); // true
console.log('age' in person); // false
```

- in 연산자는 확인 대상 객체의 프로퍼티뿐만 아니라 확인 대상 객체가 `상속받은 모든 프로토타입의 프로퍼티를 확인함`

> person 객체가 속한 프로토타입 체인 상에 존재하는 `모든 프로토타입에서 toString 프로퍼티도 존재`하므로, 코드상에서 person 객체에 속하지 않아도 `Object.prototype의 메서드`
> 이므로 `true`가 반환됨

- `Reflect.has` 메서드
  - ES6에서 도입된 in 연산자와 동일하게 동작하는 메서드

```javascript
const person = {name: 'Lee'};

console.log(Reflect.has(person, 'name')); // true
console.log(Reflect.has(person, 'toString')); // true
```

### Object.prototype.hasOwnProperty 메서드

- 객체의 특정 프로퍼티가 존재하는지 확인할 수 있음

```javascript
console.log(person.hasOwnProperty('name')); // true
console.log(person.hasOwnProperty('age')); // false
```

- in 연산자와 달리, 인수로 전달받은 프로퍼티 키가 객체 `고유의 프로퍼티 키`인 경우에만 `true` 반환

```javascript
console.log(person.hasOwnProperty('toString')); // false
```

## 프로퍼티 열거

### for…in 문

- 객체의 모든 프로퍼티를 순회하며 열거하는 구문

```javascript
// for(변수선언문 in 객체) {...}

const person = {
  name: 'Lee',
  address: 'Seoul';
};

for (const key in person) {
  console.log(key + ': ' + person[key]);
}
// name: Lee
// address: Seoul
```

- 객체의 `프로퍼티 개수만큼 순회`하여 선언한 변수에 `프로퍼티 키를 할당`함
- `열거할 때 순서를 보장하지 않지만` 대부분의 모던 브라우저는 순서를 보장함
- 상속받은 프로토타입의 `프로퍼티까지 열거`할 수 있음
  - toString과 같은 Object.prototype의 프로퍼티는 `[[Enumerable]]`의 값이 `false`이므로 열거되지 않음

  ```javascript
  console.log(Object.getOwnPropertyDescriptor(Object.prototype, 'toString'));
  // {value: f, writable: true, enumerable: false, configurable: true}
  ```

> for … in 문은 객체의 프로토타입 체인 상에 존재하는 모든 프로토타입의 프로퍼티 중에서 프로퍼티 어트리뷰트 `[[Enumerable]]`의 값이 `true`인 `프로퍼티를 순회하며 열거함`
>

- 일반적으로 `for문`, `for … of`, `Array.prototype.forEach` 메서드를 사용할 것을 `권장`

### Object.keys/values/entries 메서드

- 객체의 자신의 고유 프로퍼티만 열거하기 위한 메서드
- `Object.keys`
  - 객체 자신의 열거 가능한 `프로퍼티 키를 배열로 반환`

    ```javascript
    const person = {
    	name: 'Lee',
    	address: 'Seoul',
    	__proto__: {age: 20}
    };
    
    console.log(Object.keys(person)); // ["name", "address"]
    ```


- `Object.values`
  - ES8에 도입된 메서드로서, 객체 자신의 열거 가능한 `프로퍼티 값을 배열로 반환`

    ```javascript
    console.log(Object.values(person)); // ["Lee", "Seoul"]
    ```

- `Object.entries`
  - ES8에 도입된 메서드로서, 객체 자신의 열거 가능한 `프로퍼티 키, 값의 쌍`의 배열을 `배열에 담아 반환`

    ```javascript
    console.log(Object.entries(person)); // [["name", "Lee"], ["address", "Seoul"]]
    
    Object.entries(person).forEach(([key, value]) => console.log(key, value));
    /*
    name Lee
    address Seoul
    */
    ```

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 290 ~ 312p