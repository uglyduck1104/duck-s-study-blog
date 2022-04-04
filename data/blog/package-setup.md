---
title: 외부 패키지 관리
date: '2022-04-04'
tags: ['npm']
draft: false
summary: external package setup
layout: PostSimple
authors: ['default']
---

# 패키지 설치

## CDN 이용

- CDN(컨텐츠 전송 네트워크)으로 제공하는 라이브러리를 직접 import 함

```javascript
<script src="http://unpkg.com/react@16/umd/react.developement.js"></script>
```

- 리액트의 주소를 html에서 로딩
- CDN 서버 장애로 인해 해당 라이브러리를 import한 웹 어플리케이션의 비정상 동작 야기

## 직접 다운로드

- 예기치 못한 CDN 서버 장애를 방지하지만 추후에 최신 버전으로 교체 시 문제 발생
- 버전에 따른 하위 호환성 여부 확인에 따른 Side Effect(human error) 발생

## NPM 이용

- 한 곳에서 업데이트할 수 있고 하위 호환되는 안전한 버전만 다운 받을 수 있음
- npm install 명령어 사용

```shell
$ npm install react
```

```javascript
{
  "dependencies": {
    "react": "^18.0.0"
  }
}
```

- "dependencies"가 추가되고, npm install \<keyword>로 다운로드 받은 목록이 출력됨
- "react"의 버전 정보 확인 가능

## NPM 유의적 버전

- 버전 번호를 관리하기 위한 규칙이 필요
- 정해져 있는 패키지의 버전을 효율적으로 관리하기 위해 유의적 버전을 사용
- Sementic Version이라고 하고 버전 규칙은 아래와 같다
  - 주 버전(Major Version): 기존 버전 `호환되지 않게 변경`
  - 부 버전(Minor Version): 기존 버전 호환, `기능 추가`
  - 수 버전(Patch Version): 기존 버전 호환, `버그 수정`

## 버전의 범위

- 특정 버전보다 높거나 낮을 경우
  ```
  > 1.2.3
  >= 1.2.3
  <1.2.3
  <=1.2.3
  ```
- 틸드(~)와 캐럿(^) 사용
  - 틸트
    - 마이너 버전이 명시되어 있으면 패치 버전을 변경
    ```
    ~1.2.3 (1.2.3부터 1.3.0 미만까지 포함)
    ~0 (0.0.0부터 1.0.0 미만까지 포함)
    ```
    - 마이너 버전이 없으면, 마이너 버전 갱신
  - 캐럿
    - 정식버전일 경우 마이너와 패치 버전 변경
    ```
    ^1.2.3 (1.2.3부터 2.0.0 미만까지 포함)
    ^0 (0.0.0부터 0.1.0미만까지 포함)
    ```
    - 정식버전 미만인 0.x 버전은 패치만 갱신
  - 틸트 버전보다는 캐럿 버전 채택(node.js의 경우)
    - ~0과 같은 틸트 버전에는 하위 호환성을 지키지 못하는 문제가 발생하므로 캐럿을 주로 사용
    - 정식 릴리즈 전 하위 호환성 유지 가능

### 버전 범위 확인하기

```shell
$ npm view <keyword> versions # 확인하고 싶은 dependency의 모든 버전 출력
$ cat node_modules/<keyword>/package.json # 설치된 dependency의 버전 확인
```

- npm이 어떤식으로 버전 관리하는지 확인

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
