---
title: 이터러블(Iterable)
date: '2022-06-13'
tags: ['javascript']
draft: false
summary: ES6에서는 순회 가능한 자료구조를 이터레이션 프로토콜을 준수하는 이터러블로 통일하여 for…of 문, 스프레드 문법, 배열 디스트럭처링 할당의 대상으로 사용할 수 있도록 일원화함
layout: PostSimple
authors: ['default']
---

# 이터러블

## 이터레이션 프로토콜

### 등장 배경

- ES6에서 도입된 `이터레이션 프로토콜`은 순회 가능한 자료구조를 만들기 위해 ECMAScript 사양에 정의한 미리 약속한 규칙임
- ES6에서는 순회 가능한 자료구조를 이터레이션 프로토콜을 준수하는 이터러블로 통일하여 `for…of 문`, `스프레드 문법`, `배열 디스트럭처링` 할당의 대상으로 사용할 수 있도록 일원화함

### 이터러블 프로토콜

- Well-known Symbol인 `Symbol.iterator`를 프로퍼티 키로 사용한 메서드를 직접 구현하거나 프로토타입 체인을 통해 상속 받은 `Symbol.iterator` 메서드를 호출하면 이터레이터
  프로토콜을 준수한 `이터레이터`를 반환함
  - 이터러블 프로토콜을 준수한 `객체`를 `이터러블`이라고 함
  - `for...of` 문으로 `순회`할 수 있으며 `스프레드 문법`과 `배열 스트릭처링` 할당의 대상으로 사용할 수 있음

### 이터러블

- `Symbol.iterator`를 프로퍼티 키로 사용한 메서드를 직접 구현하거나 프로토타입 체인을 통해 상속받은 `객체`를 말함

    ```jsx
    const array = [1, 2, 3];
    
    // 배열은 Array.prototype의 Symbol.iterator 메서드를 상속받은 이터러블
    console.log(Symbol.iterator in array); // true
    
    // 이터러블인 배열은 for...of 문으로 순회 가능
    for(const item of array) {
    	console.log(item);
    }
    // 이터러블 배열은 스프레드 문법의 대상으로 사용할 수 있음
    console.log([...array]); // [1, 2, 3]
    
    // 이터러블인 배열은 배열 디스트럭처링 할당의 대상으로 사용할 수 있음
    const [a, ...rest] = array;
    console.log(a, rest); // 1, [2, 3]
    ```

  - `Symbol.iterator` 메서드를 직접 구현하지 않거나 상속받지 않은 일반 객체는 이터러블 프로토콜을 준수한 이터러블이 아님

### 이터레이터 프로토콜

- 이터러블의 `Symbol.iterator` 메서드를 호출하면 이터레이터 프로토콜을 준수한 `이터레이터`를 반환
- next 메서드를 소유하며 next 메서드를 호출하면 이터러블을 순회하며 `value`와 `done` 프로퍼티를 갖는 `이터레이터 리절트` 객체를 반환

  <img alt="javascript_iterable_1" src="/static/images/javascript_iterable_1.png" width="700"/>

  - 이터러블 프로토콜을 준수한 `객체`를 `이터레이터`라고 함

### 이터레이터

- `Symbol.iterator` 메서드를 호출하면 이터레이터 프로토콜을 준수한 이터레이터를 반환
  - 이터레이터는 `next` 메서드를 가짐

    ```jsx
    // 배열은 이터러블 프로토콜을 준수한 이터러블
    const array = [1, 2, 3];
    
    // Symbol.iterator 메서드는 이터레이터를 반환
    const iterator = array[Symbol.iterator]();
    
    // Symbol.iterator 메서드가 반환한 이터레이터는 next 메서드를 가짐
    console.log('next' in iterator); // true
    
    console.log(iterator.next()); // { value: 1, done: false }
    console.log(iterator.next()); // { value: 2, done: false }
    console.log(iterator.next()); // { value: 3, done: false }
    console.log(iterator.next()); // { value: undefined, done: true }
    ```

  - `next()`
    - 이터러블의 `각 요소를 순회`하기 위한 `포인터 역할`
    - 순회 결과를 나타내는 `이터레이터 리절트 객체`를 반환
  - `value` property
    - 현재 순회 중인 `이터러블의 값`
  - `done` property
    - 이터러블의 `순회 완료 여부`

## 빌트인 이터러블

| 빌트인 이터러블   | Symbol.iterator 메서드                                                               |
|------------|-----------------------------------------------------------------------------------|
| Array      | Array.prototype[Symbol.iterator]                                                  |
| String     | String.prototype[Symbol.iterator]                                                 |
| Map        | Map.prototype[Symbol.iterator]                                                    |
| Set        | Set.prototype[Symbol.iterator]                                                    |
| TypedArray | TypedArray.prototype[Symbol.iterator]                                             |
| arguments  | arguments[Symbol.iterator]                                                        |
| DOM 컬렉션    | NodeList.prototype[Symbol.iterator]<br/>HTMLCollection.prototype[Symbol.iterator] |

## for…of 문

### `for…of` vs `for…in`

- `for…of`

  ```
  for (변수선언문 of 이터러블) { ... }
  ```

  - 이터레이터의 next 메서드를 호출하여 이터러블을 순회하며, next 메서드가 반환한 이터레이터 리절트 객체의 value 프로퍼티 값을 `for…of` 문의 변수에 할당
  - 내부 동작 → `for문 표현`

    ```jsx
    // 이터러블
    const iterable = [1, 2, 3];
      
    // 이터러블의 Symbol.iterator 메서드를 호출하여 이터레이터를 생성
    const iterator = iterable[Symbol.iterator]();
      
    for( ;; ) {
      // 이터레이터의 next 메서드를 호출하여 이터러블을 순회
      // 이때 next 메서드는 이터레이터 리절트 객체를 반환
      const res = iterator.next();
      
      // next 메서드가 반환한 이터레이터 리절트 객체의 done 프로퍼티 값이 true이면 이터러블의 순회를 중단
      if(res.done) break;
        
      // 이터레이터 리절트 객체의 value 프로퍼티 값을 item 변수에 할당
      const item = res.value;
      console.log(item); // 1 2 3
    }
    ```

- `for…in`

  ```
  for (변수선언문 in 객체) { ... }
  ```

  - 객체의 프로토타입 체인 상에 존재하는 모든 프로토타입의 프로퍼티 중에서 프로퍼티 어트리뷰트 `[[Enmerable]]`의 값이 `true`인 프로퍼티를 순회하며 열거함

## 이터러블과 유사 배열 객체

- 유사배열 객체는 `length 프로퍼티`를 가지므로 for 문으로 순회가능하고 `인덱스`를 나타내는 숫자 형식의 문자열을 프로퍼티 키로 가지므로 마치 배열처럼 인덱스로 `프로퍼티 값에 접근 가능`

  ```jsx
  // 유사 배열 객체
  const arrayLike = {
    0: 1,
    1: 2,
    2: 3,
    length: 3
  };
  
  for(let i = 0; i < arrayLike.length; i++){
    console.log(arrayLike[i]); // 1 2 3
  }
  ```

- 이터러블이 아닌 `일반 객체`이므로 `for…of` 문으로 순회할 수 없음

### `Array.from 메서드`

- ES6에서 도입된 `Array.from` 메서드를 사용하여 `배열로 간단히 변환`할 수 있음

  ```jsx
  // 유사 배열 객체
  const arrayLike = {
    0: 1,
    1: 2,
    2: 3,
    length: 3
  };
  
  // Array.from은 유사 배열 객체 또는 이터러블 배열로 변환
  const arr = Array.from(arrayLike);
  console.log(arr); // [1, 2, 3];
  ```

## 이터레이션 프로토콜의 필요성

- 다양한 데이터 공급자가 하나의 순회 방식을 갖도록 규정하여 데이터 소비자가 효율적으로 다양한 데이터 공급자를 사용할 수 있도록 `데이터 소비자와 데이터 공급자를 연결하는 인터페이스 역할을 함`

  <img alt="javascript_iterable_2" src="/static/images/javascript_iterable_2.png" width="700"/>

## 사용자 정의 이터러블

### 무한 이터러블과 지연 평가

```jsx
// 무한 이터러블을 생성하는 함수
const fibonaccFunc = function () {
  let [pre, cur] = [0, 1];

  return {
    [Symbol.iterator]() {
      return this;
    },
    next() {
      [pre, cur] = [cur, pre + cur];
      // 무한을 구현해야 하므로 done 프로퍼티를 생략함
      return {value: cur};
    }
  };
};

// fibonacciFunc 함수는 무한 이터러블을 생성
for (const num of fibonacciFunc()) {
  if (num > 10000) break;
  console.log(num); //  1 2 3 5 8 ... 4181 6765
}

// 배열 디스트럭처링 할당을 통해 무한 이터러블에서 3개의 요소만 취득
const [f1, f2, f3] = fibonacciFunc();
console.log(f1, f2, f3); // 1, 2, 3
```

- `지연 평가`를 통해 데이터를 생성하여 데이터가 필요한 시점 이전까지는 미리 데이터를 생성하지 않다가 데이터가 필요한 시점이 되면 그때야 비로소 데이터를 생성

> 불필요한 데이터를 미리 생성하지 않고 필요한 데이터를 필요한 순간에 생성하므로 `빠른 실행 속도`를 기대할 수 있고 `불필요한 메모리를 소비하지 않을 수 있음`

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 614 ~ 626p
