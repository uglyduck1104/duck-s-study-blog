---
title: GraphQL Resolvers
date: '2022-05-20'
tags: ['GraphQL']
draft: false
summary: 명세한 Query Types에 대한 로직을 작성해야 하는데, `resolvers`라고 불리우는 객체를 Apollo server에 전달 
layout: PostSimple
authors: ['default']
---

# GraphQL Resolvers

## Query Resolvers

- 명세한 Query Types에 대한 로직을 작성해야 하는데, `resolvers`라고 불리우는 객체를 Apollo server에 전달
  - 반드시 정의한 SDL(Schema Definition Language)와 같은 형태를 가져야 함

`server.js`

```javascript
import {ApolloServer, gql} from "apollo-server";

const tweets = [
  {
    id: "1",
    text: "first one!",
  },
  {
    id: "2",
    text: "first two",
  },
]
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
    author: User
  }
  type Query {
    allTweets: [Tweet!]!
    tweet(id: ID!): Tweet
  }
`;

const resolvers = {
  Query: {
    allTweets() {
      return tweets;
    },
    tweet(root, {id}) {
      return tweets.find(tweet => tweet.id === id);
    }
  }
};
const server = new ApolloServer({typeDefs, resolvers})
```

- 명세한 types와 상응하는 resolver 함수를 작성하고 반환함
- 유저가 query의 tweet을 요청한다면, Apollo는 resolvers의 query로 해당하는 함수를 실행시킴
- `tweet(root, {id})`
  - Apollo 서버가 resolver 함수를 호출할 때 `인자 두개`를 받음(GraphQL의 명세)
    - 첫번째 인자(`root` argument)
      - 받아오는 argument가 없다면, root로부터 먼저 찾음
    - 두번째 인자(`args`)
      - 유저가 argument(ex. id 값)을 보낼 때, 항상 resolver 함수의 `두번째 인자값`으로 받음
- 현재는 데이터가 없으므로 가상의 객체(데이터)를 선언해서 유저가 요청하는 아이디값과 일치하는 아이디가 있으면 해당 데이터를 반환

## Mutation Resolvers

`server.js`

```javascript
import {ApolloServer, gql} from "apollo-server";

let tweets = [
  {
    id: "1",
    text: "first one!"
  },
  {
    id: "2",
    text: "first two"
  }
];
const typeDefs = gql`
  // ...
  type Mutation {
    postTweet(text: String!, userId: ID!): Tweet!
    deleteTweet(id: ID!): Boolean!
  }
`;
const resolvers = {
  // Query: { ... },
  Mutation: {
    postTweet(_, {text, userId}) {
      const newTweet = {
        id: tweets.length + 1,
        text
      };
      tweets.push(newTweet);
      return newTweet;
    },
    deleteTweet(_, {id}) {
      const tweet = tweets.find(tweet => tweet.id === id);
      if (!tweet) return false;
      tweets = tweet.filter(tweet => tweet.id !== id);
      return true;
    }
  }
}
const server = new ApolloServer({typeDefs, resolvers})
```

- Mutation Type에 상응하는 `Mutation` resolver 객체를 생성하여 로직을 작성
- Mutation resolver를 실행하기위해서, 유저로부터 넘겨 받은 `arguments들의 순서`에 집중할 필요가 있음
- `postTweet(_, {text, userId})`
  - 현재 id에 1을 더하고, 넘겨받은 argument인 `text`를 newTweet 객체에 담에 `tweets` 배열을 Mutation 함
- `deleteTweet(_, {id})`
  - tweets 객체에 넘겨받은 id값을 찾아서 있으면 filter method로 새로운 배열을 `tweets`에 대입하고 `true`를 반환

> database를 fake 객체를 이용해 javascript로 구현한 것이므로, 본인 언어에 따라 달라질 수 있음(Concept만 확인하기!!)

## Mutation 확인(Graphql Studio)

- 서버가 실행된 상태에서 4000 port에 접근하면, studio에 접근할 수 있음
- 현재 만들고자하는 api를 확인할 수 있으므로, 강력한 자동완성와 동적 argument 자동완성을 지원함

### postTweet

![graphql_resolvers_1](/static/images/graphql_resolvers_1.png)

![graphql_resolvers_2](/static/images/graphql_resolvers_2.png)

![graphql_resolvers_3](/static/images/graphql_resolvers_3.png)
- postTweet에 맞춰 포맷을 지정한 후, graphql에서 제공하는 자동 완성을 이용하여 하단 창에 variables(`userId`, `text`)에 값을 대입
- 우측 response 창에 mutation 응답이 성공되었는지 확인

### postTweet 반영 데이터 확인

![graphql_resolvers_4](/static/images/graphql_resolvers_4.png)

![graphql_resolvers_5](/static/images/graphql_resolvers_5.png)

- postTweet mutation이 반영된 목록을 확인

### deleteTweet

![graphql_resolvers_6](/static/images/graphql_resolvers_6.png)

![graphql_resolvers_7](/static/images/graphql_resolvers_7.png)

![graphql_resolvers_8](/static/images/graphql_resolvers_8.png)

- 삭제하고자 하는 tweet의 id값을 전달하여, 해당 아이디와 일치하는 값이 있으면 true를 반환하는지 확인

### deleteTweet 반영 데이터 확인

![graphql_resolvers_9](/static/images/graphql_resolvers_9.png)

- deleteTweet mutation이 반영된 목록을 확인

## Type Resolvers

- `Query`, `Mutation` resolver처럼 Type에 상응하는 어떤 type에도 resolver 함수를 만들 수 있음

`server.js`

```javascript
import {ApolloServer, gql} from "apollo-server";

let users = [
  {
    id: "1",
    firstName: "anony",
    lastName: "mous"
  },
  {
    id: "2",
    firstName: "Elon",
    lastName: "mask"
  }
];
const typeDefs = gql`
  type User {
    id: ID!
    firstName: String!
    lastName: String!
    fullName: String!
  }
  type Tweet {...}
  type Query {...}
  type Mutation {...}
`;
const resolvers = {
  Query: {
    //...
    allUsers() {
      return users;
    }
  },
  // Mutation: {...},
  User: {
    fullName({firstName, lastName}) {
      return `${firstName} ${lastName}`;
    }
  }
};
```

- 가상의 User 객체(database)를 정의
- data안에 존재하지 않는 `fullName field`를 동적으로 구성하기 위해 Query 내부에 있는 allUsers라는 type와 상응하여 함수를 정의
  - graphql이 작동하는 방식에 따라 fullName의 resolver를 호출할 수 있음
  - 순서는 graphql이 `Query의 allUsers resolver`에 먼저 도달한 후, return하는 데이터에 `fullName field`가 없음을 감지함
  - 이때, `type User`의 field 이름이 `fullName`인 것을 찾아 resolver를 실행함
- User resolver에 첫번째 인자(`root`)로 `firstName`과 `lastName`을 가져와서 반환함
- 데이터가 있는 만큼 호출

## Relationships(Tweet - User)

- 데이터베이스와의 연관 관계(join)과 유사한 방법으로 연결할 수 있음

`server.js`

```javascript
import {ApolloServer, gql} from "apollo-server";

let users = [
  {
    id: "1",
    firstName: "anony",
    lastName: "mous"
  },
  {
    id: "2",
    firstName: "Elon",
    lastName: "mask"
  }
];
let tweets = [
  {
    id: "1",
    text: "first one!",
    userId: "2"
  },
  {
    id: "2",
    text: "second two",
    userId: "1"
  }
];
const resolvers = {
  // Query: {...}
  Mutation: {
    postTweet(_, {text, userId}) {
      const newTweet = {
        id: tweets.length + 1,
        text,
        userId,
      };
      tweets.push(newTweet);
      return newTweet;
    },
    // deleteTweet(_, { id }) {...}
  },
  User: {
    // fullName({firstName, lastName}){...}
  },
  Tweet: {
    author({userId}) {
      return users.find(user => user.id === userId)
    }
  }
};

const server = new ApolloServer({typeDefs, resolvers});
```

- tweets 객체에 `userId`를 전달
- `Tweet resolver`에 Tweet field(`author`) 인자로 userId 값을 users 객체에서 데이터 일치 여부를 검사하고 반환함
- `postTweet resolver`에서는 새로운 `tweet`을 생성할 때, 작성한 유저의 `userId`를 넘겨줘야 함

> **Referenced**

- [Nomad coding](https://nomadcoders.co/graphql-for-beginners)
