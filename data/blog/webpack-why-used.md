---
title: 웹팩이 필요한 이유와 기본 동작
date: '2022-04-06'
tags: ['Webpack']
draft: false
summary: 문법 수준에서 모듈을 지원하기 시작한 것은 ES2015 부터 시작
layout: PostSimple
authors: ['default']
---

# 배경

- 문법 수준에서 모듈을 지원하기 시작한 것은 ES2015 부터 시작
- import/export 구문이 없던 시절에 어떻게 모듈화 했을까?

#### `math.js`

```javascript
function sum(a, b) {
  return a + b
}
```

#### `app.js`

```javascript
console.log(sum(1, 2))
// 3
```

#### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <script src="src/math.js"></script>
    <script src="src/app.js"></script>
  </body>
</html>
```

- 과거에는 index.html을 만들고 script를 불러와야만 브라우저 상에서 확인할 수 있었음
- 문제 인식
  - math.js의 function은 어디에서나 호출 가능
    - 전역 스코프가 난해해짐
    - 개발자 도구에서 window 객체에 해당 function을 호출할 수 있음
    - 예측할 수 없는 런타임 에러 발생

## IIFE 도입

- 즉시 실행 함수 표현(IIFE, Immediately Invoked Function Expression)을 말함

```javascript
(function () {
  statements
})()
```

- 첫 번째 괄호
  - 전역 스코프에 불필요한 변수를 추가해서 오염시키는 것을 방지, 내부안으로 다른 변수의 값이 접근하는 것을 막을 수 있음
- 두 번재 괄호
  - 즉시 실행 함수를 생성하는 괄호
    > 전역 스코프의 오염 방지

### IIFE 방식으로 수정

#### `math.js`

```javascript
var math = math || {}

(function () {
  function sum(a, b) {
    return a + b
  }

  math.sum = sum
})()
```

- math 변수 할당
- IIFE 방식으로 외부 함수를 만들고 해당 스코프 안에 `sum` 함수를 정의(독립)
- 외부에서 math 변수 호출을 위해 `sum` 함수를 할당

#### `app.js`

```javascript
console.log(math.sum(1, 2))
```

- math 변수에 할당되어 있는 `sum` 함수를 호출함

### IIFE 방식의 모듈 스펙

#### `CommonJS`

- 자바스크립트를 사용하는 모든 환경에서 `모듈화` 하는게 목표
  `math.js`

```javascript
exports function sum(a, b) { return a + b; }
```

`app.js`

```javascript
const sum = require('./math.js')
sum(1, 2) // 3
```

- `exports` 키워드로 모듈을 만들고 require() 함수로 호출하는 형식
- 대표적으로 Server-side platform `NodeJS`에서 사용

#### `AMD(Asynchronous Module Definition)`

- 브라우저 환경의 비동기 로딩 방식 모듈

#### `UMD(Universal Module Definition)`

- AMD 기반으로 CommonJS 방식까지 지원(통합 형태)
  > ES2015에서 표준 모듈 시스템을 발표
  > 지금의 babel, webpack

## ES2015 표준 모듈 시스템

`math.js`

```javascript
export function sum(a, b) {
  return a + b
}
```

`app.js`

```javascript
import * as math from './math.js'
// import { sum } from './math.js'; 해당 방식으로도 호출 가능

math.sum(1, 2) // 3
```

- `export` 구문으로 모듈을 만들고 `import` 구문으로 호출

## 브라우져의 모듈 지원

- 인터넷 익스플로러를 포함한 몇 브라우져에서는 모듈을 사용하지 못함

#### `index.html`

```html
<script type="module" src="app.js"></script>
```

- \<script> 태그로 로딩 시, type 속성을 이용해 `module`을 명시함
- 그럼에도 불구하고 여전히 몇몇 브라우저에서는 사용이 불가능함

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
