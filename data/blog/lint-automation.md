---
title: Lint Automation
date: '2022-04-13'
tags: ['Lint']
draft: false
summary: Lint의 자동화 구성
layout: PostSimple
authors: ['default']
---

# 자동화

- 린트는 코딩할 때마다 수시로 실행해야 함
- 그러므로, 자동화 구성이 필수로 설정되어야 하는데 git hook을 이용하는 방법과 editor 확장 도구를 이용하는 방법이 있음
  - editor를 이용한 부분은 사용자마다 다르므로, 본인 editor 혹은 IDE에서 eslint 확장 도구를 다운 받아서 설정할 것을 권장

## Husky를 이용한 검사

- 소스 트래킹 도구인 깃을 사용한면 깃 훅을 보통 사용하는데, 깃 훅을 좀 더 편하게 설정할 수 있는 도구

### 설치

```shell
$ npm install husky
```

### 적용

`package.json`

```json lines
{
  "husky": {
    "hooks": {
      "pre-commit": "eslint app.js --fix"
    }
  }
}
```

- commit 하기전에 해당 eslint를 app.js 파일 한해서 실행하고 자동으로 수정할 수 있는 부분은 수정함
- 포맷팅은 --fix 옵션에 의해서 수정되지만, 코드 품질 파트는 에러를 반환하고 커밋이 실패됨

## Lint Stage 패키지 설치

- 특정 파일 및 경로를 대상으로 하는 모든 소스 파일을 lint 검사 진행한다면 굉장히 느려질 수 있음
- 그래서 commit 대상이 되는 파일(js)만 특정화해야하는데 도와주는 도구

### 설치

```shell
$ npm install -D lint-staged
```

### 적용

`package.json`

```json lines
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": "eslint --fix"
  }
}
```

- lint-staged 패키지를다운 받은 후 확장자 패턴을 지정
- husky의 hooks setting도 pre-commit때 lint-staged를 실행할 수 있게 설정
- git staged(변경한 파일) 상태의 js 확장자를 가지는 소스파일에 대해 ESLint가 실행됨
- 이 역시, lint를 통과하지 못한 소스코드는 에러를 내뱉고 커밋이 실패함

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
