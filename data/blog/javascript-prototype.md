---
title: 프로토타입
date: '2022-04-03'
tags: ['javascript']
draft: false
summary: 프로토타입 기반 객체지향 프로그래밍 언어
images: []
layout: PostSimple
canonicalUrl:
authors: ['default']
---

# 프로토타입 객체

- 자바스크립트는 프로토타입 기반 객체지향 프로그래밍 언어
- 객체지향 프로그래밍 언어
  - 객체 생성 이전에 클래스 정의 → 인스턴스화
- 프로토타입 기반 객체지향 프로그래밍 언어
  - Class-less해도 객체 생성 가능
- 객체 → 모든 객체의 부모
- 생성자 함수에 의해 생성된 각각의 객체에 공유 프로퍼티를 제공

  ```jsx
  var student = {
    name: 'Lee',
    score: 90,
  }

  // student에는 hasOwnProperty 메소드가 없지만 아래 구문은 동작한다.
  console.log(student.hasOwnProperty('name')) // true

  console.dir(student)
  ```

![student class](/static/images/prototype_1.png)

Google chrome에서 student 객체 출력 결과

- 자바스크립트의 모든 객체는 [[Prototype]]이라는 `인터널 슬롯`을 가짐
  - 값 → null 또는 객체이며 상속을 구현하는 데 사용
  - 데이터 프로퍼티 → get 액세스를 위해 상속되어 자식 객체의 프로퍼티처럼 사용할 수 있음

# [[Prototype]] vs prototype 프로퍼티

- 모든 객체는 자신의 프로토타입 객체를 가리키는 [[Prototype]] 인터널 슬록을 갖으며 상속을 위해 사용
- 함수 객체는 일반 객체와 달리 prototype 프로퍼티도 소유하게 됨

> prototype 프로퍼티는 프로토타입 객체를 가리키는 [[Prototype]] 인터널 슬롯은 다름. prototype 프로퍼티와 [[Prototype]]은 모두 프로토타입 객체를 가리키지만 관점의 차이가 있음

```jsx
function Person(name) {
  this.name = name
}

var foo = new Person('Lee')

console.dir(Person) // prototype 프로퍼티가 있다.
console.dir(foo) // prototype 프로퍼티가 없다.
```

- [[Prototype]]

  - 함수를 포함한 모든 객체가 가지고 있는 인터널 슬롯임
  - 객체의 입장에서 자신의 부모 역할을 하는 프로토타입 객체를 가리키며 함수 객체의 경우 `Function.prototype`를 가리킴

    ```jsx
    console.log(Person.__proto__ === Function.prototype)
    ```

- prototype 프로퍼티

  - 함수 객체만 가지고 있는 프로퍼티
  - 함수 객체가 생성자로 사용될 때 이 함수를 통해 생성될 객체의 부모 역할을 하는 객체를 가리킴

    ```jsx
    console.log(Person.prototype === foo.__proto__)
    ```

# constructor 프로퍼티

- constructor 프로퍼티는 객체의 입장에서 자신을 생성한 객체를 가리킴

  ```jsx
  function Person(name) {
    this.name = name
  }

  var foo = new Person('Lee')

  // Person() 생성자 함수에 의해 생성된 객체를 생성한 객체는 Person() 생성자 함수이다.
  console.log(Person.prototype.constructor === Person)

  // foo 객체를 생성한 객체는 Person() 생성자 함수이다.
  console.log(foo.constructor === Person)

  // Person() 생성자 함수를 생성한 객체는 Function() 생성자 함수이다.
  console.log(Person.constructor === Function)
  ```

- foo 객체를 생성한 객체는 Person() 생성자 함수
  - 이때 foo 객체 입장에서 자신을 생성한 객체는 Person() 생성자 함수이며, foo 객체의 프로토타입 객체는 Person.prototype임.
  - 따라서 프로토타입 객체 Person.prototype의 constructor 프로퍼티는 Person() 생성자 함수를 가리킨다.

# Prototype chain

- 특정 객체의 프로퍼티나 메소드에 접근하려고 할 때 해당 객체에 접근하려는 프로퍼티 또는 메소드가 없다면 [[Prototype]]이 가리키는 링크를 따라 자신의 부모 역할을 하는 프로토타입 객체의 프로퍼티나 메소드를 차례대로 검색 하는 것

  ```jsx
  var student = {
    name: 'Lee',
    score: 90,
  }

  // Object.prototype.hasOwnProperty()
  console.log(student.hasOwnProperty('name')) // true
  ```

- student 객체는 hasOwnProperty 메소드를 가지고 있지 않으므로 에러가 발생하여야 하나 정상적으로 결과가 출력됨

  - 이는 student 객체의 [[Prototype]]이 가리키는 링크를 따라가서 student 객체의 부모 역할을 하는 프로토타입 객체(Object.prototype)의 메소드 hasOwnProperty를 호출하였기 때문에 가능함

  ```jsx
  var student = {
    name: 'Lee',
    score: 90,
  }
  console.dir(student)
  console.log(student.hasOwnProperty('name')) // true
  console.log(student.__proto__ === Object.prototype) // true
  console.log(Object.prototype.hasOwnProperty('hasOwnProperty')) // true
  ```

## 객체 리터럴 방식으로 생성된 객체의 프로토타입 체인

- 객체 생성 방법

  - 객체 리터럴 → 내장 함수(Built-in)인 Object() 생성자 함수로 객체를 생성하는 것을 단순화
  - 생성자 함수 → 함수 객체인 Object() 생성자 함수는 일반 객체와 달리 prototype 프로퍼티가 있음
  - Object() 생성자 함수

  ```jsx
  var person = {
    name: 'Lee',
    gender: 'male',
    sayHello: function () {
      console.log('Hi! my name is ' + this.name)
    },
  }

  console.dir(person)

  console.log(person.__proto__ === Object.prototype) // ① true
  console.log(Object.prototype.constructor === Object) // ② true
  console.log(Object.__proto__ === Function.prototype) // ③ true
  console.log(Function.prototype.__proto__ === Object.prototype) // ④ true
  ```

  > 객체 리터럴을 사용하여 객체를 생성한 경우, 그 객체의 프로토타입 객체는 Object.prototype임

## 생성자 함수로 생성된 객체의 프로토타입 체인

- 생성자 함수 정의하는 방식
  - 함수 선언식
  - 함수 표현식
  - Function() 생성자 함수
- 함수선언식, 함수표현식 모두 함수 리터럴 방식을 사용

> 즉, 3가지 함수 정의 방식은 결국 Function() 생성자 함수를 통해 함수 객체를 생성함. 따라서 `어떠한 방식으로 함수 객체를 생성하여도 모든 함수 객체의 prototype 객체는 Function.prototype`이고 생성자 함수도 함수 객체이므로 생성자 함수의 prototype 객체는 Function.prototype임

```jsx
function Person(name, gender) {
  this.name = name
  this.gender = gender
  this.sayHello = function () {
    console.log('Hi! my name is ' + this.name)
  }
}

var foo = new Person('Lee', 'male')

console.dir(Person)
console.dir(foo)

console.log(foo.__proto__ === Person.prototype) // ① true
console.log(Person.prototype.__proto__ === Object.prototype) // ② true
console.log(Person.prototype.constructor === Person) // ③ true
console.log(Person.__proto__ === Function.prototype) // ④ true
console.log(Function.prototype.__proto__ === Object.prototype) // ⑤ true
```

- foo 객체의 프로토타입 객체 Person.prototype 객체와 Person() 생성자 함수의 프로토타입 객체인 Function.prototype의 프로토타입 객체는 Object.prototype 객체임
  - 객체 리터럴 방식이나 생성자 함수 방식이나 결국은 모든 객체의 부모 객체인 Object.prototype 객체에서 프로토타입 체인이 끝나기 때문
  - Object.prototype 객체를 프로토타입 체인의 종점(End of prototype chain)이라고 함

# 프로토타입 객체의 확장

- 프로퍼티 추가/삭제

  ```jsx
  function Person(name) {
    this.name = name
  }

  var foo = new Person('Lee')

  Person.prototype.sayHello = function () {
    console.log('Hi! my name is ' + this.name)
  }

  foo.sayHello()
  ```

- 생성자 함수 Person은 프로토타입 객체 Person.prototype와 prototype 프로퍼티에 의해 바인딩됨
- 생성자 함수 Person에 의해 생성된 모든 객체는 프로토타입 체인에 의해 부모객체인 Person.prototype의 메소드를 사용할 수 있게 됨

# 원시 타입이 확장

- 원시 타입을 제외한 모든 것은 객체임
- 아래 예제에서는 원시 타입인 문자열이 객체와 유사하게 동작함

```jsx
var str = 'test'
console.log(typeof str) // string
console.log(str.constructor === String) // true
console.dir(str) // test

var strObj = new String('test')
console.log(typeof strObj) // object
console.log(strObj.constructor === String) // true
console.dir(strObj)
// {0: "t", 1: "e", 2: "s", 3: "t", length: 4, __proto__: String, [[PrimitiveValue]]: "test" }

console.log(str.toUpperCase()) // TEST
console.log(strObj.toUpperCase()) // TEST
```

- 원시 타입 문자열과 String() 생성자 함수로 생성한 문자열 객체의 타입은 분명히 `다름`
- 객체가 아니므로 `프로퍼티나 메소드를 가질 수 없음`

- 원시 타입으로 프로퍼티나 메소드를 호출할 때 `원시 타입과 연관된 객체로 일시적으로 변환되어 프로토타입 객체를 공유함`

```jsx
var str = 'test'

// 에러가 발생하지 않는다.
str.myMethod = function () {
  console.log('str.myMethod')
}

str.myMethod() // Uncaught TypeError: str.myMethod is not a function
```

- 원시 타입은 객체가 아니므로 프로퍼티나 메소드를 직접 추가할 수 없다.

```jsx
var str = 'test'

String.prototype.myMethod = function () {
  return 'myMethod'
}

console.log(str.myMethod()) // myMethod
console.log('string'.myMethod()) // myMethod
console.dir(String.prototype)
```

- String 객체의 프로토타입 객체 String.prototype에 메소드를 추가하면 원시 타입, 객체 모두 메소드를 사용할 수 있음

# 프로토타입 객체의 변경

- 객체를 생성할 때 프로토타입 결정

  - 다른 임의의 객체로 변경 가능

- 이것은 부모 객체인 `프로토타입을 동적으로 변경`할 수 있다는 것을 의미

> **프로토타입 객체 변경 시 주의할 점**

- 프로토타입 객체 변경 시점 이전에 생성된 객체
  - 기존 프로토타입 객체를 [[Prototype]]에 바인딩한다.
- 프로토타입 객체 변경 시점 이후에 생성된 객체
  - 변경된 프로토타입 객체를 [[Prototype]]에 바인딩한다.

```jsx
function Person(name) {
  this.name = name
}

var foo = new Person('Lee')

// 프로토타입 객체의 변경
Person.prototype = { gender: 'male' }

var bar = new Person('Kim')

console.log(foo.gender) // undefined
console.log(bar.gender) // 'male'

console.log(foo.constructor) // ① Person(name)
console.log(bar.constructor) // ② Object()
```

# 프로토타입 체인 동작 조건

- 객체의 프로퍼티를 참조하는 경우
  - 해당 객체에 프로퍼티가 없는 경우, 프로토타입 체인이 동작함
- 객체의 프로퍼티에 값을 할당하는 경우
  - 프로토타입 체인이 동작하지 않음

```jsx
Person.prototype.gender = 'male' // ①

var foo = new Person('Lee')
var bar = new Person('Kim')

console.log(foo.gender) // ① 'male'
console.log(bar.gender) // ① 'male'

// 1. foo 객체에 gender 프로퍼티가 없으면 프로퍼티 동적 추가
// 2. foo 객체에 gender 프로퍼티가 있으면 해당 프로퍼티에 값 할당
foo.gender = 'female' // ②

console.log(foo.gender) // ② 'female'
console.log(bar.gender) // ① 'male'
```

- foo 객체의 gender 프로퍼티에 값을 할당하면 프로토타입 체인이 발생하여 Person.prototype 객체의 gender 프로퍼티에 값을 할당하는 것이 아니라 foo 객체에 프로퍼티를 동적으로 추가함

> **Referenced**

- https://poiemaweb.com/js-prototype
