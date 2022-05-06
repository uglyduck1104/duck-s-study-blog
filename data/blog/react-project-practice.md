---
title: React Movie App Example
date: '2022-05-05'
tags: ['react']
draft: false
summary: React Movie App 예제
layout: PostSimple
authors: ['default']
---

# Movie App

- 특정 API를 호출해서 받은 데이터를 통해, 영화들의 정보를 보여줌

## useEffect를 이용해 페이지 내 컴포넌트 구성

`App.js`

```javascript
import { useEffect, useState } from 'react'

function App() {
  const [loading, setLoading] = useState(true)
  const [movies, setMovies] = useState([])
  const getMovies = async () => {
    const json = await (
      await fetch(`https://yts.mx/api/v2/list_movies.json?minimum_rating=9&sort_by=year`)
    ).json()
    setMovies(json.data.movies)
    setLoading(false)
  }
  useEffect(() => {
    getMovies()
  }, [])

  return (
    <div>
      {loading ? (
        <h1>Loading...</h1>
      ) : (
        <div>
          {movies.map((movie) => (
            <div key={movie.id}>
              <img src={movie.medium_cover_image} />
              <h2>{movie.title}</h2>
              <p>{movie.summary}</p>
              <ul>
                {movie.genres.map((g) => (
                  <li key={g}>{g}</li>
                ))}
              </ul>
            </div>
          ))}
        </div>
      )}
    </div>
  )
}

export default App
```

- useEffect hook을 이용해서 Movie Data Url Query String에 해당하는 정보를 fetch함
  - `fetch api + then()` method를 이용해 데이터 가져오기(v1)
    ```javascript
    fetch(`https://yts.mx/api/v2/list_movies.json?minimum_rating=9&sort_by=year`)
      .then((response) => response.json())
      .then((json) => {
        setMovies(json.data.movies)
        setLoading(false)
      })
    ```
  - `async function`을 이용해 데이터 가져오기(v2)
    ```javascript
    const getMovies = async () => {
      const response = await fetch(
        `https://yts.mx/api/v2/list_movies.json?minimum_rating=9&sort_by=year`
      )
      const json = response.json()
      setMovies(json.data.movies)
      setLoading(false)
    }
    useEffect(() => {
      getMovies()
    }, [])
    ```
  - async function + `nested await`을 이용해 `lazy`하게 데이터 가져오기(v3)
    `` javascript const getMovies = async() => { const json = await ( await fetch( `https://yts.mx/api/v2/list_movies.json?minimum_rating=9&sort_by=year` ) ).json(); setMovies(json.data.movies); setLoading(false); } useEffect(() => { getMovies(); }, []);  ``
    > - API 호출에 따른 응답 데이터를 State에 갱신
    > - 갱신된 State에 따라서 컴포넌트에 해당 데이터들을 출력함

## 컴포넌트 분리

- `App.js` 컴포넌트에서 movie에 대한 정보를 출력하는 부분을 컴포넌트해서 관리할 필요가 있음

### `App.js`

```javascript
// ...
<div>
  {movies.map((movie) => (
    <Movie
      key={movie.id}
      coverImg={movie.medium_cover_image}
      title={movie.title}
      summary={movie.summary}
      genres={movie.genres}
    />
  ))}
</div>
```

- 부모 컴포넌트(App.js)에서 넘겨줄 props의 이름과 자식 컴포넌트(Movie.js)에서 넘겨 받을 props의 이름은 동일해야 함
- key는 React.js에서만, map안에서 컴포넌트들을 render할 때 사용함
  - api로부터 응답 받은 데이터의 고유 값인 `id`를 주로 사용함

### `Movie.js`

```javascript
import PropTypes from 'prop-types'

function Movie({ coverImg, title, summary, genres }) {
  return (
    <div>
      <img src={coverImg} alt={title} />
      <h2>{title}</h2>
      <p>{summary}</p>
      <ul>
        {genres.map((g) => (
          <li key={g}>{g}</li>
        ))}
      </ul>
    </div>
  )
}

Movie.propTypes = {
  coverImg: PropTypes.string.isRequired,
  title: PropTypes.string.isRequired,
  summary: PropTypes.string.isRequired,
  genres: PropTypes.arrayOf(PropTypes.string).isRequired,
}
export default Movie
```

- 자식 컴포넌트에서 부모 컴포넌트로부터 넘겨받을 props를 정의함
- props의 type을 체크하기 위한 의존성(prop-types)을 추가하고 props를 지정함

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
