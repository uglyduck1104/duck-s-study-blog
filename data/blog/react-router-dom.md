---
title: React Router Example
date: '2022-05-08'
tags: ['react']
draft: false
summary: React Router Dom 예제
layout: PostSimple
authors: ['default']
---

# react-router-dom

- 라우트를 구성하는 컴포넌트들의 모음을 제공

## 설치

```shell
$ npm i react-router-dom
```

## react-router-dom을 이용해 `App.js` 구성

```javascript
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom'
import Home from './routes/Home'
import Detail from './routes/Detail'

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/hello" element={<h1>Hello</h1>} />
        <Route path="/movie/:id" element={<Detail />} />
      </Routes>
    </Router>
  )
}

export default App
```

- `<Router>`
  - 라우트를 구성하는 Router 컴포넌트안에 유저에게 보여주고 싶은 하위 Route 컴포넌트를 구성
- `<Route>`
  - 실질적으로 보여지는 컴포넌트 혹은 Dom을 구성하고, path 키에 유저가 접근하는 url pattern을 지정
  - 즉, `url path`를 정의한 path에 맞게 접근하면 하위의 컴포넌트를 렌더링함
  - `element`에 반환하고자 하는 컴포넌트를 정의
- React Router는 동적 URL을 지원함
  - `/movie/:id`처럼 parameter를 받아 라우트를 구성할 수 있음

## Route간의 이동 방법

- 기존의 a 태그를 이용한 페이지 이동은 React의 관점(`재실행 지양`)과 맞지 않는 방식임
  - 페이지 전체가 다시 새로고침이 됨
- react-router-dom package가 제공하는 `Link 컴포넌트` 사용
  - 페이지의 새로고침 없이도 유저를 다른 페이지로 이동시킴

`Movie.js`

```javascript
import { Link } from 'react-router-dom'

function Movie({ id, coverImg, title, summary, genres }) {
  return (
    <div>
      <img src={coverImg} alt={title} />
      <h2>
        <Link to={`/movie/${id}`}>{title}</Link>
      </h2>
      <p>{summary}</p>
      <ul>
        {genres.map((g) => (
          <li key={g}>{g}</li>
        ))}
      </ul>
    </div>
  )
}
```

- Link 컴포넌트를 사용해 `App.js`에서 구성한 라우트를 기반하여 이동하고자하는 `path`를 지정
  - `Home.js`
    ```javascript
    <Movie
      key={movie.id}
      id={movie.id}
      coverImg={movie.medium_cover_image}
      title={movie.title}
      summary={movie.summary}
      genres={movie.genres}
    />
    ```
    - Movie 컴포넌트에 `id`를 넘겨줄 수 있음
  - Movie 컴포넌트도 결국 함수이므로, 호출하는 라우트에서 id를 props로 넘겨받아 Link 컴포넌트로 라우트에 구성되어 있는 `url path`와 `parameter`로 넘겨줌

## Parameter

`Detail.js`

```javascript
import { useEffect } from 'react'
import { useParams } from 'react-router-dom'

function Detail() {
  const { id } = useParams()
  const getMovie = async () => {
    const json = await (
      await fetch(`https://yts.mx/api/v2/movie_details.json?movie_id=${id}`)
    ).json()
  }
  useEffect(() => {
    getMovie()
  }, [])
  return <h1>Detail</h1>
}
export default Detail
```

- react-router-dom에서 지원하는 useParams hook을 이용해서 렌더링된 Detail의 컴포넌트의 변수 값을 넘겨받을 수 있음
- 라우트 구성에서 봤던 `/movie/:id`의 `id` 변수 값을 이용하여 API를 요청할 수 있음

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
