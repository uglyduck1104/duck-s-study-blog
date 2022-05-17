---
title: GraphQL Overview
date: '2022-05-17'
tags: ['GraphQL']
draft: false
summary: Apollo GraphQL의 등장과 사용 이점
layout: PostSimple
authors: ['default']
---

# GraphQL

## 등장

<img alt="graphql_1" src="/static/images/graphql_1.png" width="500"/>

- Apollo GraphQL은 현재까지 약 279,425개 웹 사이트의 데이터에 액세스함
  - 국내에서는 342개의 웹사이트가 GraphQL을 사용중임
- 2012년부터 Facebook 모바일 어플리케이션이 GraphQL을 사용하면서 수면 위로 올라옴
- 2015년부터 graphql specification `오픈 소스`화
- GraphQL을 별도로 명세해놓은 하나의 spec임
  - 해당 스펙을 기준으로 [javascript로 구현](https://github.com/graphql/graphql-js)
    하거나 [명세 문서](https://github.com/graphql/graphql-spec)가 존재
  - 해당 스펙을 따르는 server를 제공

## Overfetching

- Rest API의 문제점인 쓰지 않는 데이터를 포함해서 `너무 많은 data를 넘겨받는 문제` 발생
  - 응답하는 backend나 database가 더 많은 일을 처리해야함
- GraphQL은 URL로 불필요한 모든 data를 받지 않고, `필요한 data만을 요청`해서 응답받을 수 있음

`GraphQL`

```
{
  hero{
    name
    height
    mass
  }
}
```

`Json`

```json lines
{
  "hero": {
    "name": "Luke Skywalker",
    "height": 1.72,
    "mass": 77
  }
}
```

## Underfatching

- Overfatching과는 반대로 요청하고자 하는 데이터에 필요한 정보를 가지고 있지 않을 때, 한 화면을 구성하기 위하여 `두 개 이상의 데이터를 요청`하는 비효율적인 문제가 발생
  - 요청하고자 하는 데이터를 로딩하는 과정에서의 `지연`과 `실패`가 발생할 우려가 있음
- GraphQL은 필요한 데이터를 넘겨받아 `하나의 요청으로 처리`할 수 있는 방법을 제공함

## GraphQL 예제로 알아보기

- GraphQL 문법을 웹 환경에서 예제를 통해 Sample API를 테스트해 볼 수 있음
  - [GraphQL Query 테스트해보기](http://graphql.org/swapi-graphql/?query=%23%20Welcome%20to%20GraphiQL%0A%23%0A%23%20GraphiQL%20is%20an%20in-browser%20tool%20for%20writing%2C%20validating%2C%20and%0A%23%20testing%20GraphQL%20queries.%0A%23%0A%23%20Type%20queries%20into%20this%20side%20of%20the%20screen%2C%20and%20you%20will%20see%20intelligent%0A%23%20typeaheads%20aware%20of%20the%20current%20GraphQL%20type%20schema%20and%20live%20syntax%20and%0A%23%20validation%20errors%20highlighted%20within%20the%20text.%0A%23%0A%23%20GraphQL%20queries%20typically%20start%20with%20a%20%22%7B%22%20character.%20Lines%20that%20start%0A%23%20with%20a%20%23%20are%20ignored.%0A%23%0A%23%20An%20example%20GraphQL%20query%20might%20look%20like%3A%0A%23%0A%23%20%20%20%20%20%7B%0A%23%20%20%20%20%20%20%20field(arg%3A%20%22value%22)%20%7B%0A%23%20%20%20%20%20%20%20%20%20subField%0A%23%20%20%20%20%20%20%20%7D%0A%23%20%20%20%20%20%7D%0A%23%0A%23%20Keyboard%20shortcuts%3A%0A%23%0A%23%20%20Prettify%20Query%3A%20%20Shift-Ctrl-P%20(or%20press%20the%20prettify%20button%20above)%0A%23%0A%23%20%20%20%20%20Merge%20Query%3A%20%20Shift-Ctrl-M%20(or%20press%20the%20merge%20button%20above)%0A%23%0A%23%20%20%20%20%20%20%20Run%20Query%3A%20%20Ctrl-Enter%20(or%20press%20the%20play%20button%20above)%0A%23%0A%23%20%20%20Auto%20Complete%3A%20%20Ctrl-Space%20(or%20just%20start%20typing)%0A%23%0A%0A)

### Overfatching 예제

`GraphQl`

```text
allFilms{
  totalCount
  films{
    title
  }
}
```

`Json`

```json
{
  "data": {
    "allFilms": {
      "totalCount": 6,
      "films": [
        {
          "title": "The Phantom Menace"
        },
        {
          "title": "Attack of the Clones"
        },
        {
          "title": "Revenge of the Sith"
        }
      ]
    }
  }
}
```

- 위에서 서술한 내용처럼 내가 원하는 데이터의 요청만 받아을 수 있도록 쿼리를 작성할 수 있음

### Underfatching 예제

`GraphQl`

```text
allFilms{
    totalCount
    films{
      title
    }
  }
allPeople{
  people{
    name
    hairColor
    eyeColor
    birthYear
  }
}
```

`Json`

```json
{
  "data": {
    "allFilms": {
      "totalCount": 6,
      "films": [
        {
          "title": "A New Hope"
        },
        {
          "title": "The Empire Strikes Back"
        },
        {
          "title": "Attack of the Clones"
        },
        {
          "title": "Revenge of the Sith"
        }
      ]
    },
    "allPeople": {
      "people": [
        {
          "name": "Luke Skywalker",
          "hairColor": "blond",
          "eyeColor": "blue",
          "birthYear": "19BBY"
        },
        {
          "name": "C-3PO",
          "hairColor": "n/a",
          "eyeColor": "yellow",
          "birthYear": "112BBY"
        },
        {
          "name": "R2-D2",
          "hairColor": "n/a",
          "eyeColor": "red",
          "birthYear": "33BBY"
        }
      ]
    }
  }
}
```

- REST API 였다면, people와 films 데이터를 두 번 요청해야 함
- GraphQL은 API 요청을 두 번 보낼 필요 없이, 파편화되어 있는 데이터를 `하나의 request(요청)로 불러올 수 있음`

> **Referenced**

- [Nomad coding](https://nomadcoders.co/graphql-for-beginners)
- [GraphQL Official Doc](https://graphql-kr.github.io/)
