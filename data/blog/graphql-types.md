---
title: GraphQL Types
date: '2022-05-19'
tags: ['GraphQL']
draft: false
summary: GraphQL의 프로젝트 설정과 types, nullable field의 개념
layout: PostSimple
authors: ['default']
---

# GraphQL Types

- graphql은 명세이므로, 구현하기 위해서는 실제 코드를 작성해야함
- Apollo server를 이용하여 그 자체로 실행할 수 있는 환경을 구성할 수 있음
  - 기존에 실행되던 REST API를 프레임워크에 상관없이 graphql로 `미들웨어 추가`를 통해 최상단에 추가할 수 있음

## Setup

```shell
### Npm initialize
$ npm init -y
```

- 빈 프로젝트에 npm 패키지 매니저를 초기화

### 의존성 추가

```shell
$ npm install apollo-server graphql
$ npm install -D nodemon 
```

- apollo-server와 graphql을 설치하고, 더 나은 개발 경험을 위해 `nodemon`도 설치함

### 파일 구성

`package.json`

```json lines
{
  //...
  "scripts": {
    "dev": "nodemon server.js"
  },
  //...
  "type": "module"
}

```

- build script에 nodemon이 바라보는 파일을 지정
- 마지막번째 줄에 `type: module`을 추가하여 import 구문을 사용하기 위한 설정
  `server.js`

```javascript
import {ApolloServer, gql} from "apollo-server";

const server = new ApolloServer({})

server.listen().then(({url}) => {
  console.log(`Running on ${url}`);
});
```

- Apollo Server로 부터 listen 메서드를 통해 Promise 형태로 ServerInfo 객체의 url 값을 불러옴

## 실행

```shell
$ npm run dev
```

- `package.json`에서 작성했던 script를 실행하여 nodemon이 해당 파일의 변경사항이 생기면 서버를 재시작 시켜줌

### 오류 발생

`Error: Apollo Server requires either an existing schema, modules or typeDefs`

- Apollo Server는 존재하는 schema나 modules 또는 typeDefs를 가져야한다는 오류를 던짐
- graphql이 data의 shape를 미리 알고 있어야 함
- http method와 url의 조합인 rest api 규칙과는 달리 graphql은 graphql server한테 data의 shape를 설명해줘야 함

### Query Type 명세

`server.js`

```javascript
import { ApolloServer, gql } from "apollo-server";
const typeDefs = gql`
  type Query{
    text: String
    hello: String
  }
`;
const server = new ApolloServer({typeDefs})
```

- 위에 설명한 것처럼 apollo server에게 data의 shape를 알려주기 위해서는 SDL(Schema Definition Language)를 정의해야하는 데 꼭 ``(backtick)으로 감싸줘야 함
- 가장 중요한 `Query`라는 type을 작성해야 함(필수)
  - `Query`라는 이름의 root type을 지정하지 않으면 다음과 같은 오류를 내뱉음
    - `Error: Query root type must be provided`
    - 별칭을 사용해서 따로 지정이 가능함
  - Query Type에 명세한 type들은 모든 사용자가 요청 할 수 있음
- REST API의 규칙으로 치환하면 `GET /text`, `GET /hello`와 유사함

## 실행 확인

![graphql_2](/static/images/graphql_2.png)

- localhost:4000에서 실행되는 apollo server의 화면을 보면 기본 실행 환경 구성은 완료

## Scalar and Root Types

`server.js`

```javascript
const typeDefs = gql`
  type User {
    id: ID
    username: String
  }
  type Tweet {
    id: ID
    text: String
    author: User
  }
  type Query{
    allTweets: [Tweet]
    tweet(id: ID): Tweet
  }
`;
```

- 어떤 field가 반환 될 것인지에 대한 설명을 설계해야 함
- graphql에 내장되어 있는 type
  - ID
  - String
  - Boolean
  - Int
- `Query`
  - `allTweets`
    - Tweet 타입의 `둘 이상`의 데이터(`[]`)를 list type으로 반환 하도록 명세
  - `tweet`
    - 하나의 Tweet만 받아오고 싶을 경우, Tweet의 id를 인자로 받을 수 있음
      - value가 무엇인지 argument가 무엇인지 함께 지정해야 함
    - REST API `GET /api/v1/tweets/:id`와 유사함
- `Tweet`
  - 트윗의 아이디, 내용, 작성자의 필드와 타입을 명세
    - 작성자는 `한명`의 User의 정보를 가져옴
- `User`
  - 아이디와 유저의 이름을 담는 필드와 타입을 명세

## Mutation Type

`server.js`

```javascript
const typeDefs = gql`
  ...
  type Mutation {
    postTweet(text: String, userId: ID): Tweet
    deleteTweet(id: ID): Boolean
  }
`
```

- query type과 다르게 mutation type은 backend를 mutate하는 것(database)들을 명세
  - user가 data를 보내고 그게 backend에서 mutate한다면 `mutation`에서 명세해야 함
- rest api에서의 post, delete와 같은 http method와 유사하게 동작
- `postTweet`
  - postTweet(Tweet 생성)을 호출할때에는 text, userId를 보내야 함
- `deleteTweet`
  - deleteTweet(특정 Tweet 삭제)을 호출할때에는 삭제하려는 tweet의 id를 보내고 해당 tweet을 찾으면 Boolean 타입(true)을 반환

## Non Nullable Fields

`server.js`

```javascript
const typeDefs = gql`
  type User {
    id: ID!
    username: String!
    firstName: String!
    lastName: String
  }
  type Tweet {
    id: ID!
    text: String!
    author: User!
  }
  type Query{
    allTweets: [Tweet!]!
    tweet(id: ID!): Tweet
  }
  type Mutation {
    postTweet(text: String!, userId: ID!): Tweet!
    deleteTweet(id: ID!): Boolean!
  }
`;
```

- `!`를 이용해 특정 타입에 대해 필수 값을 지정할 수 있음
- graphql은 느낌표가 선언된 필드에 대해서 null 값을 반환해도 되는지 여부를 검사함
  - null이 될 수 있는 field인지 확인
- `tweet(id: ID!): Tweet`
  - tweet Query를 쓰고 싶으면 반드시 ID를 전달해야 함
  - 하지만 반환 값은 조회하는 id값을 가진 데이터가 없을 경우도 존재하므로 nullable해야 함
- non-nullable field임에도 불구하고 반환하는 값이 없거나 인자가 없을 경우 오류를 출력함

> graphql에게 어떤 건 항상 존재하고 어떤 건 null이 될 수 있는지 알려줄 수 있음

> **Referenced**

- [Nomad coding](https://nomadcoders.co/graphql-for-beginners)
