---
title: Webpack HMR(Hot Module Replacement)
date: '2022-04-14'
tags: ['Webpack']
draft: false
summary: Hot Module Replacement의 등장 배경 및 설정
layout: PostSimple
authors: ['default']
---

# HMR(Hot Module Replacement)

## 배경

- 코드의 변화를 감지하고 전체 화면을 갱신하는 웹팩 개발서버의 특징으로 인해 SPA 개발 환경에서 브라우저가 주도권을 가지는 데이터의 초기화 문제 발생
- 변경한 모듈만 Refresh하는 목적을 `Hot Module Replacement`, 웹팩 개발서버가 제공해 줌

## 설정

- 설정은 비교적 간단한 편
  `webpack.config.js`

```javascript
module.exports = {
  devServer: {
    hot: true,
  },
}
```

- `hot`에 true 값을 설정하면 hot module replacement가 활성화 됨

## 변경 감지 확인

`app.js`

```javascript
if (module.hot) {
  console.log('핫 모듈 켜짐')

  module.hot.accept('./result', () => {
    console.log('result 모듈 변경됨')
  })
}
```

- `HMR`을 설정하면 `module`의 `hot`에 접근이 가능
- module의 hot의 객체의 `accept` 함수를 통해 변경할 module을 명시
- 지정한 module의 변경이 감지가 되면, callback() 함수가 실행

## 핫로딩을 지원하는 로더

- 핫로딩을 지원하는 로더는 style-loader에서도 `hot.accept()` 함수를 사용하고 있음
- 이외에도 리액트를 지원하는 react-hot-loader, 파일을 지원하는 file-loader도 있음
  - 지원하는 개발 환경에 따라 로더를 확인해서 적용하는 것을 권장함

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
