---
title: 타입 변환과 단축 평가
date: '2022-05-11'
tags: ['javascript']
draft: false
summary: 기존 원시 값을 변경하는 것이 아니라, 기존 원시 값을 사용해 다른 타입의 새로운 원시 값을 생성
layout: PostSimple
authors: ['default']
---

# 타입 변환과 단축 평가

- `명시적 타입 변환(=타입 캐스팅)`
  - 개발자가 의도적으로 값의 타입을 변환
- `암묵적 타입 변환(=타입 강제 변환)`
  - 개발자의 의도와는 상관없이 표현식을 평가하는 도중에 자바스크립트 엔진에 의해 변환되는 것
- 기존 원시 값을 변경하는 것이 아니라, 기존 원시 값을 사용해 다른 타입의 `새로운 원시 값을 생성`
- 타입 변환 결과를 예측하지 못하거나 예측이 결과와 일치하지 않는다면 오류가 생산됨
- 반드시 암묵적 타입변환이 나쁜 것은 아니므로 `예측 가능한 코드를 작성`해야하고, 코드를 읽는 동료로 하여금 `쉽게 이해할 수 있어야 하는 것`이 키포인트!

## 암묵적 타입 변환

### 문자열 타입으로 변환

```javascript
1 + '2' // → '12'
```

- `+` 연산자는 피연산자 중 하나 이상이 문자열이므로 연결 연산자로 동작
  - 문자열 연결 연산자의 모든 피연산자는 코드의 문맥상 모든 문자열 타입이 전제 조건
- 자바스크립트 엔진은 문자열 연결 연산자의 피연산자 중에서 문자열 타입이 아닌 피연산자를 문자열 타입으로 암묵적 타입 변환함
  - 반드시 피연산자만이 암묵적 타입 변환이 되는 것이 아님

### 숫자 타입으로 변환

```javascript
1 - '1' // -> 0
1 * '10' // -> 10
1 / 'one' // -> NaN
```

- 산술 연산자를 이용해 숫자 값을 만듦
- 산술 연산자의 모든 피연산자는 코드 문맥상 모두 숫자 타입이어야 함
- 피연산자를 숫자 타입으로 변환할 수 없는 경우는 `NaN 반환`

#### `+` 단항 연산자

```javascript
// 문자열 타입
+''       // -> 0
+ '0'      // -> 0
+ 'string' // -> NaN

// 불리언 타입
+ true     // -> 1
+ false    // -> 0

// null 타입
+ null     // -> 0

// undefined 타입
+ undefined // -> NaN

// 객체 타입
+ {}            // -> NaN
+ []            // -> 0
+ [10, 20]      // -> NaN
+ (funtion(){}) // -> NaN
```

- 빈 문자열, 빈 배열, null, false는 0으로 변환
- 객체와 빈 배열이 아닌 배열, undefined는 NaN 반환

### 불리언 타입으로 변환

```javascript
if ('') console.log('1')
if (true) console.log('2')
if (0) console.log('3')
if ('str') console.log('4')
if (null) console.log('5')

// 2 4
```

- 불리언 타입이 아닌 값을 Truthy(참으로 평가) 또는 Falsy(거짓으로 평가)값으로 구분
  - Truthy → true
  - Falsy → false

## 명시적 타입 변환

### 문자열 타입으로 변환

#### String 생성자 함수를 new 연산자 없이 호출

```javascript
String(1) // -> "1"
String(NaN) // -> "NaN"
String(true) // -> "true"
```

#### Object.prototpe.toString 메서드 호출

```javascript
;(1).toString() // -> "1"
NaN.toString() // -> "NaN"
true.toString() // -> "true"
false.toString() // -> "false"
```

#### 문자열 연결 연산자 사용

```javascript
1 + '' // -> "1"
NaN + '' // -> "NaN"
true + '' // -> "true"
```

### 숫자 타입으로 변환

#### Number 생성자 함수를 new 연산자 없이 호출하는 방법

```javascript
Number('0') // -> 0
Number(true) // -> 1
```

#### parseInt, parseFloat 함수를 이용(문자열만 가능)

```javascript
parseInt('0') // -> 0
parseFloat('10.53') // -> 10.53
```

#### `+` 단항 산술 연산자 이용

```javascript
;+'0' + // -> 0
  true // true
```

#### `+` 산술 연산자를 이용

```javascript
'0' * 1 // -> 0
'10.53' * 1 // -> 10.53
true * 1 // -> 1
```

### 불리언 타입으로 변환

#### Boolean 생성자 함수를 new 연산자 없이 호출하는 방법

```javascript
Boolean('x') // -> true
Boolean('') // -> false
```

#### ! 부정 논리 연산자를 두 번 사용하는 방법

```javascript
!!'x' // -> true
!!'' // -> false
!!'false' // -> true
!!0 // -> false
```

## 단축 평가

### 논리 연산자를 사용한 단축 평가

#### 논리곱(`&&`)

```javascript
'Cat' && 'Dog' // -> "Dog"
```

- 두 개의 피연산자가 `모두 true로 평가`될 때 true 반환
- 좌항에서 우항으로 평가

#### 논리합(`||`)

- 두 개의 피연산자 중 `하나만 true`로 평가되어도 true 반환
- 좌항에서 우항
- 피연산자를 타입 변환하지 않고 그대로 반환(`단축 평가`)
  - 표현식을 평가하는 도중에 평가 결과가 확정된 경우, `나머지 평가 과정을 생략`하는 것을 말함

| 단축 평가 표현식  | 평가 결과 |
| ----------------- | --------- |
| true ll anything  | true      |
| false ll anything | anything  |
| true && anything  | anything  |
| false && anything | false     |

### 단축 평가 사용해서 TypeError 피하기

```javascript
var elem = null
// elem이 null이나 undefined와 같은 Falsy 값이면 elem으로 평가
// elem이 Truthy 값이면 elem.value로 평가
var value = elem && elem.value // -> null
```

### 단축 평가 사용해서 기본값 설정

- 함수 호출 시 인수가 전달되지 않으면 undefined가 할당됨

```javascript
// 단축 평가로 매개변수 기본값 설정
function getStringLength(str) {
  str = str || ''
  return str.length
}

// ES6의 매개변수 기본값 설정
function getStringLength(str = '') {
  return str.length
}
```

### 옵셔널 체이닝 연산자

- ES11에서 도입된 옵셔널 체이닝 연산자(`?.`)
  - 좌항의 피연산자가 null 또는 undefined인 경우 undefined를 반환
    - 그렇지 않으면 우항 프로퍼티 참조
- 주로 객체를 가리키는 변수의 null 또는 undefined 체크할때 사용

```javascript
var elem = null

var value = elem?.value
console.log(value) // undefined

var str = ''

var length = str?.length
console.log(length) // 0
```

### null 병합 연산자

- ES11에서 도입된 null 병합 연산자(`??`)
  - 좌항의 피연산자가 null 또는 undefined인 경우 우항의 피연산자를 반환
    - 그렇지 않은 경우 좌항의 피연산자를 반환
- 주로 기본값을 설정할 때 유용

```javascript
var foo = '' ?? 'default string'
console.log(foo) // ""
```

- 좌항의 피연산자가 Falsy 값이라도 null 또는 undefined가 아니면 좌항의 피연산자를 반환

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 116 ~ 120p
