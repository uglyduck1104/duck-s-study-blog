---
title: 프로토타입 객체와 생성
date: '2022-05-23'
tags: ['javascript']
draft: false
summary: 자바스크립트는 명령형, 함수형, 프로토타입 기반 객체 지향 프로그래밍이 가능한 `멀티 패러다임 프로그래밍 언어`
layout: PostSimple
authors: ['default']
---

# 프로토타입 객체와 생성

- 자바스크립트는 `멀티 패러다임 프로그래밍 언어`로 정의할 수 있음
  - 명령형
  - 함수형
  - 프로토타입 기반 `객체 지향 프로그래밍 언어`
    - C++, JAVA와 같은 크래스 기반 객체지향 프로그래밍 언어의 특징인 접근 지시자 키워드가 없어서 오해하는 경우가 있음(public, private, protected 등)
    - 클래스 기반 객체지향 언어보다 더 강력한 객체지향 프로그래밍 능력을 지니고 있음
    - 자바스크립트를 이루고 있는 것은 거의 `모든 것`이 객체
      - 원시 타입의 값을 제외한 `나머지 값들은 모두 객체`

## OOP

- `객체의 집합`으로 프로그램을 표현하려는 프로그래밍의 패러다임
- `상태`와 `행위`를 가진 객체간의 상호작용(메세지)을 통해 `역할`, `책임`, `협력`하는 철학적 사고를 프로그래밍에 접목한 것

## 상속과 프로토 타입

- `상속`
  - 객체의 프로퍼티 또는 메서드를 다른 객체가 상속받아 사용할 수 있음
  - 상속을 구현하면 불필요한 중복을 제거할 수 있고, 기존의 코드를 `적극 재사용`할 수 있음

### 상속 미구현(동일한 메서드 중복 소유)

```javascript
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
  this.getArea = function () {
    return Math.PI * this.radius ** 2;
  }
}

const circle1 = new Circle(1);
const circle2 = new Circle(2);

console.log(circle1.getArea === circle2.getArea); // false

console.log(circle1.getArea()); // 3.141592653589793
console.log(circle2.getArea()); // 12.566370614359172
```

> Key Point🔑 - `인스턴스의 생성`을 봐라!\
> Circle 생성자 함수는 인스턴스를 생성할 때마다 동일한 동작을하는 getArea 메서드를 `중복 생성`하고 `모든 인스턴스가 중복 소유`함

<img alt="javascript_prototype_object1" src="/static/images/javascript_prototype_object1.png" width="700"/>

- 인스턴스가 동일한 메서드를 중복 소유한다? `불필요한 메모리가 낭비됨`
- 인스턴스를 생성할 때마다 메서드도 같이 생성된다? `서비스 퍼포먼스에도 악영향`

### 상속 구현(`프로토타입` 기반 불필요한 중복 제거)

```javascript
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
  /*
  this.getArea = function() {
    return Math.PI * this.radius ** 2;
  }
  */
}

Circle.protype.getArea = function () {
  return Math.PI * this.radius ** 2;
};

const circle1 = new Circle(1);
const circle2 = new Circle(2);

console.log(circle1.getArea === circle2.getArea); // true

console.log(circle1.getArea()); // 3.141592653589793
console.log(circle2.getArea()); // 12.566370614359172
```

- Circle 생성자 함수가 생성한 모든 인스턴스가 `getArea` 메서드를 `공유`할 수 있도록 함
  - getArea 메서드는 Circle 생성자 함수의 `prototype 프로퍼티에 바인딩`되어 있음
  - Circle 생성자 함수가 생성하는 모든 인스턴스는 `하나의 getArea 메서드를 공유`함
    <img alt="javascript_prototype_object2" src="/static/images/javascript_prototype_object2.png" width="700"/>

> 👉 상속은 `코드의 재사용 관점`에서 매우 유용함\
> 사용할 프로퍼티나 메서드를 프로토타입에 미리 구현해 두면 생성자 함수가 생성할 모든 인스턴스는 별도의 구현없이 상위 객체인 `프로토타입의 자산을 공유`하여 사용할 수 있음

## 프로토타입 객체

- 객체 간 `상속`을 구현하기 위해 사용
- 모든 객체는 `[[Prototype]]`이라는 `내부 슬롯`을 가짐
  - `프로토타입의 참조`
  - [[Prototype]] 내부 슬롯에는 `직접 접근할 수 없음`
- `객체 생성 방식`에 의해 결정
  <img alt="javascript_prototype_object3" src="/static/images/javascript_prototype_object3.png" width="700"/>
- `__proto__` 접근자 프로퍼티를 통해 [[Prototype]] 내부 슬롯이 가리키는 프로토타입에 `간접적으로 접근`할 수 있음

### **`__proto__` 접근자 프로퍼티**

- 모든 객체는 `__proto__` 접근자 프로퍼티를 통해 자신의 프로토타입에 간접적으로 접근할 수 있음
  - `접근자 프로퍼티`를 통해 프로토타입 접근
    - 내부적으로 getter 함수인 `[[Get]]`이 호출됨
    - 새로운 프로토타입을 할당하면 setter 함수인 `[[Set]]`이 호출됨
- 객체가 직접 소유하는 프로퍼티가 아니라, `Object.prototype`의 프로퍼티
- 모든 객체는 상속을 통해 `Object.prototype.__proto__` 접근자 프로퍼티를 사용할 수 있음

```javascript
const person = {name: 'Lee'};

// person 객체는 __proto__ 프로퍼티를 소유하지 않음
console.log(person.hasOwnProperty('__proto__')); // false

// __proto__ 프로퍼티는 모든 객체의 프로토타입 객체인 Object.prototype의 접근자 프로퍼티
console.log(Object.getOwnPropertyDescriptor(Object.prototype, '__proto__'));

// 모든 객체는 Object.prototype의 접근자 프로퍼티 __proto__를 상속받아 사용할 수 있음
console.log({}.__proto__ === Object.prototype); // true
```

### `Object.prototype`

- 모든 객체는 프로토타입의 계층 구조인 `프로토타입 체인`에 묶여 있음
- 자바스크립트 엔진은 객체의 프로퍼티에 접근하려고 할 때 해당 객체에 접근하려는 프로퍼티가 없다면 `__proto__` 접근자 프로퍼티가 가리키는 참조를 따라 자신의 부모 역할을 하는 프로토타입의 프로퍼티를
  순차적으로 검색
- 프로토타입 체인의 `최상위 객체`는 `Object.prototype`임

### __proto__ 접근자 프로퍼티를 통해 프로토타입에 접근하는 이유

- 상호 참조에 의해 `프로토타입 체인이 생성되는 것을 방지`하기 위해

```javascript
const parent = {};
const child = {};

// child의 프로토타입을 parent로 지정
child.__proto__ = parent;
// parent의 프로토타입을 child로 지정
parent.__proto__ = child; // TypeError: Cyclic __proto__ value
```

<img alt="javascript_prototype_object4" src="/static/images/javascript_prototype_object4.png" width="300"/>

- 해당 예제를 통해 서로가 자신의 프로토 타입이 되는 `비정상적인 프로토타입 체인 생성 방지`를 위해 접근자 에러를 발생
- 프로토타입 체인은 `단방향 링크드 리스트`로 구현되어야 함
  - 프로퍼티 검색 방향이 `한쪽 방향`으로만 흘러야 `프로토타입 체인 종점`으로 도달할 수 있음

### `__proto__` 접근자 프로퍼티의 사용

- `__proto__` 접근자 프로퍼티를 `코드 내에서 직접 사용하는 것은 지양`해야 함
  - 모든 객체가 해당 접근자 프로퍼티를 사용할 수 있는 것은 아님
- `Object.getPrototypeOf` 메서드 사용하기

```javascript
const obj = {};
const parent = {x: 1};

// obj 객체의 프로토타입을 취득
Object.getPrototypeOf(obj); // obj.__proto__;
// obj 객체의 프로토타입을 교체
Object.setPrototypeOf(obj, parent); // obj.__proto__ = parent;

console.log(obj.x); // 1
```

### 함수 객체의 prototype 프로퍼티

- 함수 객체만이 소유하는 prototype 프로퍼티는 `생성자 함수`가 `생성할 인스턴스의 프로토타입`을 가리킴

```javascript
// 함수 객체는 prototype 프로퍼티를 소유
(function () {
}).hasOwnProperty('prototype'); // -> true

// 일반 객체는 prototype 프로퍼티를 소유하지 않음
({}).hasOwnProperty('prototype'); // -> false
```

- 생성자 함수로서 호출할 수 없는 함수(`non-constuctor`)인 `화살표 함수`와 `ES6 메서드 축약 표현`으로 정의한 메서드는 **prototype 프로퍼티를 소유하지 않으며 프로토타입도 생성하지
  않음**
- 모든 객체가 가지고 있는 `__proto__ 접근자 프로퍼티`와 함수 객체만이 가지고 있는 `prototype 프로퍼티`는 결국 동일한 프로토타입을 가리킴

| 구분                   | 소유          | 값         | 사용 주체  | 사용 목적                                       |
|----------------------|-------------|-----------|--------|---------------------------------------------|
| `__proto__` 접근자 프로퍼티 | 모든 객체       | 프로토타입의 참조 | 모든 객체  | 객체가 자신의 프로토타입 접근 또는 교체하기 위해 사용              |
| prototype 프로퍼티       | constructor | 프로토타입의 참조 | 생성자 함수 | 생성자 함수가 자신이 생성할 객체(인스턴스)의 프로토타입을 할당하기 위해 사용 |

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

console.log(Person.prototype === me.__proto__); // true
```

> 객체의 __proto__ `접근자 프로퍼티`와 함수 객체의 `prototype 프로퍼티`는 `동일한 프로토타입을 가리킴`

### 프로토타입의 constructor 프로퍼티와 생성자 함수

- 모든 프로토타입은 `constructor 프로퍼티`를 갖음
- prototype 프로퍼티로 `자신을 참조`하고 있는 생성자 함수

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

// me 객체의 생성자 함수는 Person임
console.log(me.constructor === Person); // true
```

## `리터럴 표기법`에 의해 생성된 객체의 생성자 함수와 프로토타입

```javascript
// 객체 리터럴
const obj = {};

// 함수 리터럴
const add = function (a, b) {
  return a + b;
};

// 배열 리터럴
const arr = [1, 2, 3];

// 정규 표현식 리터럴
const regexp = /is/ig;
```

- 리터럴 표기법에 의해 생성된 객체의 경우 프로토타입의 `constructor 프로퍼티가 가리키는 생성자 함수가 반드시 객체를 생성한 생성자 함수라고 단정할 수 없음`
- ECMAScript 사양 의사 코드에 의하면 Object 생성자 함수 호출과 객체 리터럴의 평가는 `추상 연산을 호출하여 빈 객체를 생성하는 점은 동일`하나 `new.target`의 확인이나 `프로퍼티 추가`
  처리 방식은 다름
- 리터럴 표기법에 의해 생성된 객체는 `가상`적인 `생성자 함수`를 갖음

> 프로토타입과 생성자 함수는 단독으로 존재할 수 없고 언제나 쌍으로 존재함

| 리터럴 표기법    | 생성자 함수   | 프로토타입              |
|------------|----------|--------------------|
| 객체 리터럴     | Object   | Object.prototype   |
| 함수 리터럴     | Function | Function.prototype |
| 배열 리터럴     | Array    | Array.prototype    |
| 정규 표현식 리터럴 | RegExp   | RegExp.prototype   |

## 프로토타입의 생성 시점

- `생성자 함수가 생성되는 시점`에 더불어 생성됨

### `사용자 정의 생성자 함수`와 프로토타입 생성 시점

- 내부 메서드 [[Construct]]를 갖는 함수 객체, 일반 함수(함수 선언문, 함수 표현식)로 정의한 함수 객체는 `new 연산자`와 함께 생성자 함수로서 호출할 수 있음
  - 즉 constructor는 `함수 정의가 평가되어 함수 객체를 생성하는 시점`에 프로토타입도 더불어 생성됨

  ```javascript
  console.log(Person.prototype); // {constructor: f}
    
  // 생성자 함수
  function Person(name) {
    this.name = name;
  }
  ```

  ```javascript
  // 화살표 함수는 non-constructor
  const Person = name => {
    this.name = name;
  }
    
  // non-constructor는 프로토타입이 생성되지 않음
  console.log(Person.prototype); // undefined
  ```

  - 함수 선언문은 런타임 이전에 자바스크립트 엔진에 의해 먼저 실행됨
    - 따라서, Person 생성자 함수는 어떤 코드보다 먼저 평가되어 함수 객체가 됨(프로토타입 생성)

  <img alt="javascript_prototype_object5" src="/static/images/javascript_prototype_object5.png" width="700"/>

  > 사용자 정의 생성자 함수는 `자신이 평가`되어 `함수 객체로 생성되는 시점`에 프로토타입도 더불어 생성되며, 생성된 프로토타입의 프로토타입은 언제나 `Object.prototype`임

### `빌트인 생성자 함수`와 프로토타입 생성 시점

- Object, String, Number, Function, Array, RegExp, Date, Promis 등과 같은 빌트인 생성자 함수도 일반 함수와 마찬가지로 동일한 시점에 프로토타입이 생성됨
  - 생성된 프로토타입은 빌트인 생성자 함수의 prototype 프로퍼티에 바인딩됨

<img alt="javascript_prototype_object6" src="/static/images/javascript_prototype_object6.png" width="800"/>

- 이후 생성자 함수 또는 리터럴 표기법으로 객체를 생성하면 프로토타입은 생성된 객체 `[[Prototype]]` 내부 슬롯에 할당됨

## 객체 생성 방식과 프로토타입의 결정

### 객체의 생성 방식

- 객체 리터럴
- Object 생성자 함수
- 생성자 함수
- Object.create 메서드
- 클래스(ES6)

> 추상 연산 `OrdinaryObjectCreate`에 의해 생성되고 추상 연산에 전달되는 인수에 의해서 결정됨

### `객체 리터럴`에 의해 생성된 객체의 프로토타입

- 객체 리터럴을 평가하여 생성할 때 추상 연산을 호출
  - 추상 연산에 전달되는 프로토타입은 `Object.prototype`

> 객체 리터럴에 의해 생성된 obj 객체

```javascript
const obj = {x: 1};

// 객체 리터럴에 의해 생성된 obj 객체는 Object.prototype을 상속받음
console.log(obj.constructor === Object); // true
console.log(obj.hasOwnProperty('x')); // true
```

<img alt="javascript_prototype_object7" src="/static/images/javascript_prototype_object7.png" width="800"/>

- `Object.prototype`을 프로토타입으로 갖고 상속받음
  - `constructor 프로퍼티`와 `hasOwnProperty 메서드`를 자신의 자산인 것처럼 사용 가능

### `Object 생성자 함수`에 의해 생성된 객체의 프로퍼티

- 이 역시 객체 리터럴과 마찬가지로 추상 연산(OrdinaryObjectCreate)이 호출됨

```javascript
const obj = new Object();
obj.x = 1;

// Object 생성자 함수에 의해 생성된 obj 객체는 Object.prototype을 상속받음
console.log(obj.constructor === Obejct); // true
console.log(obj.hasOwnProperty('x')); // true
```

<img alt="javascript_prototype_object8" src="/static/images/javascript_prototype_object8.png" width="800"/>

> 객체 리터럴과 Object 생성자 함수에 의한 객체 생성 방식의 차이는 프로퍼티를 추가하는 방식에 있음\
`객체 리터럴` → 내부에 `프로퍼티 추가`\
`Object 생성자 함수 방식` → `빈 객체를 생성한 이후` 프로퍼티 추가

### `생성자 함수`에 의해 생성된 객체의 프로토타입

- 생성자 함수에 의해 생성되는 객체의 프로토타입은 생성자 함수의 `prototype 프로퍼티에 바인딩`되어 있는 객체

```javascript
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');
```

<img alt="javascript_prototype_object9" src="/static/images/javascript_prototype_object9.png" width="700"/>

- Object.protype은 다양한 빌트인 메서드를 갖고 있음
  - 사용자 정의 생성자 함수 Person과 더불어 생성된 프로토타입 Person.prototype의 프로퍼티는 `constructor` 뿐임
- `Person.prototype`에 프로퍼티 추가해서 `자식 객체가 상속 받게 구현`할 수 있음

```javascript
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

const me = new Person('Lee');
const you = new Person('Kim');

me.sayHello(); // Hi! My name is Lee
you.sayHello(); // Hi! My name is Kim
```

<img alt="javascript_prototype_object10" src="/static/images/javascript_prototype_object10.png" width="700"/>

## 프로토타입 체인

<img alt="javascript_prototype_object11" src="/static/images/javascript_prototype_object11.png" width="700"/>

- 자바스크립트가 객체 지향프로그래밍의 `상속`과 프로퍼티 검색을 구현하는 메커니즘
  - 자바스크립트는 객체의 프로퍼티에 접근하려 할 때 해당 객체에 접근하려는 프로퍼티가 없으면 [[Prototype]] 내부 슬롯의 참조에 따라 `자신의 부모 역할`을 하는
    프로토타입의 `프로퍼티를 순차적으로 검색`함

```javascript
// hasOwnProperty는 Object.prototype의 메서드
// me 객체는 체인을 따라 hasOwnProperty 메서드를 검색하여 사용
me.hasOwnProperty('name'); // -> true
 ```

### Execution Sequences

1. me 객체에서 hasOwnProperty 메서드 검색했을때 없으므로 [[Prototype]] 내부 슬롯에 바인딩되어 있는 프로토타입으로 이동하여 `메서드 검색`
2. Person.prototype에도 hasOwnProperty 메서드가 없으므로 체인을 따라 이동하여 `hasOwnProperty 메서드를 검색`함
3. Object.prototype에서 hasOwnProperty 메서드가 존재하면 자바스크립트 엔진은 `Object.prototype.hasOwnProperty`메서드를 호출하고 `this에는 me객체가 바인딩`

### Object.prototype

- 프로토타입 체인의 최상위는 언제나 `Object.prototype`
- 따라서 모든 객체는 Object.prototype을 상속받음
- Object.prototype을 프로토타입 체인의 종점(end of prototype chain)이라 함
  - 체인의 종점에서조차 프로퍼티를 검색할 수 없는 경우 undefined를 반환(`에러가 발생하지 않음`)

## 오버라이딩과 프로퍼티 섀도잉

```javascript
const Person = (function () {
  function Person(name) {
    this.name = name;
  }

  // 프로토타입 메서드
  Person.prototype.sayHello = function () {
    console.log(`Hi! My name is ${this.name}`);
  }

  // 생성자 함수를 반환
  return Person;
}());

const me = new Person('Lee');

// 인스턴스 메서드
me.sayHello = function () {
  console.log(`Hey! My name is ${this.name}`);
};

// 인스턴스 메서드가 호출됨. 프로토타입 메서드는 인스터스 메서드에 의해 가려짐
me.sayHello(); // Hey! My name is Lee
```

<img alt="javascript_prototype_object12" src="/static/images/javascript_prototype_object12.png" width="800"/>

- 프로토타입 프로퍼티와 같은 이름의 프로퍼티를 인스턴스의 추가하면 프로토타입 체인을 따라 검색하여 프로토타입 프로퍼티를 덮어쓰는 것이 아니라 인스턴스 프로퍼티로 추가함
- 인스턴스 메서드 sayHello는 `프로토타입 메서드 sayHello를 오버라이딩`함
- `프로퍼티 섀도잉`
  - 상속 관계에 의해 프로퍼티가 가려지는 현상
- 코드 상의 오버라이딩한 `인스턴스 객체`를 통해 프로토타입에 `get 액세스는 허용`되나 set 액세스는 허용되지 않으므로 `set 액세스를 하려면 프로토타입에 직접 접근`해야 함

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 259 ~ 290p