---
title: strict mode
date: '2022-06-10'
tags: ['javascript']
draft: false
summary: 자바스크립트 언어의 문법을 좀 더 엄격하게 적용하여 오류를 발생시킬 가능성이 높거나 자바스크립트 엔진의 최적화 작업에 문제를 일으킬 수 있는 코드에 대해 명시적인 에러를 발생시킴
layout: PostSimple
authors: ['default']
---

# 엄격 모드

## 장점

- 자바스크립트 언어의 문법을 좀 더 엄격하게 적용하여 오류를 발생시킬 가능성이 높거나 자바스크립트 엔진의 최적화 작업에 문제를 일으킬 수 있는 코드에 대해 `명시적인 에러를 발생시킴`

## 단점(strict mode가 발생시키는 에러)

### 암묵적 전역

```jsx
(function () {
  'use strict';

  x = 1;
  console.log(x); // ReferenceError: x is not defined
}());
```

- 선언하지 않은 변수를 참조하면 `ReferenceError: x is not defined` 에러 발생

### 변수, 함수, 매개변수의 삭제

```jsx
(function () {
  'use strict';

  var x = 1;
  delete x; // SyntaxError: Delete of an unqualified identifier in strict mode.

  function foo(a) {
    delete a; // SyntaxError: Delete of an unqualified identifier in strict mode
  }

  delete foo; // SyntaxError: Delete of an unqualified identifier in strict mode
}());
```

- delete 연산자로 변수, 함수, 매개변수를 삭제하면 `SyntaxError`가 발생

### 매개변수 이름의 중복

```jsx
(function () {
  'use strict';

  // SyntaxError: Duplicate parameter name not allowed in this context
  function foo(x, x) {
    return x + x;
  }

  console.log(foo(1, 2));
}());
```

- 중복된 매개변수 이름을 사용하면 `SyntaxError`가 발생함

## 전역 strict mode 사용 시

- `strict mode`와 `non-strict mode`의 혼용 하는 경우 오류 발생의 위험이 있음
- 특히, 외부 서드파티 라이브러리를 사용하는 경우 라이브러리가 non-strict mode인 경우도 있기 때문에 전역 strict mode 적용 시 오류를 발생시킬 수 있음
- `해결`
  - 즉시 실행 함수로 스크립트 전체를 감싸서 스코프를 구분하고 `즉시 실행 함수`의 선두에 strict mode를 적용

    ```jsx
    // 즉시 실행 함수의 선두에 strict mode 사용
    (function(){
    	'use strict';
    
    	// Do something...
    }());
    ```

## 브라우저 콘솔에서 사용하기

- 기본적으로 `use strict`가 적용되지 않는다는 점에 주의해야 함

```jsx
'use strict'; // <Shift+Enter를 눌러 줄바꿈>
// 테스트 코드 입력
// <Enter 눌러 실행>
```

```jsx
(function(){
  'use strict';

  // 테스트 코드 입력
}());
```

## 꼭 사용해야 하는가?

- 모던 자바스크립트는 ES6에 도입된 `클래스`와 `모듈`이라는 구조를 제공하므로 엄격모드(use strict)가 자동으로 적용됨

> **Referenced**

- [모던 JavaScript 튜토리얼](https://ko.javascript.info/strict-mode)
- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 313 ~ 319p
