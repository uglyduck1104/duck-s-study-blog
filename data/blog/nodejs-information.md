---
title: NodeJS Overview
date: '2022-10-20'
tags: ['NodeJS']
draft: false
summary: NodeJS 소개 및 역할
layout: PostSimple
authors: ['default']
---

# NodeJS Overview
<img alt="nodejs_information_1" src="/static/images/nodejs_information_1.png" width="700"/>

- JavaScript 런타임으로써 브라우저뿐만아니라 다른 곳에서도 실행할 수 있도록 서버에서 실행할 웹 애플리케이션을 구현하는데 매우 유용하게 쓰이고 있음
- V8 구글 엔진으로 Node.js Javascript 코드를 머신 코드로 컴파일해줌
- V8 자체는 C++로 쓰여있으며, 로컬 파일 시스템, 파일 열기, 읽기, 삭제 등의 기능을 추가해 줌
  - 브라우저에서는 이와 같은 기능이 보안상의 이유로 액세스 할 수 없으므로 지원하지 않음

## Node.js 역할과 사용법 이해

<img alt="nodejs_information_2" src="/static/images/nodejs_information_2.png" width="700"/>

### Client

- my-page.com 페이지에 방문해서 해당되는 요청을 보냄
  - 브라우저에 URL을 입력하고 요청을 전달

### Server

- 인터넷에서 실행 중인 컴퓨터로 해당 도메인과 관련된 IP를 가지고 있으면 자동으로 할당됨
- 서버에서 들어오는 요청을 처리해서 응답을 회신하는 코드를 실행
- 성능 또는 보안상의 이유로 브라우저에서 할 수 없는 작업을 수행
- 데이터를 불러오거나 저장하기 위해 데이터베이스에 접속하거나 사용자 인증을 수행
- 사용자에게 데이터를 회신하는 코드를 서버에 작성해서 클라이언트가 사용

> **Referenced**

- [NodeJS - The Complete Guide (MVC, REST APIs, GraphQL, Deno)](https://www.udemy.com/course/nodejs-mvc-rest-apis-graphql-deno/)
