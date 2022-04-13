---
title: Prettier
date: '2022-04-13'
tags: ['Lint']
draft: false
summary: Prettier의 설치 및 적용방법, ESLint와 통합 설정
layout: PostSimple
authors: ['default']
---

# Prettier

- ESLint과 포맷팅이 겹치는 부분이 있으나, 더 직관적인 스타일을 제공
- 한가지 차이점은 `코드 품질`과 관련된 기능은 하지 않음

## 설치 및 적용

### 설치

```shell
$ npm install -D prettier
```

### 적용

`app.js`

```javascript
console.log()
```

- 해당 파일에 console.log를 호출하는 코드를 작성

### 검증

```shell
$ npx prettier app.js --write
```

- `--write` 옵션을 줄 경우, ESLint의 `--fix` 옵션처럼 수정을 진행

#### 강점

```javascript
console.log(
  '----------------------매 우 긴 문 장 입 니 다 80자가 넘 는 코 드 입 니 다.----------------------'
)
```

- 에디터상의 가독성이 떨어지는 코드는 다음과 같이 포맷팅을 적용함

```javascript
console.log(
  '----------------------매 우 긴 문 장 입 니 다 80자가 넘 는 코 드 입 니 다.----------------------'
)
```

- 80자가 넘어가는 코드를 보다 읽기 쉬운 코드로 수정해줌
- prettier가 포맷팅은 더 강력할지 몰라도, ESLint의 코드 품질 개선 기능을 포함하고 있지 않으므로 지원하는 통합 방법을 사용

## 통합 방법

- ESLint는 앞서 설명한 것 처럼, 코드 품질과 관련된 검사를 지원하므로 둘을 같이 사용하는 방법을 권장함
- 다만 Prettier와 충돌하는 규칙을 상호간의 점검해야 하는데, `eslint-config-prettier` 패키지를 통해 잠재적인 충돌 오류를 방지해야 함

### `eslint-config-prettier` 설치

```shell
$ npm install -D eslint-config-prettier
```

### `eslint-config-prettier` 적용

`.eslintrc.js`

```javascript
{
  extends: [
    "eslint:recommended",
    "eslint-config-prettier"
  ]
}
```

- eslint init한 파일에서 extends 부분에 `eslint-config-prettier`를 추가
- `eslint:recommended` 규칙 중에 configuration이 prettier와 겹치는 부분 OFF하는 기능을 제공

### 예제

`app.js`

```javascript
var foo = ''

console.log()
```

- `var foo = '';`
  - 사용하지 않는 코드이므로, `코드 품질` 사유에 해당하고 ESLint가 간섭
- `console.log();;;;;;;`
  - 불필요한 세미콜론이므로, `포맷팅` 사유에 해당하고 ESLint와 Prettier가 간섭
  - 충돌을 방지하는 eslint-config-prettier가 정상적으로 동작하는지 확인

```shell
$ npx eslint app.js --fix # eslint
$ npx prettier app.js --write # prettier
```

- 위와 같이 각각 개별로 실행하지 않고, 통합해서 실행할 수 있는 command가 필요한 시점

### `eslint-plugin-prettier` 설치

```shell
$ npm install -D eslint-plugin-prettier
```

### `eslint-plugin-prettier` 적용

`.eslintrc.js`

```javascript
module.exports = {
  plugins: ['prettier'],
  rules: {
    'prettier/prettier': 'error',
  },
}
```

- 설정 파일에서 plugins와 rules에 설정을 추가

### 실행

```shell
$ npx eslint app.js --fix
```

- Prettier의 포맷팅 규칙을 가져와서 ESLint만 실행해도 Prettier가 적용됨

#### 단순한 설정

```javascript
{
  "extends": [
    "eslint:recommended",
    "plugin:prettier/recommended"
  ]
}
```

- Prettier는 두 패키지를 함께 사용하는 단순한 설정을 제공하는데, 위의 설정을 추가할 수 있음

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
