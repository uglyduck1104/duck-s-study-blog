---
title: ESLint
date: '2022-04-13'
tags: ['Lint']
draft: false
summary: 린트의 등장 배경과 구성 방법
layout: PostSimple
authors: ['default']
---

# 린트

## 배경

### 린트가 필요한 상황

```javascript
console.log()(function () {})()
// TypeError 발생
// console.log()(function () {})() 로 읽힘
```

- 브라우저는 코드에 세미콜론을 자동으로 넣는 과정 수행(ASI)
- 하지만 의도치 않는 문법상의 오류로 Human Error 발생

### ESLint 등장

- ECMAScript 코드에서 문제점을 검사하고 더 나은 코드로 정정하는 린트 도구
- 코드의 가독성을 높이고 잠재적인 오류와 버그를 제거
- 과거 JSLint, JSHint에 이어 최근에 많이 사용하는 린트 도구

#### 역할

- 포맷팅
  - 들여쓰기, 줄 길이 등 일련의 규칙을 정할 수 있음
- 코드품질
  - 잠재적인 오류를 사전에 체크해주는 기능

## 설치 및 실행

### 설치

```shell
$ npm install eslint
```

### 설정 파일 생성

`.eslintrc.js`

```javascript
module.export = {}
```

- 아무런 설정없이 설정 파일을 먼저 생성(최초 정상 실행 확인)

### 실행

#### 실행 파일 생성

`app.js`

```javascript
console.log()(function () {})()
```

- 초기에 썼던 문법 오류 코드로 검증

#### 실행

```shell
$ npx eslint app.js
```

- 아무런 오류 없이 실행됨을 확인
- 아직 eslint의 configuration을 하지 않았기 때문에 아무런 로그가 출력되지 않음

## 설정

### 규칙

- ESLint는 검사 규칙을 미리 정해놓았기 때문에 Rule을 환경설정에 추가해야 함
- [공식 문서 참고](https://eslint.org/docs/rules/)
  - 설정하고자 하는 규칙을 정함

### no-unexpected-multiline 예제

`.eslintrc.js`

```javascript
module.exports = {
  rules: {
    'no-unexpected-multiline': 'error',
  },
}
```

- 설정하고자하는 규칙 중에 multiline 관련 오류를 출력하는 규칙을 설정
  - ```text
    [Error 경로 출력 경로 / 파일명]
    2:1  error  Unexpected newline between function and ( of function call  no-unexpected-multiline
    ✖ 1 problem (1 error, 0 warnings)
    ```
  - 함수와 괄호 사이에 줄바꿈 오류를 내뱉음
    > 해당 파일의 문법 오류를 수정한 후 다시 실행하면 오류 로그없이 아무런 출력을 하지 않음(정상 동작)

### no-extra-semi

`.eslintrc.js`

```javascript
module.exports = {
  rules: {
    'no-extra-semi': 'error',
  },
}
```

`app.js`

```javascript
console.log()
;(function () {})()
```

- 세미콜론(;)이 여러개인 경우, 오류 사항은 아니지만 `no-extra-semi` rule을 추가하면 error로 출력할 수 있음
  - ```text
    [Error 경로 출력 경로 / 파일명]
    1:15  error  Unnecessary semicolon  no-extra-semi
    1:16  error  Unnecessary semicolon  no-extra-semi
    1:17  error  Unnecessary semicolon  no-extra-semi
    1:18  error  Unnecessary semicolon  no-extra-semi
    1:19  error  Unnecessary semicolon  no-extra-semi
    1:20  error  Unnecessary semicolon  no-extra-semi
    ✖ 6 problems (6 errors, 0 warnings)
    6 errors and 0 warnings potentially fixable with the `--fix` option.
    ```
  - 불필요한 세미콜론 사용을 수정하라는 오류를 출력
  - `--fix option 제안`
    - 자동으로 수정할 수 있는 규칙 존재
    - 위에서 첨부한 eslint 공식 사이트에서 🔧 아이콘이 붙어있는 경우 자동으로 수정할 수 있음

### Extensible Config

`.eslintc.js`

```javascript
module.exports = {
  extends: ['eslint:recommended'],
}
```

- ESLint의 규칙이 너무 많아서 고르기 어렵다면, fix option을 제공하는 규칙들을 미리 정해놓은 `eslint:recommended` 설정을 추가함
- 이외에도 기본적으로 제공하는 설정 외에 자주 사용하는 두가지 규칙이 존재
  - standard
    - javascript standard style
  - airbnb
    - airbnb의 스타일 가이드

## 초기화

- 위와 같은 설정을 `--init` 옵션을 추가하여 대화 형식으로 손쉽게 구성 가능

```shell
$ npx eslint --init
#? How would you like to use ESLint? To check syntax and find problems
#? What type of modules does your project use? JavaScript modules (import/export)
#? Which framework does your project use? None of these
#? Does your project use TypeScript? No
#? Where does your code run? Browser
#? What format do you want your config file to be in? JavaScript
#Local ESLint installation not found.
#The config that you've selected requires the following dependencies:
#eslint@latest
#? Would you like to install them now with npm? Yes
```

- 본인 개발 환경에 따라 선택하면, 아까 구성했던 `.eslintrc.js` 파일이 새로 갱신됨

### 확인

`.eslintrc.js`

```javascript
module.exports = {
  env: {
    browser: true,
    es6: true,
  },
  extends: 'eslint:recommended',
  globals: {
    Atomics: 'readonly',
    SharedArrayBuffer: 'readonly',
  },
  parserOptions: {
    ecmaVersion: 2018,
    sourceType: 'module',
  },
  rules: {},
}
```

- 터미널에서 구성했던 옵션을 해당 파일에서 확인할 수 있음

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
