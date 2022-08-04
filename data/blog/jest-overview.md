---
title: Jest, TDD
date: '2022-08-04'
tags: ['Jest', 'Testing', 'TDD']
draft: false
summary: 단지 라이브러리에서 그치는 것이 아니라, 작성 방식에 있어 특정한 관행을 권장하는 테스트 라이브러리
layout: PostSimple
authors: ['default']
---
# React Testing Library(RTL)

## 테스트 라이브러리와 Jest 소개

- 단지 라이브러리에서 그치는 것이 아니라, 작성 방식에 있어 특정한 관행을 권장함

### 철학

1. 사용자 사용 방식으로 소프트웨어 테스트
    - 소프트웨어가 원래대로 작동하는가
2. 테스트 ID를 사용하는 대신 스크린 리더와 다른 보조 기술로 요소를 찾음

### RTL

- 테스트를 위한 `가상 DOM`을 제공
    - 브라우저 없이 가상 DOM을 이용해 테스트가 가능함

### Jest

- 테스트 러너
    - 테스트를 찾고 실행하고 테스트 통과 여부를 결정하는 역할을 함

## 테스트 라이브러리를 사용한 테스트

### CRA(Create-React-App)를 통해 프로젝트 생성

```bash
$ npx create-react-app
```

- npx 명령어를 통해서 프로젝트를 생성
- react app을 최초 생성하는데, 필요한 구성과 의존성을 자동으로 생성해주고 개발 웹 서버와 테스팅 라이브러리, webpack과 babel을 제공해줌
- 매번 create-react-app 템플릿을 다운로드할 필요없이 npx를 실행할 때마다 최신 버전을 가져옴

### 기본으로 제공하는 테스트 스크립트 실행

```bash
$ npm test
```

- 프로젝트가 설치된 경로에서 해당 스크립트를 실행해서 테스트를 진행

<img alt="스크린샷 2022-08-03 오후 11.58.20.png" src="/static/images/jest_overview_1.png" width="400"/>

- 명령형 콘솔을 통해 watch mode에 진입하여 테스트를 진행

<img alt="스크린샷 2022-08-03 오후 11.58.20.png" src="/static/images/jest_overview_2.png" width="400"/>

- 테스트 결과를 콘솔에서 확인할 수 있고, 안내에 따라 진행할 수 있음

### `App.test.js`

```jsx
import { render, screen } from '@testing-library/react';
import App from './App';

test('renders learn react link', () => {
  render(<App />);
  const linkElement = screen.getByText(/learn react/i);
  expect(linkElement).toBeInTheDocument();
});
```

- npm test를 실행했을 때, 실행되던 테스트를 확인할 수 있음
- `render(<App />);`
    - 테스트 함수에서 첫 번째로 render 메서드를 실행함
    - 인수로 제공하는 JSX에 관한 가상 DOM을 생성
- `screen.getByText(/learn react/i);`
    - screen global 객체로 가상 DOM에 접근
    - getByTest 메서드를 통해 표시되는 모든 텍스트를 기반으로 DOM 요소에서 찾음
        - 정규 표현식을 통해 learn react라는 문자열을 제한함
- `expect(linkElement).toBeInTheDocument();`
    - 단언문을 통해, 테스트 성공과 실패의 원인을 명시

### 테스트가 실패하는 경우

`App.test.js`

```jsx
import { render, screen } from '@testing-library/react';
import App from './App';

test('renders learn react link', () => {
  render(<App />);
  const linkElement = screen.getByText(/learn testing library/i);
  expect(linkElement).toBeInTheDocument();
});
```

- `screen.getByText(/learn testing library/i);`

  <img alt="스크린샷 2022-08-04 오전 12.09.36.png" src="/static/images/jest_overview_3.png" width="800"/>

  <img alt="스크린샷 2022-08-04 오전 12.09.46.png" src="/static/images/jest_overview_4.png" width="800"/>

    - `learn testing library`라는 문자열을 못 찾는 경우 아래와 같은 테스팅 실패 결과를 확인할 수 있음

## Jest와 Jest-DOM 단언문

- `expect(linkElement).toBeInTheDocument();`
    - Jest에서 전역 메서드인 expect 메서드로 시작
    - expect `argument`
        - 테스트를 확인하기 위한 인자를 넘겨줌
    - `toBeInTheDocument()`
        - 매처(Matcher)를 사용해 단언의 유형을 지정
        - Jest DOM에서 온 메서드
        - 개수를 확인하는 매처의 경우, 넘어오는 인수를 사용
- `jset-dom`
    - `src/setupTest.js` 파일을 사용해 각 테스트 전에 Jest DOM을 가져옴
        - DOM 기반은 매처를 제공
    - 모든 테스트에서 jest-dom 매처를 사용할 수 있음

## Jest: Watch 모드와 테스트가 작동하는 방식

- RTL이 주는 이점
    - 컴포넌트를 가상 DOM으로 렌더링할 수 있음
    - 가상 DOM을 검색할 수 있음
    - 가상 DOM과 상호 작용할 수 있음
- Test Runner의 필요성
    - 테스트를 찾고 실행하며 단언할 무언가가 필요한데, Jest를 이용해 도움을 받을 수 있음
- `npm test` 명령어를 실행함으로써, `watch mode`로 진입해서 결과를 쉽게 인지할 수 있음
- `Watch Mode`
    - Jest를 실행하는 방법으로써, 마지막 커밋이후 파일의 모든 변경 사항을 확인하고 연관된 테스트만 실행
    - 실행할 테스트가 존재하지 않는다면 테스트를 실행하지 않음
- `테스트 통과나 실패를 아는 법`
    - `2개의 인수`를 가진 전역 테스트 메서드 제공
    - `첫 번째` 인수
        - 테스트의 문자열 설명
        - 테스트가 실패했을 때, 어떤 테스트가 실패했는지 알려줌
    - `두 번째` 인수
        - 테스트 함수로서 성공과 실패를 결정하기 위해 실행되는 함수
        - 테스트 함수를 실행할 떄 에러가 발생하면 실패함

## TDD: 테스트 주도 개발

- 코드 작성 전에 테스트를 작성하고 테스트에 통과하도록 코드를 작성하는 방법
- `레드-그린` 테스트
    - 코드 작성 전에 테스트에 실패하는 `레드` 테스트를 먼저 실행
    - 코드 작성 후에는 통과하는 테스트로 그린 테스트를 확인

### TDD를 하는 이유

1. 테스트를 작성하는 것이 프로세스의 한 부분(프로세스의 일환)
2. 효율적인 작동 방식
    - 테스트를 작성해놓으면 변경 사항이 생길 때마다 자동 회귀 테스트가 가능해짐
    - 수동으로 테스트할 필요가 없음

## React 테스팅 라이브러리(RTL)의 철학

- 테스트를 위한 가상 DOM을 생성하고, DOM과 `상호 작용`하기 위한 유틸리티 제공
- 브라우저 없이 테스트 가능

### 테스트 종류

- `유닛 테스트`
    - 함수나 별개의 React 컴포넌트 코드의 한 유닛 혹은 단위 테스트
- `통합 테스트`
    - 여러 유닛이 함께 작동하는 방식을 테스트해서 유닛 간의 상호 작용을 테스트
- `기능 테스트`
    - 소프트 웨어의 특정 기능을 테스트
- `인수 테스트`(E2E 테스트)
    - 애플리케이션이 연결된 서버가 필요하고 실제 브라우저 위에서 동작함

## 기능 테스트 vs 유닛 테스트

`유닛 테스트`

- 테스트를 최대한 격리
- 함수나 컴포넌트를 테스트할 때 의존성 표시
- 실패를 쉽고 정확하게 파악할 수 있음
- `단점`
    - 사용자가 소프트웨어와 상호 작용하는 방식과 조금 덜 밀접하게 연결되어 있음
    - 리팩토링으로 인해 테스트가 실패할 수 있음

`기능 테스트`

- 테스트하는 `특정 동작`과 `유저 플로우`와 연관된 모든 단위를 포함
- 사용자가 소프트웨어와 상호 작용하는 방식이 밀접함
- 테스트에 통과하면 사용자에게 문제 없고, 실패하면 사용자에게 문제가 발생할 가능성이 높아짐
- `견고한 테스트 방식`으로 인해, 코드 작성 방식을 리팩토링해도 동작이 동일하면 테스트에 실패하지 않음
- `단점`
    - 실패한 테스트를 `디버깅하기 어려움`
    - 코드가 유닛테스트처럼 직접적인 연관이 없으므로 코드상에서 테스트 실`패의 원인`을 정확히 알 수 없음

### TDD vs BDD

- `BDD`
    - 다양한 역할 간의 협업이 전제 조건
    - 서로 다른 그룹이 상호 작용하는 방식에 관한 프로세스 정의

## 테스팅 라이브러리와 접근성

- 접근성으로 요소를 찾거나 요소를 찾을 수 있는 스크린 리더와 같은 보조 기술로 요소 기술을 찾음
    - 테스팅 라이브러리 가이드 - [참고 링크](https://testing-library.com/docs/queries/about/#priority)
- 가상 DOM에서 요소를 찾을 때, 어떤 우선순위로 사용해야 할 지 알려줌

### 우선 순위

1. 누구나 액세스 가능한 쿼리(`수동 쿼리`)
    - 마우스를 사용하고 있고 화면을 시간적으로 보고 있는 사람들
    1. `getByRole`
        - 페이지에서의 요소 역할을 식별
    2. `getByLabelText`
        - 스크린 리더가 액세스 할 수 있는 것
    3. `getByPlaceholderText`
        - 입력 요소에 관련된 것
    4. `getByText`
        - 대화형이 아닌 디스플레이
    5. `getByDisplayValue`
        - 폼 관련해서 사용됨
2. 어느 것도 사용하지 못한다면 시맨틱 쿼리 사용
    - `getByAltText`
        - 지정된 상위 요소의 텍스트를 가져옴
        - 이미지에서 사용
    - `getByTitle`
        - 타이틀과 연관된 요소
3. `Test IDs`
    - 꼭 필요한 경우에만 사용

> **Referenced**

- [Testing React with Jest and React Testing Library (RTL)](https://www.udemy.com/course/understanding-typescript/)