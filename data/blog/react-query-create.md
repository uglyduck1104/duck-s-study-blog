---
title: 쿼리 생성 및 로딩/에러 상태
date: '2023-01-12'
tags: ['react']
draft: false
summary: useQuery로 쿼리 생성하기, 로딩 상태와 에러 상태 처리하기, React Query 개발자 도구, staleTime vs cacheTime
layout: PostSimple
authors: ['default']
---

# 쿼리 생성 및 로딩/에러 상태

## useQuery로 쿼리 생성하기

### React Query 설치

```bash
npm install react-query@^3
```

### App.jsx 구성

```jsx
import { Posts } from './Posts'
import { QueryClient, QueryClientProvider } from 'react-query'
import './App.css'
const queryClient = new QueryClient()

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <h1>Blog Posts</h1>
        <Posts />
      </div>
    </QueryClientProvider>
  )
}

export default App
```

- 모든 하위 컴포넌트가 클라이언트를 사용할 수 있게 하는 `QueryProvider`를 추가해 `React Query`에서 필요한 것을 가져옴
  - 클라이언트가 가지고 있는 캐시와 모든 기본 옵션을 하위 컴포넌트에서 사용할 수 있음
  - `React Query Hook`을 사용할 수 있음

### Post.jsx

```jsx
import { useState } from 'react'
import { useQuery } from 'react-query'

import { PostDetail } from './PostDetail'
const maxPostPage = 10

async function fetchPosts() {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=10&_page=0')
  return response.json()
}

export function Posts() {
  const [currentPage, setCurrentPage] = useState(0)
  const [selectedPost, setSelectedPost] = useState(null)
  const { data } = useQuery('posts', fetchPosts)
  if (!data) return <div />

  return (
    <>
      <ul>
        {data.map((post) => (
          <li key={post.id} className="post-title" onClick={() => setSelectedPost(post)}>
            {post.title}
          </li>
        ))}
      </ul>
      <div className="pages">
        <button disabled onClick={() => {}}>
          Previous page
        </button>
        <span>Page {currentPage + 1}</span>
        <button disabled onClick={() => {}}>
          Next page
        </button>
      </div>
      <hr />
      {selectedPost && <PostDetail post={selectedPost} />}
    </>
  )
}
```

- `useQuery` 훅을 사용해 서버에서 데이터를 가져올 수 있음
  - `jsonplaceholder`를 사용해 `Mock JSON`의 목록을 가져옴
  - 데이터 구조 분해 할당을 통해 `data`의 값을 변수에 할당
- `useQuery`의 인수의 첫번째는 쿼리 키값, 두번째 인수는 데이터를 가져오는 비동기 함수를 전달
  - 비동기 함수이므로, `fetchPosts`가 해결될 때까지 `useQuery`에서 정의되지 않음
    - `data`가 정의되지 않은 경우 빈 `div` 태그를 반환하는 분기문 추가

## 로딩 상태와 에러 상태 처리하기

- React Query에서는 데이터를 Fetching할때 처리하는 여러 속성이 있는데, 그중 로딩 상태와 에러 상태를 정의할 수 있음
  - 이외의 속성은 [참고 링크](https://react-query-v3.tanstack.com/reference/useQuery)에서 확인 가능함

```jsx
const { data, isError, error, isLoading } = useQuery('posts', fetchPosts)
if (isLoading) return <h3>Loading...</h3>
if (isError)
  return (
    <>
      <h3>Oops, something went wrong</h3>
      <p>{error.toString()}</p>
    </>
  )
```

### React Query 속성

- 데이터가 로딩 중인지 여부와 데이터를 가져올 때 오류가 있는지 알려주는 불리언
- `isFetching`
  - 비동기 쿼리가 아직 완료되지 않는 경우
- `isLoading`
  - `isFetcing`의 하위 집합으로써 데이터를 가져오는 상태를 의미함(아직 해결되지 않은 상태)
  - 데이터를 가져오는 중이고 표시할 캐시 데이터도 없음
- `isError`
  - 쿼리 함수인 `fetchPosts`에서 오류가 발생했을 경우
  - `error` 속성과 함께 사용하여 `error message`을 화면에 노출할 수 있음

## React Query 개발자 도구

- 앱에 추가할 수 있는 컴포넌트로 개발 중인 모든 쿼리의 상태를 표시해주고 예상대로 작동하지 않는 경우 문제를 해결하는데 도움을 받을 수 있음
- 쿼리 키로 쿼리를 표시해주며 활성(`active`), 비활성(`inactive`), 만료(`stale`) 등 모든 쿼리의 상태를 알려줌
  - 마지막으로 업데이트된 타임스탬프도 확인할 수 있음
- 쿼리를 볼 수 있는 쿼리 탐색기, 데이터를 확인할 수 있는 데이터 탐색기 제공
- 기본적으로 개발자 도구는 프로덕션 번들에 포함되어 있지 않아서 `NODE_ENV` 변수에 따라 프로덕션 환경에 있는지 여부가 결정됨
  - 예를 들어 `CRA` 앱은 `npm run build`를 실행할 때에만 `NODE_ENV` 변수를 `production`으로 설정함

### 개발자 도구 구성 - App.jsx

```jsx
import { Posts } from './Posts'
import { QueryClient, QueryClientProvider } from 'react-query'
import { ReactQueryDevtools } from 'react-query/devtools'

import './App.css'
const queryClient = new QueryClient()

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <h1>Blog Posts</h1>
        <Posts />
      </div>
      <ReactQueryDevtools />
    </QueryClientProvider>
  )
}

export default App
```

- `react-query` 의존성 하위 패키지에 `devtools`에 있는 `ReactQueryDevtools`를 가져온 후, `QueryClientProvider` 하위에 위치시켜 줌

## staleTime vs cacheTime

### staleTime

- 데이터 리페칭은 만료된 데이터에서만 실행되는데 예들 들어, 컴포넌트가 다시 마운트되거나 윈도우가 다시 포커스되었을 경우 만료된 데이터에 한해서 리페칭이 실행됨
- 즉, 데이터가 만료됐다고 판단하기 전까지 허용하는 시간을 의미함

```jsx
const { data, isError, error, isLoading } = useQuery('posts', fetchPosts, {
  staleTime: 2000,
})
```

- 세번째 인자의 속성값(`options`)으로 `staleTime`키를 가진 값을 `millisecond` 단위로 할당
- staleTime의 기본값은 0으로 설정되었음
  - 데이터는 항상 만료 상태이므로 서버에서 다시 가져와야 한다는 것을 가정함(`HTTP / connectless / stateless`)

### cacheTime

- `staleTime`과 다르게 캐시는 나중에 다시 필요할 수도 있는 데이터 용도로 사용됨
- 특정 쿼리에 대한 활성된 `useQuery`가 없는 경우 해당 데이터는 콜드 스토리지로 이동되고 구성된 `cacheTime`이 지나면 캐시의 데이터가 만료되며 유효 시간의 기본값은 5분으로 설정되어 있음
- `cacheTime`이 관찰하는 시간의 양은 특정 쿼리에 대한 `useQuery`가 활성화된 후 경과한 시간임
- 캐시가 만료되면 가비지 컬렉션이 실행되고 클라이언트는 데이터를 사용할 수 없음
  - 만료된 데이터가 위험할 경우 주로 `cacheTime`을 0으로 설정

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
