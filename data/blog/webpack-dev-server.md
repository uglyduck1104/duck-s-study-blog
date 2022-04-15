---
title: Webpack Dev Server
date: '2022-04-14'
tags: ['Webpack']
draft: false
summary: 웹팩 개발 서버의 배경 및 설치 사용
layout: PostSimple
authors: ['default']
---

# Webpack Dev Server

- 운영환경에 맞춰서 배포시 잠재적 문제를 미리 확인하거나, ajax 방식의 api 연동은 cors 정책 떄문에 반드시 서버가 필요함
- 프론트엔드 개발환경에서 이러한 개발용 서버를 제공해 주는 것이 `webpack-dev-server`

## 설치

```shell
$ npm install -D webpack-dev-server
```

## 적용

`package.json`

```json lines
{
  "scripts": {
    "start": "webpack-dev-server"
  }
}
```

- webpack-dev-server를 실행하는 build script를 추가

## 실행

```shell
$ npm start
# ℹ ｢wds｣: Project is running at http://localhost:8080/
# ℹ ｢wds｣: webpack output is served from /
# ℹ ｢wds｣: Content not from webpack is served from /:project_path
```

- localhost 8080 port에서 실행된 webpack dev server를 확인할 수 있음
- 개발 서버를 제공해주는 것 뿐만 아니라, 코드 변화 감지를 통해서 브라우저에서 실시간으로 보여줌

## 설정

- 웹팩 설정 파일의 devServer 객체에 개발 서버 옵션 설정 가능
  `webpack.config.js`

```javascript
module.exports = {
  devServer: {
    contentBase: path.join(__dirnaem, 'dist'),
    publicPath: '/',
    host: 'dev.domain.com',
    overlay: true,
    port: 8081,
    stats: 'errors-only',
    historyApiFallback: true,
  },
}
```

### Options

- `contentBase` 정적 파일을 제공할 경로(default = webpack output)
- `publicPath` 브라우져를 통해 접근하는 경로(default = /)
- `host`
  - 개발 환경에서 도메인을 맞추는 환경에서 사용
    - ex) 쿠키 기반의 인증 -> 인증 서버와 동일한 도메인으로 개발환경 구성
      - 운영체제 호스트 파일에 해당 도메인과 127.0.0.1 연결을 추가한 뒤 host 속성에 도메인 설정해서 사용
- `overlay` 에러나 경로를 브라우저로 출력할 것인지
- `port` 개발 서버 포트 번호 설정(default = 8080)
- `stats`
  - 메시지 수준 정의
    - `none`, `errors-only`, `minimal`, `normal`, `verbose`
- `historyApiFallBack`
  - 히스토리 API를 사용하는 SPA 개발시 설정
  - 404발생 시 index.html로 리다이렉트
- `--progress` option
  - 빌드 진행율 출력(빌드 시간 측정 시 유용함)

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
