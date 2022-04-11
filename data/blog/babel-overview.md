---
title: 바벨
date: '2022-04-11'
tags: ['Webpack']
draft: false
summary: 바벨의 기본 개념과 배경 살펴보기
layout: PostSimple
authors: ['default']
---

# 바벨

## 배경

- 브라우저마다 사용하는 언어가 달라서 일관적이지 못한 코드 발생
  - 크로스 브라우징 이슈로 인해 코드의 일관성이 떨어지고 브라우저 간 동작 불일치가 야기됨
- 바벨은 일관성이 떨어지는 크로스브라우징 이슈를 보완하고자 각기 다른 브라우저간의 `호환성`을 지켜줌

## 바벨의 기본 동작

### 설치

```shell
$ npm install @babel/core @bable/cli
```

- babel core를 사용하기 위한 cli도 같이 설치

## 바벨 변환 코드 작성

`app.js`

```javascript
const alert = (msg) => window.alert(msg)
```

- 해당 코드는 ES6에서 동작하는 코드
- babel을 사용해서 ES6가 아닌 환경(IE포함)에서 돌아갈 수 있게 코드 작성 필요

```shell
$ npx babel app.js
# const alert = msg => window.alert(msg);
```

- npx command를 통해 방금 설치한 babel을 실행하고 대상이 되는 파일을 지정
- `변환`하지 않고 출력됨
  - 바벨은 기본적으로 파싱과, 출력만을 실행함
  - 변환은 별도의 플러그인을 사용해야 함

### babel 빌드 과정

1. 파싱(Parsing)
   - 코드를 받고, 토큰 별로 분해
2. 변환(Transforming)
   - ES6 코드를 ES5로 변환
3. 출력(Printing)
   - 변환한 코드를 출력

## 플러그인

### 커스텀 플러그인 생성

`my-babel-plugin.js`

```javascript
module.exports = function mybabelPlugin() {
  return {
    visitor: {
      // method 선언
    },
  }
}
```

- 최상위 경로에 js를 생성하고 위와 같이 작성
- `visitor` 객체를 반환해야 함

#### 실행

```shell
$ npx babel app.js --plugins './my-babel-plugins.js'
```

- babel에 플러그인 항목에 추가하기 위해서 npx command를 실행

#### 검증

`Identifier(path) method`

```javascript
Identifier(path){
  const name = path.node.name;

  // 바벨이 만든 AST 노드를 출력함
  console.log('Identifier() name:', name)

  // 변환작업: 코드 문자열을 역순으로 반환
  path.node.name = name
    .split("")
    .reverse("")
    .join("")
}
```

- 파싱된 결과물을 `path`로 받아와서 출력함
  - token 별로 파싱한 내용을 확인할 수 있음
    `VariableDeclaration method`

```javascript
VariableDelaration(path) {
  console.log('VariableDeclaration() kind:', path.node.kind); // const

  // const => var 변환
  if(path.node.kind === 'const'){
    path.node.kind = 'var'
  }
}
```

- 파싱된 결과물을 `path`로 받아와서 출력함
  - `path.node.kind` => const를 var로 변환

## 플러그인 사용

### `preset-env`

#### 설치

```shell
$ npm install @babel/preset-env
```

### `Block-scoping`

#### 실행

```shell
$ npx babel app.js --plugins @babel/plugin-transform-block-scoping
# > var alert = msg => window.alert(msg);
```

- babel `preset-env`에 내장되어 있는 `block-scoping` plugin을 실행
- 위에서 소개한 커스텀 `VariableDeclaration method`의 동작처럼 const, let keyword를 var로 정상적인 동작을 확인할 수 있음

### `Arrow function`

#### 실행

```shell
$ npx babel app.js --plugins @babel/plugin-transform-arrow-functions
# const alert = function (msg) {
#  return window.alert(msg);
# };
```

- babel `preset-env`에 내장되어 있는 `arrow-functions` plugin을 실행
- Arrow function -> 일반 함수로 치환됨

### `use strict`

#### 설치

```shell
$ npm install @babel/plugin-transform-strict-mode
# "use strict"
```

- 브라우저 환경에서 `use strict` 모드를 활성화, 비활성화 할 수 있음

> 이처럼 늘어나는 인라인 command를 효율적으로 관리하기 위해 설정 파일(bable.config.js)로 분리

## babel configuration

`babel.config.js`

```javascript
module.exports = {
  plugins: [
    '@babel/plugin-transform-block-scoping',
    '@babel/plugin-transform-arrow-functions',
    '@babel/plugin-transform-strict-mode',
  ],
}
```

- Node.js가 제공하는 module 구문을 사용하여 배열에 plugins를 대입해 줌

### 실행

```shell
$ npx babel app.js
```

- babel이 babel.config.js 파일을 읽어서 플러그인을 적용하고, 해당 플러그인으로 코드를 변환한 후 출력

### 커스텀 프리셋

- 늘어나는 플러그인 배열로 `babel.config.js` 파일의 복잡성을 줄이기 위해 분리해서 관리
  `my-babel-preset.js`

```javascript
module.exports = function myBabelPreset() {
  return {
    plugins: [
      '@babel/plugin-transform-block-scoping',
      '@babel/plugin-transform-arrow-functions',
      '@babel/plugin-transform-strict-mode',
    ],
  }
}
```

- `my-babel-preset`의 파일을 root path에 생성
  `babel.config.js`

```javascript
module.exports = {
  presets: ['./my-babel-preset.js'],
}
```

- custom preset js 경로로 파일을 지정해 줌
- 보다 효율적으로 preset을 관리할 수 있음

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
