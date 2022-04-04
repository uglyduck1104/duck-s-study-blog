---
title: project-init
date: '2022-04-04'
tags: ['npm']
draft: false
summary: NPM chapter project initialize
layout: PostSimple
authors: ['default']
---

# FE 개발 환경에서의 Node.js

## 최신 스펙 개발 가능

- 뒤쳐진 브라우져의 지원 속도 단점 보완
- 구현에 하는데 있어 징검다리 역할을 하는 바벨, 노드 기반의 웹팩, NPM 환경에서 사용할 때 FE 개발 환경을 갖출 수 있음
- 고수준 프로그래밍 언어인 Typescript, SASS를 사용하려면 `전용 트랜스 파일러`가 필요함
  - Node.js 환경이 뒷받침 되어야 함

## 빌드 자동화

- React.js의 `CRA(create-react-app)`, Vuejs 의 `vue-cli` 같이 각 프레임워크가 제공하는 도구를 사용하면 손쉽게 개발 환경을 갖출 수 있음
  - 하지만, 개발 프로젝트의 형태에 따라 커스터마이징 하려면 Node.js dml wltlrdl vlfdygka
  - 빌드 자동화의 의존성 경계 필요
    > Node.js는 FE에 `필수 기술`로써의 포지션을 갖추고 있음

## Node.js 설치

- [공식 사이트 참고](https://nodejs.org/ko/)
  - 본인의 OS 별로 설치 파일을 다운받으면 됨
  - ![project-init_1](static/images/project-init_1.png)
  - LTS
    - 짝수 형태
    - 안정적이고 신뢰도가 높음
    - `서버 운영 추천`
      > Node.js로 서버 구성 시, 본인 환경에 따라 신중하게 선택해야 함
  - Current Version
    - 주로 홀수 형태
    - 최신 기능을 가지고 있음
    - `개발 환경 구축 추천`

## Node 터미널 도구 실행(REPL)

- php나, 파이썬 같은 언어에서도 지원하는 터미널 도구
- 자바스크립트 코드를 실행할 수 있음
- Node, npm 버전 확인하기
  ```shell
  $ node --version
  v13.7.0
  $ npm --version
  6.13.6
  ```

# 프로젝트 생성

```shell
$ mkdir sample #sample 폴더 생성
$ cd sample # sample 폴더 이동
$ sample npm init # sample project init
```

```shell
package name: (sample) # 괄호안의 값은 default
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repostory:
keywords:
author:
license: (ISC)
```

## `package.json`

```json
{
  "name": "sample",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

- npm init에 값을 설정한대로 json을 생성해 줌

### Script 실행

```shell
$ npm test
# "scripts" > "test" script 실행
Error: no test specified
npm ERR! Test failed. See adove for more details.
# Error message 반환 후 에러코드 1 반환
```

## npm이 지원하는 command 목록

- access, adduser, audit, bin, bugs, c, cache, ci, cit,
  clean-install, clean-install-test, completion, config,
  create, ddp, dedupe, deprecate, dist-tag, docs, doctor,
  edit, explore, fund, get, help, help-search, hook, i, init,
  `install`, install-ci-test, install-test, it, link, list, ln,
  login, logout, ls, org, outdated, owner, pack, ping, prefix,
  profile, prune, publish, rb, rebuild, repo, restart, root,
  run, run-script, s, se, search, set, shrinkwrap, star,
  stars, `start`, stop, t, team, `test`, token, tst, un,
  uninstall, unpublish, unstar, up, update, v, version, view,
  whoami

### Custom npm command

- package.json에 "script"에 추가

```json
{
  "scripts": {
    "build": "echo \"여기에 빌드 스크립트를 추가합니다\""
  }
}
```

- npm run \<keyword> 형태로 npm에서 기본적으로 제공하는 커맨드와 달리 `run`을 명시해야 함

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
