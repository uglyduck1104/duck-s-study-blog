---
title: Webpack Connect API
date: '2022-04-14'
tags: ['Webpack']
draft: false
summary: 웹팩 개발 서버의 API 연동 방법
layout: PostSimple
authors: ['default']
---

# API 연동

## Mockup API(devServer.before)

- 개발 서버 설정의 before 속서을 이용해 웹팩 서버에 기능을 추가할 수 있음
- Node Express.js(Framework)에 사전 지식이 있으면 유리

### 적용

`webpack.config.js`

```javascript
module.exports = {
  devServer: {
    before: (app) => {
      app.get('/api/users', (req, res) => {
        res.json([
          {
            id: 1,
            name: 'Alice',
          },
          {
            id: 2,
            name: 'Bek',
          },
          {
            id: 3,
            name: 'Chris',
          },
        ])
      })
    },
  },
}
```

- ExpressJS의 Server instance를 `app`이라는 이름으로 webpack-dev-server가 넣어줌
- HTTP Method `get`으로 이루어진 API를 만들어 내는 함수 추가
- req, res 인자로 받아서 json 형태로 반환

### 확인

- `npm start` command로 webpack-dev-server를 활성화하고, URL에 endpoint를 기입
  - `http://localhost:8080/api/users`

## axios

- ajax를 호출하는 라이브러리 중 `axios`를 설치해서 적용

### 설치

```shell
$ npm install axios
```

### 적용

`app.js`

```javascript
import axios from 'axios'

document.addEventListener('DOMContentLoaded', async () => {
  const res = await axios('/api/users')
  console.log(res)

  document.body.innerHTML = (res.data || [])
    .map((user) => {
      return `
      <div>${user.id}: ${user.name}</div>
    `
    })
    .join('')
})
```

- 설치한 axios를 import하고 dom의 로드가 끝나면 api 요청을 날리고, webpack-dev-server로 부터 api 응답을 받아 console.log로 출력
- user의 id와 name을 출력

## Mockup package

### `connect-api-mocker`

- 목업 api 작업 한개가 아니라 여러개 일때 사용하는 패키지
- 특정 목업 폴더를 만들어 api 응답을 담은 파일을 저장한 뒤, 폴더를 api로 제공

#### 설치

```shell
$ npm install -D connect-api-mocker
```

#### 적용

`mocks/api/users/GET.json`

```json lines
[
  {
    "id": 1,
    "name": "Alice"
  },
  {
    "id": 2,
    "name": "Bek"
  },
  {
    "id": 3,
    "name": "Chris"
  }
]
```

- 해당 경로에 JSON 파일을 생성

`webpack.config.js`

```javascript
const apiMocker = require('connect-api-mocker')

module.exports = {
  devServer: {
    before: (app) => {
      app.use(apiMocker('/api', 'mocks/api'))
    },
  },
}
```

- ExpressJS의 use함수를 이용해서 미들웨어를 추가(apiMocker)
  - 첫번째 인자(urlRoot)
    - 해당 URL로 요청 오는 모든 api는 apiMocker가 처리
  - 두번째 인자(pathRoot)
    - project 내의 api 경로를 추가

## API 연동(devServer.proxy)

- api 서버를 로컬환경에서 띄우고 목업이 아닌 직접 api 요청하는 방법

### 실행 가정

```shell
$ curl localhost:8081/api/keywords
[{"keyword":"이탈리어"},{"keyword":"셰프의 요리"},{"keyword":"제철"},{"keyword":"홈파티"}]
```

- 웹팩 개발 서버를 띄우고 axios로 요청을 하게 되면 브라우저 상에서 COR 정책에 의해 호출 오류를 냄
  > CORS(Cross Origin Resource Sharing) 브라우져와 서버간의 보안상의 정책, 브라우저가 최초로 접속한 서버에서만 ajax 요청할 수 있음
- 그러므로, localhost는 같은 도메인이지만 포트번호가 달라서 다른 서버로 인식하는 문제가 발생됨

### 해결 방안

#### 서버에서 설정하는 방법

`server/index.js`

```javascript
app.get('/api/keyworks', (req, res) => {
  res.header('Access-Control-Allow-Origin', '*') // 헤더를 추가
  res.jon(keywords)
})
```

- 해당 API 응답 헤더에 `Access-Control-Allow-Origin`을 추가

#### 브라우저 환경에서 설정하는 방법

`webpack.config.js`

```javascript
module.exports = {
  devServer: {
    proxy: {
      '/api': 'http://localhost:8081', // 프록시
    },
  },
}
```

- 서버 응답 헤더를 추가할 필요 없이 웹팩 개발 서버에서 api 서버로 프록싱
  - webpack-dev-server는 `proxy` 속성을 지원
- 예시에서는 localhost의 8081 포트지만, 실 도메인을 추가

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
