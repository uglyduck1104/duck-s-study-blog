---
title: 기본 개념 이해
date: '2022-10-26'
tags: ['NodeJS']
draft: false
summary: NodeJS 기본 개념 이해
layout: PostSimple
authors: ['default']
---

# 기본 개념 이해

## Node 서버 생성

- Node.js 애플리케이션을 구성하는 루트 경로에 `server.js` 혹은 `app.js`로 파일 생성

### `app.js`

```jsx
const http = require('http')

const server = http.createServer((req, res) => {
  console.log(req)
})

server.listen(3000)
```

- `Node.js`에서 기본으로 탑재된 코어 모듈을 불러옴
  - `http` 모듈을 `const` 키워드로 불러와서 절대 변경되지 않는 값으로 할당
  - `Node.js`는 전역으로 노출하는 특성이 있으므로 실행하는 모든 파일에서 `require` 키워드를 사용할 수 있음
  - 파일 경로는 반드시 상대경로인 `./`나 절대 경로의 경우 `/`로 시작
- `http.createServer((req, res) ⇒ {...})`
  - `createServer` 메서드는 서버를 생성할 때 필요한 메서드로서, `requestListener`를 인수로 가짐
    - `requestListener`는 들어오는 모든 요청을 실행하는 기능
  - 첫번째 인수는 요청에 대한 데이터, 두번째 인수는 응답에 대한 데이터를 받을 수 있음
- `server.listen()`
  - Node.js가 스크립트를 바로 종료하지 않고 계속 실행
  - 전달하는 인수는 포트 번호, 호스트 이름을 주로 사용

### 실행

```bash
$ node app.js
```

## Node의 라이프 사이클 및 이벤트 루프

### Node.js 라이프 사이클

<img alt="node_basic_1" src="/static/images/node_basic_1.png" width="700"/>

1. `node app.js` 파일 실행
2. 스크립트 실행
3. Node.js 파일 스캔

- 전체 코드를 읽고 실행하며, 프로그램은 꺼지지 않은 상태

4. 이벤트 루프

- Node.js가 관리하는 이벤트 루프는 작업이 남아 있는 한 계속해서 작동하는 루프 프로세스
- 이벤트 리스너가 있는 한 계속 동작함
- 코어 노드 어플리케이션은 이벤트 루프에 의해 관리됨
- 자바스크립트는 단일 스레드 방식으로써, 실행 중인 컴퓨터에서 전체 노드 프로세스가 하나의 스레드를 사용함

5. `process.exit`

- 말 그대로 이벤트 루프를 종료해 프로그램을 종료 처리함
- 대개는 서버를 중지하지 않으므로 `process.exit`를 호출할 일이 없음

### `app.js`

```jsx
const http = require('http')

const server = http.createServer((req, res) => {
  console.log(req)
  process.exit()
})

server.listen(3000)
```

## 요청과 응답

```jsx
const http = require('http')

const server = http.createServer((req, res) => {
  console.log(req, res)
  res.setHeader('Content-Type', 'text/html')
  res.write('<html>')
  res.write('<head><title>My First Page</title></head>')
  res.write('<body><h1>Hello from my Node.js Server</h1></body>')
  res.write('</html>')
  res.end()
})

server.listen(3000)
```

### 요청(request)

- 요청 객체를 콘솔로 확인해보면 아주 복잡한 객체의 구조를 가지고 있음
- 요청 헤더의 정보, 응답 유형, 암호화 응답등의 정보를 확인할 수 있음
- 주로 `req.url`, `req.method`, `req.headers`를 사용함
  - `url` → 호스트 다음에 붙는 모든 주소
  - `method` → HTTP Method 정보
  - `headers` → 요청 헤더에 대한 정보

### 응답(response)

- 새로운 헤더를 설정할 수 있음
  - 응답의 일부가 될 콘텐츠 유형은 `HTML`이라는 일련의 메타 정보를 전달할 수 있음
- `HTML` 코드를 `write` 메서드를 통해 여러 라인에 걸쳐 작성할 수 있음
- 응답의 생성이 끝난 뒤에는 노드에 알려주기 위해 `end`를 호출함

## 라우터 요청

```jsx
const http = require('http')

const server = http.createServer((req, res) => {
  const url = req.url
  if (url === '/') {
    res.setHeader('Content-Type', 'text/html')
    res.write('<html>')
    res.write('<head><title>My First Page</title></head>')
    res.write(
      '<body><form action="/message" method="POST"><input type="text" name="message"><button type="submit">Send</button></form></body>'
    )
    res.write('</html>')
    return res.end()
  }
  res.setHeader('Content-Type', 'text/html')
  res.write('<html>')
  res.write('<head><title>My First Page</title></head>')
  res.write('<body><h1>Hello from my Node.js Server</h1></body>')
  res.write('</html>')
  res.end()
})

server.listen(3000)
```

- 요청과 응답을 연결하기 위한 라우트를 구성
- `URL`의 파싱을 위해 상수에 저장하고 조건문을 추가해 URL이 `/`일때, 입력 양식과 신규 요청 버튼을 통해 `POST Method`로 요청이 전달될 URL(`/message`)을 사용해서 폼 양식을 제출하고 `end` 메서드를 통해 종료
  - 메서드를 명시하지 않으면 기본적으로 `GET` 요청으로 처리

## 요청 리디렉션

```jsx
const http = require('http')
const fs = require('fs')

const server = http.createServer((req, res) => {
  const url = req.url
  const method = req.method
  if (url === '/') {
    res.setHeader('Content-Type', 'text/html')
    res.write('<html>')
    res.write('<head><title>My First Page</title></head>')
    res.write(
      '<body><form action="/message" method="POST"><input type="text" name="message"><button type="submit">Send</button></form></body>'
    )
    res.write('</html>')
    return res.end()
  }
  if (url === '/message' && method === 'POST') {
    fs.writeFileSync('message.txt', 'DUMMY')
    res.statusCode = 302
    res.setHeader('Location', '/')
    return res.end()
  }
  res.setHeader('Content-Type', 'text/html')
  res.write('<html>')
  res.write('<head><title>My First Page</title></head>')
  res.write('<body><h1>Hello from my Node.js Server</h1></body>')
  res.write('</html>')
  res.end()
})

server.listen(3000)
```

- 조건절을 추가해 `url`이 `message`이고 `method`가 `POST` 요청일 경우를 추가
  - 새로운 파일을 생성하여 유저가 입력한 메시지를 저장하는 로직을 구성
- Node.js 코어 모듈인 fs(파일 시스템)를 사용해 새로운 파일을 생성하고 내용을 작성
  - `writeFile` 혹은 `writeFileSync`를 사용해파일로 가는 경로를 취하고 `app.js`와 동일한 위치에 `txt` 파일을 생성
  - `res.statusCode`를 `302`로 설정한 뒤 간단하게 `setHeader`에 위치를 지정
  - `res.end`를 호출해 종료임을 명시

## 요청 본문 분석

### 데이터 스트림 & 버퍼

![node_basic_2](/static/images/node_basic_2.png)

- 들어오는 요청이 데이터 스트림으로 보내지는데, 이는 JavaScript가 일반적으로 알고 있는 구조체이며 Node.js에서 주로 사용됨
- 노드가 많은 양의 요청을 한 청크씩 읽다가 어느 시점에 특정 청크들을 다룰 수 있음(버퍼 - `Buffer`)
  - 업로드된 파일의 경우, 분석하는게 일반적인 요청에 비해 시간이 소요되므로 파일 전체가 분석 완료되고 전부 업로드 되기까지 기다릴 필요없이 일부 청크를 이용할 수 있음
- `버퍼`는 여러 개의 청크를 보유하고 파싱이 끝나기 전에 작업할 수 있도록 함

```jsx
const fs = require('fs')

const requestHandler = (req, res) => {
  const url = req.url
  const method = req.method
  if (url === '/') {
    res.setHeader('Content-Type', 'text/html')
    res.write('<html>')
    res.write('<head><title>Enter Page</title></head>')
    res.write(
      '<body><form action="/message" method="POST"><input type="text" name="message"><button type="submit">Send</button></form></body>'
    )
    res.write('</html>')
    return res.end()
  }
  if (url === '/message' && method === 'POST') {
    const body = []
    req.on('data', (chunk) => {
      console.log(chunk)
      body.push(chunk)
    })
    req.on('end', () => {
      const parsedBody = Buffer.concat(body).toString()
      const message = parsedBody.split('=')[1]
      fs.writeFileSync('message.txt', message)
    })
    res.statusCode = 302
    res.setHeader('Location', '/')
    return res.end()
  }
  res.setHeader('Content-Type', 'text/html')
  res.write('<html>')
  res.write('<head><title>My First Page</title></head>')
  res.write('<body><h1>Hello from my Node.js Server!</h1></body>')
  res.write('</html>')
  res.end()
}
```

- request 객체를 통해 이벤트 리스너를 등록하고 on 메서드를 이용해 특정 이벤트를 사용
- `data` 이벤트 리스너로 새 청크가 읽힐 준비가 될 때마다 데이터 이벤트가 발생하는 데에 버퍼가 도움을 줌
  - 두 번째 인자로 모든 데이터에 실행될 함수를 정의하고 청크와 상호 작용할 수 있음
  - 요청 본문을 분석하기 위해 `body` 배열에 `chunk`를 할당
- `end` 이벤트 리스너로 들어온 요청 데이터를 분석한 후에 발생하는 트리거를 이용
  - 전역에서 사용 가능한 버퍼 객체를 사용해 받은 청크 배열을 주입함
  - parsedBody라는 버퍼에 toString을 호출해 문자열로 전환하여 들어오는 데이터를 파싱함
  - `message="BUFFREDTEXT"`로 문자열 변환한 후 split method를 사용해 `=`을 기준으로 Text를 추출

## 이벤트 기반 코드 실행의 이해

- 응답은 이벤트 리스너 실행이 끝났다는 의미가 아니라, 응답이 온 후에도 이벤트 리스너는 계속 실행됨
  - 동시 이벤트 리스너에 응답에 영향을 줄 수 있는 어떠한 처리를 하는 건 잘못된 설정임
- `Node.js`는 함수를 함수 안에 넣으면 안에 넣은 함수를 나중에 실행하는데 이를 비동기식이라고 함
- 요청 분석이 완료되면 `req.on` 또는 `end` 이벤트가 자동으로 실행됨

## 블로킹 및 논블로킹 코드

```jsx
const fs = require('fs')

const requestHandler = (req, res) => {
  const url = req.url
  const method = req.method
  if (url === '/') {
    res.setHeader('Content-Type', 'text/html')
    res.write('<html>')
    res.write('<head><title>Enter Page</title></head>')
    res.write(
      '<body><form action="/message" method="POST"><input type="text" name="message"><button type="submit">Send</button></form></body>'
    )
    res.write('</html>')
    return res.end()
  }
  if (url === '/message' && method === 'POST') {
    const body = []
    req.on('data', (chunk) => {
      console.log(chunk)
      body.push(chunk)
    })
    req.on('end', () => {
      const parsedBody = Buffer.concat(body).toString()
      const message = parsedBody.split('=')[1]
      fs.writeFile('message.txt', message, (error) => {
        res.statusCode = 302
        res.setHeader('Location', '/')
        return res.end()
      })
    })
  }
  res.setHeader('Content-Type', 'text/html')
  res.write('<html>')
  res.write('<head><title>My First Page</title></head>')
  res.write('<body><h1>Hello from my Node.js Server!</h1></body>')
  res.write('</html>')
  res.end()
}
```

- `writeFileSync` vs `writeFile`
  - `Sync` 키워드는 동기화를 의미하며 파일이 생성되기 전까지 코드 실행을 막는 메서드
    - 파일이 완료될 때까지 코드의 다음 라인이 실행되지 않으므로, 가급적이면 작은 파일에서 해당 키워드를 적용해야 함
  - `writeFile`
    - 경로와 데이터를 받을 수 있고 세 번째 인수인 콜백까지도 포함함
      - 에러가 발생하는 경우 콜백에서 에러 응답등을 반환해 파일 작업이 완료된 경우에만 전송되도록 기존의 응답 객체 로직을 콜백 함수에 넣어줌
    - Node.js는 암묵적으로 이벤트 리스너를 등록시킴
- 이벤트 드리븐 아키텍처에서는 `Node.js`에게 작업을 진행핟도록 지시하며 이후 `Node.js`가 해당 프로세스를 멀티 스레딩을 사용하는 운영체제에 전달함
- 이벤트 콜백을 파악하기 위해 이벤트 루프를 계속하면서 코드 실행을 막지 않도록하고 운영체제에서 작업이 끝난 뒤에는 항상 복귀하는 식의 구조를 지님

## Node.js 백그라운드 확인

> Node.js가 코드를 정확히 어떻게 실행해서 계속 성능을 유지하며 파일을 다루는 등 오래 걸리는 작업을 수행할까?

### 싱글 스레드

![node_basic_3](/static/images/node_basic_3.png)

- 하나의 Javascript 스레드만 사용하는 자바스크립트는 각 요청마다 스레드를 지정할 수 없으므르 결국 모두 하나의 스레드에서 실행되며, 이는 곧 보안 상의 문제가 제기됨
  - 또, 요청 A를 아직 처리 중이라면 요청 B는 처리할 수 없는지의 물음이 발생
- `성능 문제의 해결`
  - 요청으로 파일을 다루고 있을 때 새로 들어오는 두 번째 요청은 아직 처리할 수 없기 때문에 자바스크립트는 이벤트 루프에 의한 이벤트 콜백을 다룸
  - 즉, 특정 이벤트가 일어났을 때 바로 이벤트 루프가 해당 코드를 실행함
    - 빠르게 끝낼 수 있는 코드를 포함한 콜백만을 다룸
  - 파일 시스템 연산 등의 오래 걸리는 작업은 워커풀에 보내져서 자바스크립트 코드로부터 완전히 분리되어 다른 스레드에서 작동함
    - 워커가 작업을 마치면 이벤트 루프가 이벤트와 콜백을 책임지므로 결국 이벤트 루프에 들어감

### 이벤트 루프

![node_basic_4](/static/images/node_basic_4.png)

- 이벤트 루프의 역할
  - `Node.js`에 의해 실행되어 모든 콜백을 처리함

1. 루핑을 계속하는 루프로 새로운 반복이 시작될 때마다 실행해야 하는 타이머 콜백이 있는지 확인함
   - 타이머가 끝나면 실행할 함수를 체크해서 다른 콜백을 실행
   - 아직 처리되지 않은 콜백이 너무 많이 있다면 루프 반복하지 않고 남은 콜백을 다음 반복에서 실행할 수 있도록 지연시킴

2. 콜백을 모두 처리한 후, `Poll` 단계에 진입해 새로운 `I/O` 이벤트를 찾아 최대한 해당 이벤트의 콜백을 빨리 실행
3. `Check` 단계에서 `setImmediate` 콜백이 실행됨
   - 반드시 열린 콜백이 모두 실행된 다음에 실행되며, 보통 타이머 콜백보다 1ms만큼 빠르지만 현재 주기가 끝나거나 적어도 현 반복에 열린 콜백을 처리한 후에 일어남

4. `Close Callbacks`이 모두 실행된 후 만약 이번 코드에서는 등록하지 않았으나 닫힌 이벤트를 등록했다면 해당 시점에 콜백을 실행함
5. 프로그램 종료
   - 등록한 이벤트 핸들러가 남지 않았는지(`refs===0`) 내부적으로 열린 이벤트 리스너를 추적함
   - 일반적으로는 노드 웹 서버 프로그램을 종료할 일이 없음

## Node 모듈 시스템 사용

- 관심사를 분리하기 위해, `routes.js`와 `app.js` 파일을 분리해야 함
- `module.exports` 키워드를 사용해 분리

### `routes.js`

```jsx
const fs = require('fs')

const requestHandler = (req, res) => {
  const url = req.url
  const method = req.method
  if (url === '/') {
    res.setHeader('Content-Type', 'text/html')
    res.write('<html>')
    res.write('<head><title>Enter Page</title></head>')
    res.write(
      '<body><form action="/message" method="POST"><input type="text" name="message"><button type="submit">Send</button></form></body>'
    )
    res.write('</html>')
    return res.end()
  }
  if (url === '/message' && method === 'POST') {
    const body = []
    req.on('data', (chunk) => {
      console.log(chunk)
      body.push(chunk)
    })
    return req.on('end', () => {
      const parsedBody = Buffer.concat(body).toString()
      const message = parsedBody.split('=')[1]
      fs.writeFile('message.txt', message, (error) => {
        res.statusCode = 302
        res.setHeader('Location', '/')
        return res.end()
      })
    })
  }
  res.setHeader('Content-Type', 'text/html')
  res.write('<html>')
  res.write('<head><title>My First Page</title></head>')
  res.write('<body><h1>Hello from my Node.js Server!</h1></body>')
  res.write('</html>')
  res.end()
}

module.exports = {
  handler: requestHandler,
  someText: 'Some hard coded text',
}
```

- 새로운 함수 `requestHandler` 함수를 생성해 `req`, `res`를 인수로 가지면 `http.createServer` 함수를 대체할 수 있음
- `Node.js`에 의해 전역으로 노출할 키워드 혹은 객체로 내보내기 속성을 사용할 수 있음
- `app.js`에서 호출할 상수 `requestHandler`를 `modules.exports` 통해 키-값 쌍의 형태로 내보냄

### `app.js`

```jsx
const http = require('http')

const routes = require('./routes')

console.log(routes.someText)

const server = http.createServer(routes.handler)

server.listen(3000)
```

- `require` 키워드를 이용해 `./` 로컬 경로의 `routes` 파일을 불러옴
- `Node.js`가 `app.js`와 같은 폴더에서 `routes.js` 파일을 찾아 `module.exports`에 무엇이 등록되어 있는지 봄
- `Node.js` 모듈 시스템은 파일 내용의 캐시가 저장되고, 외부에서 수정할 수 없음

> **Referenced**

- [NodeJS - The Complete Guide (MVC, REST APIs, GraphQL, Deno)](https://www.udemy.com/course/nodejs-mvc-rest-apis-graphql-deno/)
