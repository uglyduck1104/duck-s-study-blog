---
title: 타입 다루기
date: '2022-11-11'
tags: ['javascript']
draft: false
summary: 타입 검사, undefined와 null, eqeq 줄이기, 형변환 주의하기
layout: PostSimple
authors: ['default']
---

# 타입 다루기

## 타입 검사

### `typeof` 연산자

```jsx
typeof '문자열' // 'string'
typeof true // 'boolean'
typeof undefined // 'undefined'
typeof 123 // 'number'
typeof Symbol() // 'symbol'
```

- 피연산자(우항)를 평가해서 문자열로 반환하는 연산자

### typeof는 만능이 아님

- `typeof`로 모든게 커버가능하지 않음
- `PRIMITIVE` vs `REFERENCE`

  - 원시값은 typeof로 감별하기 무난하지만 참조값은 타입검사로 구분하기 어려움

    ```jsx
    function myFunction() {}
    console.log(typeof myFunction) // 'function'

    class MyClass {}
    console.log(typeof MyClass) // 'function'
    ```

    - 함수는 typeof 연산자를 사용했을 때, `function` 문자열을 반환하고, 클래스 또한 typeof 연산자를 사용하면 `function` 문자열을 반환해 타입을 규정하기 어려움

  - `Wrapper` 객체 또한 `object`로만 규정됨

    ```jsx
    const str = new String('문자열')
    console.log(typeof str) // 'object'
    ```

  - `null`도 `object`

    ```jsx
    console.log(typeof null) // 'object'
    ```

    - 공식적으로 수정을 할 수 없는 언어적 오류로 인정됨

- 암묵적인 변환으로 인해 타입 또한 동적으로 규정됨

### `instanceof` 연산자

```jsx
function Person(name, age) {
  this.name = name
  this.name = age
}

const p = {
  name: 'test',
  age: 99,
}

const poco = new Person('poco', 99)
p instanceof Person // false
```

- 객체의 프로토타입 체인을 검사

```jsx
const arr = []
const func = function () {}
const date = new Date()

arr instanceof Array // true
func instanceof Function // true
date instanceof Date // true

// Confused
arr instanceof Object // true
func instanceof Object // true
func instanceof Object // true
```

- 참조값의 형태는 `Object`로도 규정될 수 있음
  - 프로토타입 체인의 최상위는 `Object`이므로 타입 검사가 어려워짐

### 프로토타입 체인 역이용하기

```jsx
const arr = []
const func = function () {}
const date = new Date()

Object.prototype.toString.call('') // '[object String]'
Object.prototype.toString.call(arr) // '[object Array]'
Object.prototype.toString.call(func) // '[object Function]'
Object.prototype.toString.call(date) // '[object Date]'
```

- `call method`를 통해 각괄호를 사용한 문자열 안에 타입을 확인할 수 있음
  - 래퍼 객체까지 감별 가능

## undefined & null

- 언어적으로 엄격하지 않은 Javascript의 특징으로 봤을 때, 많이 헷갈릴 수 있는 부분이 주로 `undefiend`와 `null`의 차이점임

### null

```jsx
!null // true
!!null // false

null === false // false
!null === true // true

// null => math => 0
null + 123 // 123
```

- 비어있는 값을 명시적으로 지정하는 방법이지만, 숫자로 표현하면 0과 같음

### undefined

```jsx
// 선언했지만 값은 정의되지 않고 할당하지 않음
let varb

typeof varb // 'undefined'

undefined + 10 // NaN

!undefined // true

undefined == null // true
undefined === null // false
!undefined === !null // true
```

- 정의하지 않은 것에 대한 기본 값
- `null`은 `0`이라는 숫자로 표현가능하지만, `undefined`는 `NaN`로 표현됨
- 팀에서 `undefined`와 `null`의 쓰임새에 대해 컨벤션을 만들어 혼란을 방지하는 것을 권장

## eqeq 줄이기

- 자바스크립트에서 동등연산자(`==`)를 사용하는 행위를 자제해야 함
  - 엄격한 동등연산자(`===`)로 치환해서 사용해야 함

### 자제해야하는 이유

```jsx
'1' == 1 // true
1 == true // true

'1' === 1 // false
1 === true // false
```

- 동등 연산자를 사용할 경우, 암묵적인 형변환(`type casting`)이 일어나서 타입을 추론하기 어려움
- 두개의 피연산자를 비교하는 상황에서는 엄격한 동등 연산자를 사용해서 예측 불가능한 형변환을 방지할 수 있음
  - `Number(ticketNum.value) === 0` 과 같이, 안전한 구문 비교를 해야함
- ESLint `eqeqeq` Option을 사용해서, 문제가 될만한 상황을 사전에 예방

## 형변환 주의하기

- 앞서 다뤄진 동등연산자와 엄격한 동등연산자의 차이를 table로만 봐도 큰 차이를 느낄 수 있음

### **== (negated: !=)**

![javascript_cleancode_type1](/static/images/javascript_cleancode_type1.png)

### **=== (negated: !==)**

![javascript_cleancode_type2.png](/static/images/javascript_cleancode_type2.png)

### 형변환으로서의 관점

```jsx
'1' === 1 // 느슨한 검사 => 형 변환
1 == true
0 == false
```

- 같은 타입이 아님에도 느슨한 검사로 인해 형 변환이 발생

```jsx
// 암묵적 변환
11 + ' 문자와 결합' // '11 문자와 결합'
!!'문자열' // true
!!'' // false

// 명시적 변환
parseInt('9.9999', 10) // 9
```

- `parseInt` ⭐⭐⭐
  - 10진수로 변환될 것을 기대했지만, 값을 지정하지 않았다면 10진수가 기본이 아님
  - 두번째 `arg`에 10진수를 명시해야함

### 명시적 변환 사용

```jsx
String(11 + ' 문자와 결합') // '11 문자와 결합'

Boolean('문자열') // true
Boolean('') // false
Number('11') // 11
```

- 래퍼객체를 사용해 명시적인 형변환을 사용

> 사용자 → 명시적인 변환 (타입)
> JS → 암묵적인 변환 (타입)

## isNaN

- 사람이 인지하는 숫자(10진수)와 컴퓨터가 인지하는 숫자(2진수)가 다르기 때문에 간극이 발생
  - 숫자를 다루는 데 있어서 소수점을 다루기가 어려워짐
  - `Javascript IEEE 754` 표준을 사용해 부동 소수점(떠돌이 소수점)을 표현

```jsx
Number.MAX_SAFE_INTEGER // 최대 숫자
Number.isInteger // 정수 판별
isNaN // is Not A Number => 숫자가 아님을 의미

typeof 123 === 'number' // true
```

### isNaN의 오용

```jsx
isNaN(123 + '테스트') // true
Number.isNaN(123 + '테스트') // false
```

- 숫자가 아닌 것에 대해서 검사하기 어려움
- `isNaN`을 독립적으로 사용하지 않고 `Number.isNaN`를 사용하는 것을 권장
  - `isNaN` → 느슨한 검사
  - `Number.isNaN` → 엄격한 검사

> **Referenced**

- [클린코드 자바스크립트](https://www.udemy.com/course/clean-code-js/)
