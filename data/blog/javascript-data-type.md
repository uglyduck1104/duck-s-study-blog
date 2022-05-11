---
title: 데이터 타입과 변수
date: '2022-04-03'
tags: ['javascript']
draft: false
summary: 자바스크립트 데이터 타입과 변수
layout: PostSimple
authors: ['default']
---

# 데이터 타입

- 원시 타입
  - boolean
  - null
  - undefined
  - number
  - string
  - symbol(ES6에서 추가)
- 객체 타입
  - obejct

## 원시 타입

- 변경 불가능한 값(immutable value) → pass-by-value(값에 의한 전달)

> 다시 알만한 내용만 부연설명

### string

- C와 같은 언어와는 다르게, 변경 불가능하다.
  ```jsx
  var str = 'Hello'
  str = 'world'
  ```
  - 첫번째 구문 실행 → 메모리에 문자열 'Hello' 생성 → str은 메모리에 생성된 문자열의 메모리 주소 참조
  - 두번째 구문 실행 → 새로운 문자열 'world' 메모리 생성 → str 참조
- 문자열은 배열처럼 인덱스를 통해 접근 가능 → `유사 배열`
  - 일부 문자 변경 불가능 → read only
  - 새로운 문자열 재할당은 가능

### undefined

- 선언은 되었지만 값을 할당하지 않은 변수에 접근했을 경우
- 존재하지 않는 객체 프로퍼티에 접근했을 경우
- 쓰레기 값이 들어있음
- 개발자가 의도적으로 undefined를 할당하지 말것 → `null로 할당하기`

### null

- 대소문자 구별(case-sensitive)
- 누구도 참조하지 않은 메모리 영역에 대해 가비지 콜렉션 수행
- 함수가 호출되었으나 유효한 값이 없을 때
- `typeof` 연산자로 null 값을 연산해보면 object가 나옴 → 설계상의 오류
  - null 타입 확인은 일치 연산자(===) 사용

### symbol

- ES6에서 새로 추가됨 → `Immutable`한 원시 타입
- 객체 프로퍼티 키를 만들기 위해 생성

  ```jsx
  // 심볼 key는 이름의 충돌 위험이 없는 유일한 객체의 프로퍼티 키
  var key = Symbol('key')
  console.log(typeof key) // symbol

  var obj = {}
  obj[key] = 'value'
  console.log(obj[key]) // value
  ```

# 객체 타입

- 데이터에 관련된 동작(절차, 방법, 기능)을 모두 포함
- 이름과 값을 가지는 프로퍼티, 동작을 의미하는 메소드를 포함할 수 있는 독립적 추체
- 자바스크립트를 이루는 거의 "모든 것"이 객체
  - 원시타입 제외한 나머지 값들(배열, 함수, 정규표현식 등)은 모두 객체
- pass-by-reference(참조에 의한 전달)

# 변수

- 프로그램에 사용되는 데이터를 일정 기간 동안 기억하여 필요한 때에 다시 사용하기 위해 데이터에 고유한 이름인 식별자를 명시함
- 변수명
  - 변수에 명시한 고유한 식별자
- 변수값
  - 변수로 참조할 수 있는 데이터
- 키워드
  - var, let, const
- 값을 할당하지 않은 변수 → undefined
- 선언하지 않은 변수 → ReferenceError

## 명명 규칙

- 반드시 영문자, underscore 또는 $로 시작
- 대소문자 구별

## 동적 타이핑

- 동적 타입 언어
- 변수의 타입 지정없이 값이 할당하는 과정에서 값의 타입에 의해 자동으로 타입 결정

## 변수 호이스팅

```jsx
console.log(foo) // ① undefined
var foo = 123
console.log(foo) // ② 123
{
  var foo = 456
}
console.log(foo) // ③ 456
```

- var 키워드는 중복 선언 가능 → 문법적으로 문제 없음
- 1에서는 ReferenceError가 발생할 것으로 예상하나 undefined가 출력됨
  - `모든 선언문은 호이스팅 됨`
    > 호이스팅
  - var 선언문이나 function 선언문 등 모든 선언문이 해당 Scope의 선두로 옮겨진 것처럼 동작. 즉, 자바스크립트는 모든 선언문이 선언되기 이전에 참조 가능

### 호이스팅 단계

- 선언 단계(Declaration phase)
  - 변수 객체에 변수 등록
  - 이 변수 객체는 스코프가 참조하는 대상이 됨
- 초기화 단계(Initialization phase)
  - 변수 객체에 등록된 변수를 메모리에 할당
  - 변수는 undefined로 초기화
- 할당 단계(Assignment phase)
  - undefined로 초기화된 변수에 실제값을 할당
- var 키워드로 선언된 변수는 선언 단계와 초기화 단계가 동시에 이루어짐
- 스코프에 변수가 등록되고 변수는 메모리에 공간을 확보한 후 undefined로 초기화함

> 변수 선언문 이전에 변수에 접근하여도 Variable Object(변수 객체)에 변수가 존재하므로 에러가 발생하지 않고 `undefined`를 반환

## 스코프

- C-family와 달리 블록 레벨 스코프를 가지지 않고 함수 레벨 스코프를 갖음
- let, const 키워드를 사용하면 블록 레벨 스코프 사용 가능

### 함수 레벨 스코프

- 함수 내에서 선언된 변수는 함수 내에서만 유효하며 함수 외부에서 참조할 수 없음
- 함수 내부에서 선언한 변수는 지역 변수이며 함수 외부에서 선언한 변수는 모두 전역 변수

### 블록 레벨 스코프

- 코드 블록 내에서 선언된 변수는 코드 블록내에서만 유효하며 코드 블록 외부에서는 참조할 수 없음

## var 키워드의 문제점

1. 함수 레벨 스코프
   - 전역 변수의 남발
   - for loop 초기화식에서 사용한 변수를 for loop 외부 또는 전역에서 참조 가능
2. var 키워드 생략 허용
   - 의도하지 않은 변수의 전역화
3. 중복 선언 허용
   - 의도하지 않은 변수값 변경
4. 변수 호이스팅
   - 변수를 선언하기 전에 참조 가능

> **Referenced**

- https://poiemaweb.com/js-data-type-variable
