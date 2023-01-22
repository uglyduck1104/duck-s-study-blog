---
title: 동적(JIT) 데이터 로드를 위한 무한 쿼리
date: '2023-01-22'
tags: ['react']
draft: false
summary: useInfiniteQuery 호출 작성하기, InfiniteScroll 컴포넌트, useInfiniteQuery 페칭과 에러 상태
layout: PostSimple
authors: ['default']
---

# 동적(JIT) 데이터 로드를 위한 무한 쿼리

## useInfiniteQuery 호출 작성하기

### InfinitePeople.jsx

```tsx
import InfiniteScroll from 'react-infinite-scroller'
import { useInfiniteQuery } from 'react-query'

import { Person } from './Person'

const initialUrl = 'https://swapi.dev/api/people/'
const fetchUrl = async (url) => {
  const response = await fetch(url)
  return response.json()
}

export function InfinitePeople() {
  const { data, fetchNextPage, hasNextPage } = useInfiniteQuery(
    'sw-people',
    ({ pageParam = initialUrl }) => fetchUrl(pageParam),
    {
      getNextPageParam: (lastPage) => lastPage.next || undefined,
    }
  )
  return <InfiniteScroll />
}
```

- `react query`에서 `useInfinitScroll` 훅을 가져와 함수형 컴포넌트에서 호출
- 구조분해를 통해 필요한 `data`, `fetchNextPage`, `hastNextPage`를 할당
  - `FetchNextPage`
    - 더 많은 데이터가 필요할 때 어느 함수를 실행항지 InfiniteScroll에 지시하는 역할
  - `hasNextPage`
    - 수집할 데이터가 더 있는지를 결정한 Boolean
- `useInfiniteQuery(queryKey, queryFn, options)`
  - 쿼리 키(sw-people)
  - 쿼리 함수
    - `useQuery`와 같은 쿼리 함수로써 객체 매개변수를 받아 프로퍼티 중 하나로 `pageParam`을 가지고 있음
    - `pageParam`은 `fetchNextPage`이 어떻게 보일지 결정한 후, 다음 페이지가 있는지 결정함
    - `fetchUrl`에 `pageParam`을 인자로 전달한 후 `json`을 반환받음
      - 처음 실행할 때에는 `initialUrl`값으로 설정
  - 옵션
    - `lastPage`와 `allPage`를 인자로 가지는 `getNextPageParam`이라는 옵션을 줘서 `next` 프로퍼티가 다음 페이지는 무엇인지, 다음 페이지로 가는데 필요한 Url이 무엇인지 알리는 역할을 함

## InfiniteScroll 컴포넌트

- `React Infinite Scroller` 라이브러리를 이용해 `useInfiniteQuery` 구현
  - `React Query`와 호환성이 잘 맞는 라이브러리
- `InfiniteScroll` 컴포넌트에서는 두 개의 프로퍼티가 주로 사용
  - `loadMore`
    - 데이터가 더 필요할 때 불러와 `useInfiniteQuery`에서 나온 `fetchNextPage` 함수값을 사용함
  - `hasMore`
    - `useInfiniteQuery`에서 나온 객체를 해체한 값인 `hasNextPage`의 함수값을 사용

### InfinitePeople.jsx

```tsx
import InfiniteScroll from 'react-infinite-scroller'
import { useInfiniteQuery } from 'react-query'

import { Person } from './Person'

const initialUrl = 'https://swapi.dev/api/people/'
const fetchUrl = async (url) => {
  const response = await fetch(url)
  return response.json()
}

export function InfinitePeople() {
  const { data, fetchNextPage, hasNextPage } = useInfiniteQuery(
    'sw-people',
    ({ pageParam = initialUrl }) => fetchUrl(pageParam),
    {
      getNextPageParam: (lastPage) => lastPage.next || undefined,
    }
  )
  return (
    <InfiniteScroll loadMore={fetchNextPage} hasMore={hasNextPage}>
      {data.pages.map((pageData) =>
        pageData.results.map((person) => (
          <Person
            key={person.name}
            name={person.name}
            hairColor={person.hair_color}
            eyeColor={person.eye_color}
          />
        ))
      )}
    </InfiniteScroll>
  )
}
```

- `useInfiniteQuery`에서 반환된 데이터의 페이지 프로퍼티를 `pageData`라고 지정한 후 순회하여 출력된 결과를 불러옴
- `Person` 컴포넌트에 데이터를 순회하여 `pageData`에 있는 속성들을 매핑

  ```tsx
  // Person.jsx
  export function Person({ name, hairColor, eyeColor }) {
    return (
      <li>
        {name}
        <ul>
          <li>hair: {hairColor}</li>
          <li>eyes: {eyeColor}</li>
        </ul>
      </li>
    )
  }
  ```

### TypeError 발생

- `Cannot read Property pages of undefined`
  - `useInfiniteQuery`를 실행할 때, 이 쿼리 함수가 해결될 때까지 정의되지 않았다고 반환함

## useInfiniteQuery 페칭과 에러 상태

- 페이지 프로퍼티를 불러올 때 `useQuery` 컴포넌트와 마찬가지로 `isLoading`을 사용해서 캐시된 데이터가 없을 때 데이터를 가져와야 함

### InfinitePeople.jsx

```tsx
import InfiniteScroll from 'react-infinite-scroller'
import { useInfiniteQuery } from 'react-query'

import { Person } from './Person'

const initialUrl = 'https://swapi.dev/api/people/'
const fetchUrl = async (url) => {
  const response = await fetch(url)
  return response.json()
}

export function InfinitePeople() {
  const { data, fetchNextPage, hasNextPage, isLoading, isFetching, isError, error } =
    useInfiniteQuery('sw-people', ({ pageParam = initialUrl }) => fetchUrl(pageParam), {
      getNextPageParam: (lastPage) => lastPage.next || undefined,
    })

  if (isLoading) return <div className="loading">Loading...</div>
  if (isError) return <div>Error! {error.toString()}</div>

  return (
    <>
      {isFetching && <div className="loading">Loading...</div>};
      <InfiniteScroll loadMore={fetchNextPage} hasMore={hasNextPage}>
        {data.pages.map((pageData) =>
          pageData.results.map((person) => (
            <Person
              key={person.name}
              name={person.name}
              hairColor={person.hair_color}
              eyeColor={person.eye_color}
            />
          ))
        )}
      </InfiniteScroll>
    </>
  )
}
```

- if 조건절을 두어, `loading`과 `error`를 `catch`해서 로딩 메시지를 보여주거나, 에러 메시지를 출력할 수 있음
- 추가로 데이터를 `pending`중인 피드백을 화면에 사용자에게 알려주기위해 `useInfiniteQuery`에 있는 `isFetching` 분기를 이용해 `loading` 메시지를 `JSX Fragement` 안에 `isLoading`과 마찬가지로 보여줌

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
