---
title: 페이지 매김, 프리페칭과 변이
date: '2023-01-13'
tags: ['react']
draft: false
summary: 블로그 댓글을 위한 쿼리 생성하기, 쿼리 키, Pagination, Pre-fatching, Mutation
layout: PostSimple
authors: ['default']
---

# 페이지 매김, 프리페칭과 변이

## 블로그 댓글을 위한 쿼리 생성하기

### `PostDetail.jsx`

```jsx
import { useQuery } from 'react-query'

async function fetchComments(postId) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${postId}`)
  return response.json()
}

async function deletePost(postId) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/postId/${postId}`, {
    method: 'DELETE',
  })
  return response.json()
}

async function updatePost(postId) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/postId/${postId}`, {
    method: 'PATCH',
    data: { title: 'REACT QUERY FOREVER!!!!' },
  })
  return response.json()
}

export function PostDetail({ post }) {
  const { data, isLoading, isError, error } = useQuery('comments', () => fetchComments(post.id))
  if (isLoading) return <h3>Loading!</h3>

  if (isError) {
    return (
      <>
        <h3>Error</h3>
        <p>{error.toString()}</p>
      </>
    )
  }

  return (
    <>
      <h3 style={{ color: 'blue' }}>{post.title}</h3>
      <button>Delete</button> <button>Update title</button>
      <p>{post.body}</p>
      <h4>Comments</h4>
      {data.map((comment) => (
        <li key={comment.id}>
          {comment.email}: {comment.body}
        </li>
      ))}
    </>
  )
}
```

- `Posts` 컴포넌트에서와 같이 `isError` 및 `isLoading`, `error`를 구조 분해 할당하여 결과를 설명할 수 있어야 함
- 쿼리 함수에는 `post.id`를 인수로 전달하기 위해 댓글을 가져오는 익명 함수으로 구현
- 각 게시물에 달린 댓글의 목록을 불러올 수 있지만, `post.id`에 해당하는 게시글의 댓글을 새로 불러올 수 없음
  - `React Query`는 캐시와 관련이 있으므로, 쿼리 키를 통해 이를 해결할 수 있음

## 쿼리 키

- 모든 쿼리가 comments 쿼리 키를 동일하게 사용하고 있기 때문에, 보통 어떠한 트리거가 있어야만 데이터를 다시 가져오게 됨

  1. 컴포넌트를 다시 마운트할때
  2. 윈도우를 다시 포커스할 때
  3. useQuery에서 반환되어 수동으로 리페칭을 실행할때
  4. 지정된 간격으로 리페칭을 자동 실행할 때
  5. 변이(Mutation)를 생성한 뒤 쿼리를 무효화할 때

  > 클라이언트의 데이터가 서버의 데이터와 불일치할 때 리페칭이 트리거 됨

- 블로그 1의 댓글을 확인하고, 블로그 2의 댓글을 확인할 때 캐시에서는 블로그 1의 댓글을 지우거나, 블로그 2의 댓글로 덮어씌우는 것은 좋지 않음
  - 쿼리는 게시물 ID를 포함하기 때문에 쿼리별로 캐시를 남길 수 있으며 `comments` 쿼리에 대한 캐시를 공유하지 않아도 됨

```
const { data, isLoading, isError, error } = useQuery(
    ["comments", post.id],
    () => fetchComments(post.id)
  );
```

- 배열의 첫번째 요소로 문자열 `comments`를 가지고 두 번째 요소로 post.id를 가질 수 있음
  - 따라서 쿼리 키(`post.id`)가 변경되면 `React Query`가 새 쿼리를 생성해서 `staleTime`과 `cacheTime`을 가지게 되고 의존성 배열이 다르다면 완전히 다른 것으로 간주됨
  - 데이터를 가져올 때 사용하는 쿼리 함수에 있는 값이 쿼리 키에 포함되어야 함
- 가비지 콜렉터로 수집되기 전까지 캐시에 남아 있음

## 페이지 매김(Pagination)

- 컴포넌트의 상태(ex. currentPage)를 통해 현제 페이지를 파악하는 페이지 매김 스타일
- 댓글에서 작업했던 것처럼 페이지마다 다른 쿼리 키가 필요함
  - 쿼리 키를 배열로 업데이트해서 가져오는 페이지 번호를 포함해야 함

### `Posts.jsx`

```jsx
import { useState } from 'react'
import { useQuery } from 'react-query'

import { PostDetail } from './PostDetail'
const maxPostPage = 10

async function fetchPosts(pageNum) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts?_limit=10&_page=${pageNum}`
  )
  return response.json()
}

export function Posts() {
  const [currentPage, setCurrentPage] = useState(1)
  const [selectedPost, setSelectedPost] = useState(null)
  const { data, isError, error, isLoading } = useQuery(
    ['posts', currentPage],
    () => fetchPosts(currentPage),
    {
      staleTime: 2000,
    }
  )
  if (isLoading) return <h3>Loading...</h3>
  if (isError)
    return (
      <>
        <h3>Oops, something went wrong</h3>
        <p>{error.toString()}</p>
      </>
    )

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
        <button
          disabled={currentPage <= 1}
          onClick={() => {
            setCurrentPage((prev) => prev - 1)
          }}
        >
          Previous page
        </button>
        <span>Page {currentPage}</span>
        <button
          disabled={currentPage >= maxPostPage}
          onClick={() => {
            setCurrentPage((prev) => prev + 1)
          }}
        >
          Next page
        </button>
      </div>
      <hr />
      {selectedPost && <PostDetail post={selectedPost} />}
    </>
  )
}
```

- 사용자가 다음 혹은 이전 페이지로 가는 버튼을 누르면 `currentPage` 상태를 업데이트
- `const [currentPage, setCurrentPage] = useState(1);`
  - `currentPage`의 `setStateAction`의 초기화 값을 1로 설정(1페이지)
- `const { data, isError, error, isLoading } = useQuery(...);`
  - 쿼리 키에 currentPage를 포함하여 currentPage 상태가 바뀌면 React Query가 바뀐 쿼리 키를 감지해, 새 쿼리 키에 대한 데이터를 업데이트 함
- `async function fetchPosts(pageNum) {…}`
  - 어떤 페이지 번호를 입력하든 간에 가져올 수 있도록 `fetchPosts`에 `pageNum`을 전달

## 데이터 프리페칭(Pre-fatching)

- 페이지에 캐시가 존재하지 않기 때문에, `Next page` 버튼을 누를 때마다 페이지가 로딩되길 기다려야 했지만, 데이터 프리페칭 기능을 통해 데이터를 사용하고자 할 때 만료 상태에서 데이터를 다시 가져오게 할 수 있음
  - 통계적으로 다수의 사용자가 웹사이트 방문 시 통계적으로 특정 탭을 누를 확률이 높다면 해당 데이터를 미리 가져오게 구성하는 것을 권장

### `Posts.jsx`

```jsx
import { useEffect } from 'react'
import { useState } from 'react'
import { useQuery, useQueryClient } from 'react-query'

import { PostDetail } from './PostDetail'
const maxPostPage = 10

async function fetchPosts(pageNum) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts?_limit=10&_page=${pageNum}`
  )
  return response.json()
}

export function Posts() {
  const [currentPage, setCurrentPage] = useState(1)
  const [selectedPost, setSelectedPost] = useState(null)

  const queryClient = useQueryClient()
  useEffect(() => {
    if (currentPage < maxPostPage) {
      const nextPage = currentPage + 1
      queryClient.prefetchQuery(['posts', nextPage], () => fetchPosts(nextPage))
    }
  }, [currentPage, queryClient])

  const { data, isError, error, isLoading } = useQuery(
    ['posts', currentPage],
    () => fetchPosts(currentPage),
    {
      staleTime: 2000,
      keepPreviousData: true,
    }
  )
  if (isLoading) return <h3>Loading...</h3>
  if (isError)
    return (
      <>
        <h3>Oops, something went wrong</h3>
        <p>{error.toString()}</p>
      </>
    )

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
        <button
          disabled={currentPage <= 1}
          onClick={() => {
            setCurrentPage((prev) => prev - 1)
          }}
        >
          Previous page
        </button>
        <span>Page {currentPage}</span>
        <button
          disabled={currentPage >= maxPostPage}
          onClick={() => {
            setCurrentPage((prev) => prev + 1)
          }}
        >
          Next page
        </button>
      </div>
      <hr />
      {selectedPost && <PostDetail post={selectedPost} />}
    </>
  )
}
```

- `prefetch` 쿼리는 `queryClient`의 메서드로써, `useQuery` 훅을 통해 `queryClient`를 가져올 수 있음
  - `const queryClient = useQueryClient();`
- Next page 버튼을 눌렀을 때, 상태 업데이트가 비동기식으로 일어나므로 이미 업데이트가 진행됐는지 알방법이 없음

  - `useEffect`를 사용해 현제 페이지에 생기는 변경 사항을 활용할 수 있음

    ```jsx
    useEffect(() => {
      if (currentPage < maxPostPage) {
        const nextPage = currentPage + 1
        queryClient.prefetchQuery(['posts', nextPage], () => fetchPosts(nextPage))
      }
    }, [currentPage, queryClient])
    ```

    - 현재 페이지가 변경될 때마다 `queryClient.prefetchQuery`를 통해 다음 페이지의 데이터를 가져옴
    - `prefetchQuery` 메서드 역시 `useQuery`와 흡사하게 인수를 전달함
    - 프리페칭하는 데이터의 범위를 maxPostPage(최대 10페이지) 미만으로 잡고 최대 9페이지 이전에만 프리페칭이 일어나게 제한

- 이전 페이지로 돌아갔을 때 캐시에 해당 데이터가 있도록 만들고 싶다면, `useQuery` 인수에 `option`값으로 `keepPreviousData`에 `true`값으로 지정

## 변이(Mutation) 입문

- 서버에 데이터를 업데이트하도록 서버에 네트워크 호출을 실시

### Mutation의 활용

1. 변경 내용을 사용자에게 보여주거나 진행된 변경 내용을 등록하여 사용자가 볼 수 있게 할 수 있음
2. 변이 호출을 실행할 때 서버에서 받는 데이터를 취하고 업데이트된 해당 데이터로 React Query 캐시를 업데이트할 수 있음
3. 관련 쿼리를 무효화해 서버에서 리페치를 개시하여 클라이언트에 있는 데이터를 서버의 데이터와 최신 상태로 유지

### Mutation의 사용

- 쿼리 키는 필요하지 않음
- isLoading은 존재하지만 isFetcing은 존재하지 안음
- 변이에 관련된 캐시는 존재하지 않고 재시도 또한 기본값으로 존재하지 않음
  - useQuery는 이에 반해 기본값으로 3회 재시도 함

## useMutation로 포스팅 삭제하기

```jsx
import { useQuery, useMutation } from 'react-query'

async function fetchComments(postId) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${postId}`)
  return response.json()
}

async function deletePost(postId) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/postId/${postId}`, {
    method: 'DELETE',
  })
  return response.json()
}

async function updatePost(postId) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/postId/${postId}`, {
    method: 'PATCH',
    data: { title: 'REACT QUERY FOREVER!!!!' },
  })
  return response.json()
}

export function PostDetail({ post }) {
  const { data, isLoading, isError, error } = useQuery(['comments', post.id], () =>
    fetchComments(post.id)
  )

  const deleteMutation = useMutation((postId) => deletePost(postId))

  if (isLoading) return <h3>Loading!</h3>

  if (isError) {
    return (
      <>
        <h3>Error</h3>
        <p>{error.toString()}</p>
      </>
    )
  }

  return (
    <>
      <h3 style={{ color: 'blue' }}>{post.title}</h3>
      <button onClick={() => deleteMutation.mutate(post.id)}>Delete</button>
      {deleteMutation.isError && <p style={{ color: 'red' }}>Error deleting the post</p>}
      {deleteMutation.isLoading && <p style={{ color: 'purple' }}>Deleting the post</p>}
      {deleteMutation.isSuccess && <p style={{ color: 'green' }}>Post has (not) been deleted</p>}
      <button>Update title</button>
      <p>{post.body}</p>
      <h4>Comments</h4>
      {data.map((comment) => (
        <li key={comment.id}>
          {comment.email}: {comment.body}
        </li>
      ))}
    </>
  )
}
```

- `const deleteMutation = useMutation((postId) => deletePost(postId));`
  - `useQuery`와 다르게 쿼리 키를 인수로써 전달하지 않고, 인수로 전달하려는 변이 함수 그 자체도 인수로 받을 수 있음
  - `postId`에 대해 특정 `action`을 실행하게 되는데, `useMutation`에서 객체는 변이 함수를 반환하게 됨
- `<button onClick={() => deleteMutation.mutate(post.id)}>Delete</button>`
  - `onClick` Event handler를 통해 `Delete` 버튼을 클릭할 때 변이 함수를 실행하고 객체를 반환하는 `deleteMutation`과 속성 함수인 `mutate`를 실행하므로 인수로 `Props`에서 받은 `postId`를 전달함

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
