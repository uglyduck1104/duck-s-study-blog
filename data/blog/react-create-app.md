---
title: Create React App
date: '2022-04-27'
tags: ['react']
draft: false
summary: 개발 서버에 접근하거나 변동사항에 대해 새로고침을 해주거나 애플리케이션 안에 CSS 파일을 로드할 수 있게 하는 등 별도로 설정을 하지 않아도 편의성을 위한 설정이 되어있음
layout: PostSimple
authors: ['default']
---

# CRA(Create React App)

- 이전의 예제로 봤던 방식은 스크립트를 import하는 방식으로 React library를 로드했음
- 개발 서버에 접근하거나 변동사항에 대해 새로고침을 해주거나 애플리케이션 안에 CSS 파일을 로드할 수 있게 하는 등 별도로 설정을 하지 않아도 편의성을 위한 설정이 되어있음
- 주로 리액트 어플리케이션을 만들때 사용함

## Quick Start

### Node.js 설치

- OS에 맞는 node.js 설치가 선행되어야 함
  - [설치 페이지](https://nodejs.org/ko)
    - LTS -> 안정적인 버전(주로 node.js 서버 개발 시 사용)
    - Current -> 최신 버전

### Node.js & npx 확인

```shell
$ node -v
$ npx -v
```

- 노드 버전 및 npx 버전 출력 확인이 된다면 준비가 완료됨

## npx command로 프로젝트 생성

```shell
npx create-react-app my-app #--template typescript 지원
cd my-app
npm start
```

- `npx create-react-app [프로젝트 명]`을 실행

## npm init 으로 프로젝트 초기화

```shell
npm init react-app my-app
```

- 이미 생성된 프로젝트에 npm init command로 react와 관련된 의존성들을 자동으로 설치해 줌

### package.json 확인하기

```json lines
{
  "name": "react-for-beginners",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.16.4",
    "@testing-library/react": "^13.1.1",
    "@testing-library/user-event": "^13.5.0",
    "react": "^18.1.0",
    "react-dom": "^18.1.0",
    "react-scripts": "5.0.1",
    "web-vitals": "^2.1.4"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  }
  //...
}
```

- create-react-app 팀에 의해 만들어짐
- 따로 설치하지 않았지만 프로젝트를 생성하면서 기본 설정이 되어 있음

## 실행

```shell
$ npm start
```

- create-react-app을 통해 어플리케이션을 만들었을 때 초기 버전을 확인할 수 있음
- 개발 서버 실행 시 기본 설정된 브라우저 자동 실행
- 변경 사항에 대해서 auto Reloading 지원

`index.js`

```javascript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

- 초기의 index.js 파일은 전에 다루었던 구조와 동일하게 `root`라는 아이디를 가진 ReactDOM을 만들고 하위에 App 컴포넌트를 감싸는 레이아웃임
- ES6 import 구문으로 `React`와 `ReactDOM`을 호출함

## module.css

`App.js`

```javascript
import Button from './Button'
import styles from './App.module.css'

function App() {
  return (
    <div>
      <h1 className={styles.title}>Welcome Back!</h1>
      <Button text={'Continue'} />
    </div>
  )
}
export default App
```

- [component_name].module.css 패턴의 파일명을 object로 받아서 css의 클래스명을 className 프로퍼티에 대응
  - 해당 패턴으로 `module.css` 파일을 만들어 순수 css로 작성해서 관리할 수 있음
  - 인라인 스타일로도 작성이 가능하지만 컴포넌트에 해당하는 독립적인 형태로 모듈을 관리할 수 있음
  - 클래스명 뒤에 hash값이 붙으므로, 클래스 이름간의 중복의 문제를 피할 수 있음

## PropTypes

### 설치

```shell
npm install prop-types
```

- props의 type을 체크하기 위한 의존성을 추가

### 적용

```javascript
Button.propTypes = {
  text: PropTypes.string.isRequired,
}
```

- Component.propTypes
  - props는 text key를 가진 value가 넘어와야하는데, 해당 value는 string type이며 필수 값이어야 함
- unpkg CDN으로 호출했을때와는 다르게 자동 완성을 지원
  - node_modules에 설치되어있는 prop-types의 `index.d.ts` 파일에 접근 가능하며, 어떤 타입을 지원하는 지 확인하기 용이
- Button 컴포넌트의 `prop type`을 지정하여 해당 타입에 맞지 않는다면, error level의 log를 출력
  > CRA(create-react-app)으로 작업할 때의 포인트는 "분할"과 "정복" ⭐️

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
