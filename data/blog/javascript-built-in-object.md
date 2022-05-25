---
title: 빌트인 객체
date: '2022-05-25'
tags: ['javascript']
draft: false
summary: 빌트인 객체의 분류와 정의
layout: PostSimple
authors: ['default']
---

# 빌트인 객체

## 자바스크립트 객체의 분류

## 표준 빌트인 객체

- `ECMAScript 사양에 정의`된 객체
  - Object, String, Number, Boolean, Symbol, Date, Math, RegExp, Array, Map/Set, WeakMap/WeakSet, Function, Promise,
    Reflect, Proxy, JSON, Error
  - `Math, Reflect, JSON`을 제외한 표준 빌트인 객체는 모두 인스턴스를 생성할 수 있는 `생성자 함수 객체`
- 애플리케이션 `전역의 공통 기능` 제공
  - 자바스크립트 실행 환경(브라우저 또는 Node.js 환경)과 관계없이 `언제나 사용 가능`
- `전역 객체의 프로퍼티`로서 제공
  - 별도의 선언 없이 전역 변수처럼 `언제나 참조`할 수 있음
- prototype 프로퍼티에 바인딩된 객체는 다양한 프로토타입 메서드를 제공함

```javascript
// Number 생성자 함수에 의한 Number 객체 생성
const numObj = new Number(1.5); // Number {1.5}

// toFixed는 Number.prototype의 프로토타입 메서드
// Number.prototype.toFixed는 소수점 자리를 반올림하여 문자열로 반환
console.log(numObj.toFixed()); // 2

// isInteger는 Number의 정적 메서드
// Number.isInteger는 인수가 정수(integer)인지 검사하여 그 결과를 Boolean으로 반환
console.log(Number.isInteger(0.5)); // false
```

## 호스트 객체

- 자바스크립트 실행 환경에서 `추가로 제공`하는 객체
- 브라우저 환경
  - `클라이언트 사이드 Web API` 제공
    - DOM, BOM, Canvas, XMLHttpRequest, fetch, requestAnimationFrame, SVG, Web Storage, Web Component, Web Worker
- Node.js 환경
  - `Node.js` 고유의 `API`

## 사용자 정의 객체

- 기본 제공되는 객체가 아닌 `사용자가 직접 정의한 객체`

## 원시값과 래퍼 객체

- `문자열`, `숫자`, `불리언` 값에 대해 `객체처럼 접근하면 생성`되는 임시 객체를 `래퍼 객체`라함

```javascript
const str = 'hi';

// 원시 타입인 문자열이 래퍼 객체인 String 인스턴스로 변환
console.log(str.length); // 2
console.log(str.toUpperCase()); // HI

// 래퍼 객체로 프로퍼티에 접근하거나 메서드를 호출한 후, 다시 원시값으로 되돌림
console.log(typeof str); // string
```

<img alt="javascript_built_in_object" src="/static/images/javascript_built_in_object.png" width="700"/>

- `문자열`에 대해 `마침표 표기법으로 접근`하면 그 순간 래퍼 객체인 `String 생성자 함수의 인스턴스가 생성`되고 문자열은 래퍼 객체의 `[[StringData]]` 내부 슬롯에 할당됨
- 래퍼 객체의 처리가 종료되면 식별자가 원시값을 갖도록 되돌리고 `래퍼 객체는 가비지 컬렉션의 대상이 됨`

## 전역 객체

- 코드가 실행되기 이전 단계에 자바스크립트 엔진에 의해 `먼저 생성되는 특수한 객체`, 어떤 객체에도 속하지 않은 `최상위 객체`
- `브라우저 환경`에서는 `window`(또는 self, this, frames)가 전역 객체를 가리킴
- `Node.js 환경`에서는 `global`
- ECMAScript2020(ES11)에 도입된 `globalThis`는 브라우저 환경과 Node.js 환경에서 전역 객체를 가리키던 다양한 식별자를 통일한 식별자 - 표준 사양을
  준수하는 `모든 환경에서 사용 가능`
- 특징
  - 개발자가 `의도적으로 생성할 수 없음`
  - `전역 객체 프로퍼티를 참조`할 때 `window(또는 global)` 생략 가능
- `var 키워드`로 선언한 전역 변수와 `선언하지 않은 변수에 값을 할당한 암묵적 전역,` 그리고 전역 함수는 전역객체의 프로퍼티가 됨

```javascript
// var 키워드로 선언한 전역 변수
var foo = 1;
console.log(window.foo); // 1

bar = 2; // window.ba = 2;
console.log(window.bar); // 2

// 전역 함수
function baz() {
  return 3;
}

console.log(window.baz());
3
```

- `let`이나 `const` 키워드로 선언한 전역 변수는 `전역 객체의 프로퍼티가 아님`
  - 보이지 않는 개념적인 블록(`전역 렉시컬 환경의 선언적 환경 레코드`)내에 존재

### 빌트인 전역 프로퍼티

- 전역 객체의 프로퍼티
- `Infinity`
  - 무한대를 나타내는 숫자값

    ```javascript
    // 양의 무한대
    console.log(3/0); // Infinity
    // 음의 무한대
    console.log(-3/0); // -Infinity
    ```

- `NaN`(=`Number.NaN`)
  - 숫자가 아님을 나타내는 숫자값 NaN을 갖음

    ```javascript
    console.log(Number('xyz')); // NaN
    ```

- `undefined`
  - 원시 타입 undefined를 값으로 갖음

    ```javascript
    console.log(window.undefined); // undefined
    ```

### 빌트인 전역 함수

- 애플리케이션 `전역에서 호출`할 수 있는 빌트인 함수
- `isFinite`
  - 전달받은 인수가 유한수인지, 무한수인지 불리언 형태로 반환

    ```javascript
    /**
     * 전달받은 인수가 유한수인지 확인하고 결과 반환
     * @param {number} testValue - 검사 대상 값
     * @returns {boolean} 유한수 여부 확인 결과
     */
    isFinite(testValue)
    ```

- `isNaN`
  - 전달받은 인수가 NaN인지 검사하여 그 결과를 불리언 타입으로 반환

    ```javascript
    /**
     * 전달받은 인수가 NaN인지 확인하고 결과 반환
     * @param {number} testValue - 검사 대상 값
     * @returns {boolean} 유한수 여부 확인 결과
     */
    isNaN(testValue)
    ```

- `parseFloat`
  - 전달받은 문자열 인수를 부동 소수점 숫자(`실수`)로 해석하여 반환

    ```javascript
    /**
     * 전달받은 문자열 인수를 실수로 해석하여 반환
     * @param {string} string - 반환 대상 값
     * @returns {number} 반환 결과
     */
    parseFloat(string)
    ```

- `parseInt`
  - 전달받은 문자열 인수를 `정수`로 해석하여 반환

    ```javascript
    /**
     * 전달받은 문자열 인수를 정수로 해석하여 반환
     * @param {string} string - 반환 대상 값
     * @param {string} [radix] - 진법을 나타내는 기수
     * @returns {number} 반환 결과
     */
    parseInt(string, radix)
    ```

- `encodeURI`
  - 완전한 URI를 문자열로 받아 이스케이프 처리를 위해 인코딩

    ```javascript
    /**
     * 완전한 URI를 문자열로 받아 이스케이프 처리를 위해 인코딩
     * @param {string} uri - 완전한 URI
     * @returns {string} 인코딩된 URI
     */
    encodeURI(uri)
    ```

- `decodeURI`
  - 인코딩된 URI를 인수로 전달받아 이스케이프 처리 이전으로 디코딩

    ```javascript
    /**
     * 인코딩된 URI를 인수로 전달받아 이스케이프 처리 이전으로 디코딩
     * @param {string} encodedURI - 완전한 URI
     * @returns {string} 디코딩된 URI
     */
    decodeURI(encodedURI)
    ```

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 320 ~ 341p