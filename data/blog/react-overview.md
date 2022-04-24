---
title: React
date: '2022-04-22'
tags: ['react']
draft: false
summary: 사용자 인터페이스를 만들기 위한 JavaScript 라이브러리
layout: PostSimple
authors: ['default']
---

# React

<div className="w-6/12 my-0 mx-auto">![react](/static/images/react_1.png)</div>
> React는 페이스북이 만든 자바스크립트 라이브러리로써, 자바스크립트 라이브러리중에 가장 큰 생태계를 가지고 있고 전세계에서 상위 1만 개 웹 사이트 중에서 약 44.23%는 React를 기반으로 만들어져있음

## 사용 이점

- SPA(Single Page Application)를 개발할 때, 좋은 선택지
  - 페이지를 다시 로드하거나, 새로 고쳐서 특정 데이터를 로드하거나 업데이트할 필요 없음
- Virtual DOM, JSX, 상태 관리 등 재사용 가능한 UI 컴포넌트를 사용해서 개발자의 개발 경험을 높여줌
- 비교적 최근(2018)에 react hook을 공식적으로 지원함으로써 Class component의 단점을 보완하고 상태 값을 관리하는데 보다 유용한 기능을 제공

## 설치

```shell
$ npx create-react-app my-app
$ cd my-app
$ npm start
```

- webpack이나 babel 같은 도구를 사용하지만, 별도의 설정 없이도 동작함
- 프로덕션 배포 시, build script인 `npm run build`를 실행하면 build 파일이 생성되고 최적화된 javascript 파일을 확인할 수 있음

## JSX 사용 예제

- Javascript를 확장한 문법
  - 템플릿 언어와 비슷하지만 Javascript의 모든 기능을 포함하고 있음
- HTML보다는 javascript에 가까우므로 camelCase 프로퍼티 명명 규칙 사용
- `class` -> `className`, `tabindex` -> `tabIndex`

### XSS(Cross-Site-Scripting) 방지

- JSX에 삽입된 모든 값은 렌더링하기 전에 Escape처리되어 애플리케이션에서 명시적으로 작성되지 않은 내용은 주입되지 않음
  - 즉, 모든 항목은 렌더링 되기 전에 문자열로 변환

### 객체 표현

```javascript
// JSX
const element = <h1 className="greeting">Hello, world!</h1>
// createElement() 사용
const element = React.createElement('h1', { className: 'greeting' }, 'Hello, world!')
```

- JSX를 사용하면 createElement()를 사용해서 구현하는 것보다 가독성이 높음
- 하지만, 브라우저는 JSX 표현식을 읽지 못하기 때문에 Babel을 사용해서 JSX 표현식을 컴파일함
  - Babel로 컴파일하면 다음과 같은 객체를 생성
    ```javascript
    const element = {
      type: 'h1',
      props: {
        className: 'greeting',
        children: 'Hello, world!',
      },
    }
    ```
    - React는 위의 객체를 읽어서, DOM을 구성하고 최신 상태로 유지하는데 사용함
      > ES6 및 JSX 코드가 올바르게 표현되려면 `Babel`설정이 권장됨

> **Referenced**

- https://ko.reactjs.org/
- https://trends.builtwith.com/
- https://www.monocubed.com/blog/why-use-react/
