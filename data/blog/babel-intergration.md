---
title: 바벨 사용법과 웹팩 통합
date: '2022-04-11'
tags: ['Webpack']
draft: false
summary: 바벨 사용법과 웹팩 통합
layout: PostSimple
authors: ['default']
---

# 프리셋 사용하기

- preset-env
  - ECMAScript2015+를 변환할 때 사용
- preset-[flow, react, typescript]
  - 각 환경마다 변환하기 위한 프리셋
    > 목적에 따라 프리셋을 제공

## `preset-env` 설치

```shell
$ npm install -D @babel/preset-env
```

## `preset-env` 설정

`babel.config.js`

```javascript
module.exports = {
  presets: ['@babel/preset-env'],
}
```

- 설치한 preset-env의 plugin을 가리킴
- 가장 현업에서 많이 쓰이는 plugin

## 타겟 브라우저

- preset으로 변환하는 코드는 특정 브라우저를 지원해야 함

### 설정

```javascript
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '79',
          ie: '11',
        },
      },
    ],
  ],
}
```

- 첫번째 인자로 plugin을 받고, 두번째 인자로 option 값을 넘겨줌
- targets에 지원하려는 브라우저의 이름과 버전을 명시함

#### 결과

```shell
"use strict";

var alert = function alert(msg) {
  return window.alert(msg);
};
```

- ie 11버전에는 `const` keyword와 `arrow function`을 지원하지 않음
  - [키워드 지원 확인하는지 보러 가기](https://caniuse.com/)

### Promise 객체를 통한 예제

`app.js`

```javascript
new Promise()
```

- 변환을 예상했지만, 해석하지 못하고 에러를 던짐
- 바벨은 ECMAScript2015+를 ECMAScript5 버전으로 변환할 수 있는 것만 변환함
  - 그렇지 못한 것들은 `폴리필`이라고 부르는 코드 조각을 추가해서 해결

### 폴리필

`babel.config.js`

```javascript
module.exports = {
  resets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '79',
          ie: '11',
        },
        useBuiltIns: 'usage', // 'entry', false
        corejs: {
          version: 2, // 3
        },
      },
    ],
  ],
}
```

- 앞서 봤던 Promise는 ECMAScript5 버전으로 대체가 불가능
  - 하지만 ECMAScript5 버전으로 구현할 수는 있음
  - core-js library option을 명시해서 폴리필을 사용할지 설정할 수 있음
    - `usage`, `entry`, false(`default`)
    - corejs 버전을 명시

#### `core.js` 설치

```shell
$ npm install core-js@2
```

#### 결과

```javascript
'use strict'
require('core-js/modules/es6.promise')
require('core-js/modules/es6.object.to-string')
new Promise()
```

- core-js가 먼저 로딩된 후 위와 같이 번들링된 코드가 순차적으로 로딩되어야 정상적인 동작을 보장받을 수 있음

## 웹팩으로 통합

- 바벨은 웹팩의 로더 형태로 제공

### 설치

```shell
$ npm install babel-loader
```

### 적용

`webpack.config.js`

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
      },
    ],
  },
  // ...
}
```

- `js` 파일에만 동작하고, `loader`는 `babel`을 지정
- `node_modules`는 바벨 로더 처리하지 않도적 지정
- entry point를 내가 빌드하고자 하는 경로와 일치하는지 다시한번 확인

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
