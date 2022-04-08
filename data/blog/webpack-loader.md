---
title: 로더
date: '2022-04-07'
tags: ['Webpack']
draft: false
summary:
layout: PostSimple
authors: ['default']
---

# 로더

## 로더의 역할

- 웹팩은 `모든 파일을 모듈로 바라봄` ⭐️⭐️⭐️⭐️⭐️
  - import 구문을 사용하면 자바스크립트 안으로 CSS, image, font까지 정적인 asset들을 가져올 수 있음
  - 웹팩의 로더 덕분에 타입 스크립트 같은 다른 언어를 변환해주거나 이미지를 data URL 형식의 문자로 변환해 줌
  - CSS 파일을 자바스크립트에서 직접 로딩할 수 있게 해줌

## 커스텀 로더 생성

#### `my-webpack-loader.js`

```javascript
module.exports = function myWebpackLoader(content) {
  console.log('myWebpackLoader가 동작함')
  return content
}
```

- 실행하고자 하는 함수를 module로 exports함
- 로더는 함수 형태로 작성
- 읽은 파일의 내용이 `content` arg로 넘어옴
- 로더의 동작확인을 위해 console.log로 확인

#### `webpack.config.js`

```javascript
const path = require('path')
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [path.resolve('./my-webpack-loader.js')],
      },
    ],
  },
  // ...
}
```

- 로더는 `module` 객체에 `rules`를 `배열` 형태로 추가
- `rules`는 `test`와 `use`라는 키를 가지고 있음
  - `test`
    - 로더가 처리해야할 파일명 패턴(정규 표현식 패턴)
    - 현재 코드는 `.js` 확장자를 가진 모든 파일은 로더로 돌림
  - `use`
    - 사용할 로더를 명시(Node.js `path` - 로더의 절대 경로)

## 커스텀 로더 동작 확인

```shell
$ npm run build
> webpack

myWebpackLoader가 동작함
myWebpackLoader가 동작함
```

- 두 번의 문자열이 찍히는 이유는 현재 파일 경로 상에 `app.js`, `math.js`라는 파일이 존재
- js 파일의 개수만큼 console.log의 출력을 확인할 수 있음(정상 동작)

### 웹 브라우저에서 확인해보기

- 확인 가능한 웹서버를 구동한 후 확인
- VS code의 Live Server 확장 도구를 설치해서 진행해보았음

```javascript
module.exports = function myWebpackLoader(content) {
  return content.replace('console.log(', 'alert(')
}
```

- console.log를 만나면 alert를 띄워주는 함수
- 브라우저에서 동작 확인

## 자주 사용하는 로더

### css-loader

- 앞서 기술한 바와 같이, 웹팩은 `모든 것`을 모듈로 바라봄
- css를 ES6 import 구문으로 불러올 수 있음

#### 로더 미적용 시

`src/app.js`

```javascript
import './app.css'
```

`src/app.css`

```css
body {
  background-color: green;
}
```

- ES6의 import 구문을 이용해 별도의 설정없이 불러와 본다면 어떤 일이 발생할까

```shell
$ npm run build
# ERROR in ./src/app.css 1:5
# Module parse failed: Unexpected token (1:5)
# You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
```

- 주석으로 처리 된 부분처럼 module을 parse할 수 없다고 에러메시지를 반환함
- webpack css 파일을 읽지 못해 발생하는 에러

#### css-loader 로더 적용하기

```shell
$ npm install css-loader
```

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['css-loader'],
      },
    ],
  },
  // ...
}
```

- css 확장자를 가진 패턴을 지정(정규 표현식 패턴)
- webpack은 엔트리 포인트부터 연결된 모든 모듈을 searching
  - css 파일을 발견하면 설정했던 css-loader 함수를 반환
- 설정이 끝났다면, npm build script를 통해 확인
- `dist/main.js`에 가서 css 코드(자바스크립트 문자열)도 확인

#### 브라우저 동작 확인

- VS code의 Live Server 확장 도구를 실행하여 확인
- Network탭의 main.js파일 확인을 해보면 자바스크립트 코드에 css 코드가 문자열 형태로 들어가 있음
  - html상에 보이지 않는 문제 발생
    - Html이 dom으로 변환되어야 브라우저에서 확인할 수 있듯이, CSS도 마찬가지로 OM이라는 형태로 변환해야 확인할 수 있음
    - 이 문제를 해결하기 위해 보통 `css-loader`와 같이 사용하는 `style-loader`가 등장

### style-loader

#### 설치

```shell
$ npm install style-loader
```

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  // ...
}
```

- 방금 설치한 `style-loader`를 배열형태로 삽입
- 실행순서는 아래에서부터 위

### file-loader

- 파일을 모듈 형태로 지원하고 웹팩 아웃풋에 파일을 옮겨주는 역할
- CSS에서 url() 함수에 이미지 파일 경로를 지정할 수 있는데 웹팩은 file-loader를 이용해서 해당 파일을 처리

#### 로더 미적용 시

`src/app.css`

```css
body {
  background-image: url(bg.png);
}
```

- background-image 속성에 url 형태로 지정하게 되면 아래와 같은 오류가 남

```shell
$ npm run build
# ERROR in ./src/bg.png 1:0
# Module parse failed: Unexpected character ''(1:0)
# You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
```

#### file-loader 로더 적용하기

```shell
$ npm install file-loader
```

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.png$/,
        use: ['file-loader'],
      },
    ],
  },
  // ...
}
```

- 파일 형태는 png이므로 새로운 파일 규칙을 지정하고, 방금 설치한 'file-loader'를 적용
- 설정을 수정했다면, `dist/main` 폴더에 가서 `bg.png` 파일을 확인
  - 웹팩은 빌드할때마다, uniq한 값을 생성하는 데 지정된 파일명은 hash 값
  - 성능을 위한 캐싱 처리로 인해 새로 갱신

#### 브라우저 동작 확인

- 개발자 도구로 확인하면 다음과 같이 파일 로딩 실패 에러가 남
  - `GET file//*.png net::ERR_FILE_NOT_FOUND`

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.png$/,
        loader: 'file-loader',
        options: {
          publicPath: './dist/',
          name: '[name].[ext]?[hash]',
        },
      },
    ],
  },
  // ...
}
```

- loader를 `file-loader`로 지정
- option에 `publicPath`, `name`을 지정
  - `publicPath`: 파일 로더가 처리하는 파일을 모듈로 사용했을 때, 경로 앞에 추가되는 문자열
  - `name`
    - 파일 로더가 file output에 복사할때 사용하는 파일이름
    - 마지막에 캐시의 무력화를 위해 쿼리 스트링(해시값) 추가

## url-loader

- 사용하는 이미지 갯수가 많으면 네트웍 리소스에 부담, 이는 곧 사이트 성능에 영향을 끼침
- 한 페이지에 작은 이미지 여러개를 사용한다면 Data URI Scheme을 사용
  - 이미지를 Base64로 인코딩하여 문자열 형태로 소스코드에 넣는 형식

### 설치

```shell
$ npm install -D url-loader
```

### url-loader 적용

`webpack.config.js`

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'url-loader',
        options: {
          publicPath: './dist/',
          name: '[name].[ext]?[hash]',
          limit: 20000, // 20kb
        },
      },
    ],
  },
  // ...
}
```

- 로더 설정은 `file-loader`와 비슷하게 진행
- 하나의 로더만 설정할때는 `loader`와 `options`를 사용
  - 두 개 이상 로더를 설정할 때는 `use` 옵션을 사용
- 20kb 미만 파일만 data url로 처리하는 `limit` 옵션을 적용
  - `./dist` 경로의 main.js에 data url 문자열로 저장
- 20kb 이상은 `file-loader`가 실행
  - `./dist` 경로의 image가 복사

> Webpack version이 5이상 `url-loader`, `file-loader`, `law-loader`는 기본 모듈로 채택되어 따로 설치할 필요 없습니다.\
> [Asset-module](https://webpack.js.org/guides/asset-modules/) 을 참고해서 개발 환경에 맞게 설정해야 합니다.

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
