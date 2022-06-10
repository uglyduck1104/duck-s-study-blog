---
title: Symbol Type
date: '2022-06-10'
tags: ['javascript']
draft: false
summary: ES6에서 도입된 7번째 데이터 타입으로 변경 불가능한 원시 타입의 값
layout: PostSimple
authors: ['default']
---

# Symbol Type

## 심벌

- ECMAScript 표준화 후 6개의 타입이 있음
  - `문자열`
  - `숫자`
  - `불리언`
  - `undefined`
  - `null`
  - `객체 타입`
- ES6에서 도입된 7번째 데이터 타입으로 `변경 불가능한 원시 타입의 값`
  - 주로 유일무이한 `프로퍼티 키`를 만들기 위해 사용

## 심벌 값의 생성

### Symbol 함수

- `Symbol 함수`를 호출하여 생성함
  - 외부로 노출되지 않아 확인할수 없고 다른 값과 절대 중복되지 않는 유일무이한 값

    ```jsx
    // Symbol 함수를 호출하여 유일 무이한 심벌 값을 생성함
    const mySymbol = Symbol();
    console.log(typeof mySymbol); // symbol
    
    // 심벌 값은 외부로 노출되지 않아 확인할 수 없음
    console.log(mySymbol); // Symbol();
    ```

  - `new 연산자`를 함께 호출하지 않음
- `디버깅 용도`로 사용할 수 있는 문자열 생성

    ```jsx
    // 심벌 값에 대한 설명이 같더라도 유일무이한 심벌 값을 생성함
    const mySymbol1 = Symbol('mySymbol');
    const mySymbol2 = Symbol('mySymbol');
    
    console.og(mySymbol1 === mySymbol2); // false
    ```

- 심벌 값도 문자열, 숫자, 불리언과 같이 객체처럼 접근하면 `암묵적`으로 `래퍼 객체를 생성함`

    ```jsx
    const mySymbol = Symbol('mySymbol');
    
    // 심벌도 래퍼 객체를 생성함
    console.log(mySymbol.description); // mySymbol
    console.log(mySymbol.toString()); // Symbol(mySymbol)
    ```

  - 암묵적으로 문자열이나 숫자 타입으로는 변환되지 않음
  - 단 `불리언 타입`으로는 암묵적으로 타입 변환됨

      ```jsx
      const mySymbol = Symbol();
      
      // 불리언 타입으로는 암묵적으로 타입 변환됨
      console.log(!!mySymbol); // true
      
      // if 문 등에서 존재 확인이 가능함
      if (mySymbol) console.log('mySymbol is not empty.');
      ```

### Symbol.for / Symbol.keyFor 메서드

- `Symbol.for`
  - 인수로 전달받은 `문자열을 키로 사용`하여 키와 심벌 값의 쌍들이 저장되어 있는 전역 심벌 레지스트리에서 `해당 키와 일치하는 심벌 값을 검색함`
    - 검색 성공
      - 새로운 심벌 값을 생성하지 않고 `검색된 심벌 값을 반환`
    - 검색 실패
      - 새로운 심벌 값 생성
      - Symbol.for 메서드의 인수로 전달된 키로 `전역 심벌 레지스트리에 저장`한 후, `생성된 심벌 값을 반환`

    ```jsx
    // 전역 심벌 레지스트리에 mySymbol이라는 키로 저장된 심벌 값이 없으면 새로운 심벌 값을 생성
    const s1 = Symbol.for('mySymbol');
    // 전역 심벌 레지스트리에 mySymbol이라는 키로 저장된 심벌 값이 있으면 해당 심벌 값을 반환
    const s2 = Symbol.for('mySymbol');
    
    console.log(s1 === s2); // true
    ```

  - 애플리케이션 전역에서 중복되지 않는 유일무이한 상수인 심벌 값을 `단 하나만 생성`하여 `전역 심벌 레지스트리를 통해 공유`할 수 있음
- `Symbol.keyFor`
  - 전역 심벌 레지스트리에 저장된 심벌 값의 키를 추출할 수 있음

      ```jsx
      // 전역 심벌 레지스트리에 mySymbol이라는 키로 저장된 심벌 값이 없으면 새로운 심벌 값을 생성
      const s1 = Symbol.for('mySymbol');
      // 전역 심벌 레지스트리에 저장된 심벌 값의 키를 추출
      Symbol.keyFor(s1); // -> mySymbol
      
      // Symbol 함수를 호출하여 생성한 심벌 값은 전역 심벌 레지스트리에 등록되어 관리되지 않음
      const s2 = Symbol('foo');
      // 전역 심벌 레지스트리에 저장된 심벌 값의 키 추출
      Symbol.keyFor(2); // -> undefined
      ```

## 심벌과 상수

### 상수 정의

```jsx
// 위, 아래, 왼쪽, 오른쪽을 나타내는 상수 정의
const Direction = {
  UP: 1,
  DOWN: 2,
  LEFT: 3,
  RIGHT: 4
};

// 변수에 상수를 할당
const myDirection = Direction.UP;

if (myDirection === Direction.UP) {
  console.log('You are going Up.');
}
```

- `문제점`
  - 상수 값 1, 2, 3, 4가 변경될 수 있으며, 다른 변수 값과 중복될 수 있음

### 심벌 정의

```jsx
// 중복될 가능성이 없는 심벌 값으로 상수 값을 생성
const Direction = {
  UP: Symbol('up'),
  DOWN: Symbol('down'),
  LEFT: Symbol('left'),
  RIGHT: Symbol('right'),
};

const myDirection = Direction.UP;

if (myDirection === Direction.UP) {
  console.log('You are going UP.');
};
```

## 심벌과 프로퍼티 키

- 객체의 프로퍼티 키는 빈 문자열을 포함하는 모든 문자열 또는 심벌 값으로 만들 수 있으며, 동적으로 생성할 수 있음

    ```jsx
    const obj = {
    	// 심벌 값으로 프로퍼티 키를 생성
    	[Symbol.for('mySymbol')]: 1
    };
    
    obj[Symbol.for('mySymbol')]; // -> 1
    ```

  - 심벌 값은 유일무이한 값이므로 프로퍼티 키를 만들면 `다른 프로퍼티 키와 절대 충돌하지 않음`

## Well-known Symbol

- 자바스크립트가 기본 제공하는 빌트인 심벌 값(`Well-known Symbol`)
  - 빌트인 심벌 값은 Symbol 함수의 프로퍼티에 할당되어 있음

> `Array`, `String`, `Map`, `Set`, `TypedArray`, `arguments`, `NodeList`, `HTMLCollection`과
> 같이 `for…of문으로 순회 가능한 빌트인 이터러블`은 Well-known Symbol인 `Symbol.iterator를 키로 갖는 메서드를 가지며`, Symbol.iterable 메서드를 호출하면 이터레이트를
> 반환하도록 ECMAScript 사양에 규정되어 있음

```jsx
const iterable = {
  // Symbol.iterator 메서드를 구현하여 이터러블 프로토콜 준수
  [Symbol.iterator]() {
    let cur = 1;
    const max = 5;
    // Symbol.iterator 메서드는 next 메서드를 소유한 이터레이터를 반환
    return {
      next() {
        return {value: cur++, done: cur > max + 1};
      }
    }

  }
};

for (const num of iterable) {
  console.log(num); // 1, 2, 3, 4, 5
}
```

- 심벌은 중복되지 않는 상수 값을 생성하는 것은 물론 기존에 작성된 코드에 영향을 주지 않고 새로운 프로퍼티를 추가하기 위해, 즉 `하위 호환성을 보장하기 위해 도입됨`

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 605 ~ 613p
