---
title: NextJS Practice Project
date: '2022-04-17'
tags: ['NextJS']
draft: false
summary: 프로젝트로 예제 알아보는 NextJS 기능
layout: PostSimple
authors: ['default']
---

# NextJS Practice Project

## Layout Pattern

- 많은 사람들이 `NEXT.JS`를 이용할 때, 따르는 흔한 패턴
- `_app.js`에서 레이아웃을 위한 컴포넌트를 따로 분리 및 생성해서 관리
  - `_app.js`에는 전역으로 설정하는 값들이 추후에 쌓일 것을 생각해서, 관심사를 분리해서 관리하는 게 좋음
    `components/Layout.js`

```javascript
import NavBar from './NavBar'

export default function Layout({ children }) {
  return (
    <>
      <NavBar />
      <div>{children}</div>
    </>
  )
}
```

- Layout을 리턴하는 함수를 `components` 경로에서 생성
- `children`는 react.js가 제공하는 prop으로써, 하나의 컴포넌트를 또 다른 컴포넌트안에 주입할 때 사용

`_app.js`

```javascript
import Layout from '../components/Layout'

export default function MyApp({ Component, pageProps }) {
  return (
    <>
      <Layout>
        <Component {...pageProps} />
      </Layout>
    </>
  )
}
```

- 위에서 확인했던 `Layout.js`에서 `children` props에 해당하는 부분은 Layout 컴포넌트가 감싸고 하위 컴포넌트들을 전달함

### Custom Head Component

- Next.js가 기본적으로 제공하는 작은 단위의 패키지
- CRA(create-react-app) 구조였다면, 앱의 head 부분을 관리하기 위해서는 `react-helmet` 과 같은 것을 다운받아야 함
  - 새로운 컴포넌트로 인한 side effect 발생이 생길 수 있음
    `components/Seo.js`

```javascript
import Head from 'next/head'

export default function Seo({ title }) {
  return (
    <Head>
      <title>{title} | Next project</title>
    </Head>
  )
}
```

- Next.js가 제공하는 Head 컴포넌트에 `title` Element를 children으로 넘겨줌
- `<head>`tag와의 중복을 피하는게 좋음
- Seo 컴포넌트는 `title` props을 넘겨받아 브라우저 창 Head에 노출

## Fetching Data

### 실습 API 불러오기

- [TMDB API](https://www.themoviedb.org) 사용하기
- 가입 후 받은 API Key로 진행

### API Data 불러오기

```javascript
import { useEffect, useState } from 'react'
import Seo from '../components/Seo'

const API_KEY = 'APIKEY' // 할당 받은 TMDB API 입력

export default function Home() {
  const [movies, setMovies] = useState([])
  useEffect(() => {
    ;(async () => {
      const { results } = await (
        await fetch(`https://api.themoviedb.org/3/movie/popular?api_key=${API_KEY}`)
      ).json()
      setMovies(results)
    })()
  }, [])
  return (
    <div>
      <Seo title="Home" />
      <h1 className="active">Hello</h1>
      {!movies && <h4>Loading...</h4>}
      {movies?.map((movie) => (
        <div key={movie.id}>
          <h4>{movie.original_title}</h4>
        </div>
      ))}
    </div>
  )
}
```

- React Hook API 사용할 때를 제외하고 react를 따로 import 하지 않아도 됨
- state 값을 빈 배열 객체로 초기화고, useEffect hook을 사용
- async 함수로 Base URL과 쿼리 스트링(API_KEY) 조합으로 API Data를 fetch 함
  - API Data 중 `results`의 값을 구조 분해 할당을 통해 가져옴
  - movies array에 dispatch 함
- 가져온 데이터를 기반으로 Map 함수로 순회
  - movies data map 처리시 '.?' optional chaining 연산 처리
- 논리곱 연산자를 이용해 가져오는 데이터가 없을 경우 Loading 문구 노출

## Redirect, Rewrite

### Redirect

- 한 페이지에서 다른 페이지(URL)로 이동할 수 있게 함
- Next는 Redirection을 지원하고 앱 실행 시, Redirect가 가능한 서버도 같이 실행됨
  `next.config.js`

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  async redirects() {
    return [
      {
        source: '/old-blog/:path*',
        destination: '/new-blog/:path*',
        permanent: false,
      },
    ]
  },
}
```

- async 함수를 만들어 array를 return 함
- `source`
  - 유저가 이동하는 URL 경로를 지정
- `destination`
  - `source` 경로로 접근한 유저를 `destination`의 경로로 redirect 함
- `permanent`
  - 브라우저나 검색엔진이 이 정보를 기억하는지의 여부 결정
- path matching과 wildcard를 이용해서 catch할 수 있음

### Rewrite

- Redirect처럼 설정한 URL(`source`)에 Matching되면 `destination` 경로로 이동하지만, URL은 변하지 않음
  `next.config.js`

```javascript
/** @type {import('next').NextConfig} */
const API_KEY = process.env.NEXT_PUBLIC_API_KEY
const nextConfig = {
  reactStrictMode: true,
  async rewrites() {
    return [
      {
        source: '/api/movies',
        destination: `https://api.themoviedb.org/3/movie/popular?api_key=${API_KEY}`,
      },
    ]
  },
}
```

- 예시 코드처럼, redirect와 다르게 api key와 같은 URL parameter의 민감한 정보를 env 파일처리해서 숨길 수 있음

## Server Side Rendering

- Next.js는 Pre-render를 지원해서 CRA(create-react-app)과 달리 앱 초기 상태의 Page를 html 형태로 볼 수 있음
- CRA(create-react-app)는 자바스크립트를 다운받고 html을 렌더링하는 단점으로 사용자의 네트워크에 유연하게 대처하지 못함
- Next.js는 Server Side Rendering 방식을 선택해서, Server에서 일어나는 data 관련 작업이 모두 완료된 후 페이지를 로딩할 수 있음

### `getServerSideProps()`

- 함수의 이름은 다른 이름으로 바꿀 수 없음
- getServerSideProps Scope안에서 작성하는 코드는 Server 환경에서 실행됨
  - client쪽이 아닌 Server에서만 작동함
- Rewrite 예제에서 살펴봤던 `API_KEY`와 같은 민감한 정보를 백엔드에서 처리 할 수 있음
- redirect와, rewrite처럼 client에서 실행되는게 아니라서, `absoluteURL(절대 경로)`을 사용해야 함

`index.js`

```javascript
export default function Home({ results }) {}
export async function getServerSideProps() {
  const { results } = await (await fetch(`http://localhost:3000/api/movies`)).json()
  return {
    props: {
      results,
    },
  }
}
```

- `props`에 api 호출값인 `results`를 대입해서 object 형태로 반환함
- 리턴한 값은 페이지의 props 값으로 들어감
  - `_app.js`에 `pageProps`가 필요한 이유

#### 실행 순서 정리

1. 사용자는 페이지에 접근한다.
2. Next.js는 접근한 URL의 page component인 `Home`을 받는다.
3. render하기 위해 `_app.js`의 `<Component />`로 로드된다.
4. `getServerSideProps()`를 확인하고 호출한다.
5. 반환된 값을 `_app.js`의 `pageProps`가 받아서 page component로 전달한다.
6. 로드된 page component의 인자값으로 `pageProps` 즉 `results`를 전달 받는다.
   > Client Side Rendering을 선택할지, Server Side Rendering을 선택할 지는 본인 몫임

## Routes

- CRA 방식의 프로젝트와는 다르게 router를 따로 구성하지 않음

### Nested Routes

- project structure 상의 폴더와 파일 이름으로만 구성
  - ex) /movies/all
    - `pages/movies/all.js` 경로처럼 파일 구조상의 폴더와 파일을 생성(Nested Routes)
  - 페이지가 하나밖에 없다면 폴더를 만들지 않고, pages 하위에 파일만 생성

### Dynamic URL

- URL에 변수를 전달해서 동적으로 구성하고 싶을 때에는 다음과 같이 구성
  - ex) /movies/12
    - `pages/movies/[id].js`
      - 파일명에 `대괄호`를 넣고 넘기고 싶은 변수명을 작성

`[id].js`

```javascript
import { useRouter } from 'next/router'
export default function Detail() {
  const router = useRouter()
  console.log(router)
  return 'detail'
}
```

- useRouter 객체를 확인해서 `query` 프로퍼티의 id 키값을 보면 정상적으로 URL의 넘기고자하는 변수를 확인할 수 있음

### 예시

#### rewrites + useRouter 조합

`index.js`

```javascript
import Link from 'next/link'

export default function Home({ results }) {
  const router = useRouter()
  const onClick = (id) => {
    router.push(`/movies/${id}`)
  }
  return (
    <div className="container">
      {results?.map((movie) => (
        <div onclick={() => onClick(movie.id)} className="movie" key={movie.id}>
          <Link href={`/movies/${movie.id}`}>
            <a>{movie.original_title}</a>
          </Link>
        </div>
      ))}
    </div>
  )
}
```

- router 객체의 push method를 이용해 function 호출시 넘겨받은 인자값(id)와 함께 해당 url로 이동

`next.config.js`

```javascript
const API_KEY = process.env.NEXT_PUBLIC_API_KEY
const nextConfig = {
  reactStrictMode: true,
  async rewrites() {
    return [
      {
        source: '/api/movies/:id',
        destination: `https://api.themoviedb.org/3/movie/:id?api_key=${API_KEY}`,
      },
    ]
  },
}

module.exports = nextConfig
```

- `/api/movies/:id`와 Matching되는 URL 접근 시, 해당 destination으로 이동
- env 파일로 `API_KEY`가 은닉(마스킹)되었음
  > source의 변수 `:id`와 destination의 변수 `:id`는 동일해야 함

#### router.push Method Option `as` 사용

- router 객체의 push method `as` 옵션 사용해서 masking 처리

`index.js`

```javascript
import Link from 'next/link'

export default function Home({ results }) {
  const router = useRouter()
  const onClick = (id, title) => {
    router.push(
      {
        pathname: `/movies/${id}`,
        query: {
          title,
        },
      },
      `/movies/${id}`
    )
  }
  return (
    <div className="container">
      {results?.map((movie) => (
        <div onclick={() => onClick(movie.id)} className="movie" key={movie.id}>
          <Link href={`/movies/${movie.id}`}>
            <a>{movie.original_title}</a>
          </Link>
        </div>
      ))}
    </div>
  )
}
```

- push method에 객체를 열고 url(pathname, query)과 as option 값을 대입

#### 인라인으로 처리

```javascript
<Link
  href={{
    pathName: `/movies/${movie.id}`,
    query: {
      title: movie.original_title,
    },
  }}
  as={`/movies/${movie.id}`}
>
  <a>{movie.original_title}</a>
</Link>
```

- 해당 소스코드 처럼 함수를 따로 만들지 않고도 href 속성으로 객체를 전달할 수 있음

`[id].js`

```javascript
import { useRouter } from 'next/router'
export default function Detail() {
  const router = useRouter()
  return (
    <>
      <h4>{router.query.title || 'Loading...'}</h4>
    </>
  )
}
```

- 이동 대상이되는 파일로 넘어가서 router 객체의 query props title키를 화면에 출력

## Catch All

- Catch-All URL은 다양한 케이스의 URL Path를 유연하게 처리할 수 있음
- 유저가 홈페이지에서 특정 클릭 흐름을 통해 상세페이지에 들어오지 않아도 상세페이지의 내용을 볼 수 있음
- `[id].js` 파일명을 `[...params].js`로 수정
  - router에서 한 개의 파라미터를 받는 파일명을 배열 형태로 받아 페이지를 로딩할 수 있음

### Client-Side-Rendering

`[...params].js`

```javascript
import { useRouter } from 'next/router'
export default function Detail() {
  const router = useRouter()
  const [title, id] = router.query.params || []
  return (
    <>
      <h4>{title}</h4>
    </>
  )
}
```

- 유저가 접근하는 URL path가 params 배열형태로 넘어와서 각 변수에 할당
- 브라우저상에서 비공개모드로 접근하면, 백엔드에서 Pre-render가 되기 때문에 서버에는 아직 `router.query.params`가 존재하지 않아 오류가 발생
  - 논리합 연산자로 찾는 params 배열이 없다면 [] 빈 배열을 반환하게 처리
    > 컴포넌트 내부에서 router를 사용하면 router는 프론트에서만 실행됨

### Server-Side-Rendering

- CSR 방식으로 구현한다면, Search Engine에서는 단지 빈 html 태그만 보이므로 좋은 점수를 받지 못함
- 절충방안으로 `[...params].js` 페이지를 `index.js`처럼 특정 API를 호출하지는 않지만, `getServerSideProps()` 함수를 이용해서 처리할 수 있음

```javascript
export default function Detail({ params }) {
  const [title, id] = params || []
  return (
    <>
      <h4>{title}</h4>
    </>
  )
}

export function getServerSideProps({ params: { params } }) {
  return {
    props: {
      params,
    },
  }
}
```

- Url path에서 전달한 값을 `params` 배열형태로 받아 props로 리턴
- Detail component에서 prop으로 `params`을 받아서 각 변수에 할당

## 404 pages 생성

- 의도치 않은 URL에 접근했을 때, 띄워주는 error status code(404)를 `pages/404.js`의 이름으로 생성 반환
  `404.js`

```javascript
// pages/404.js
export default function Custom404() {
  return <h1>404 - Page Not Found</h1>
}
```

- [공식 문서 참고](https://nextjs.org/docs/advanced-features/custom-error-page#404-page)

> **Referenced**

- [Nomad coding - NextJS 시작하기](https://nomadcoders.co/nextjs-fundamentals)
