---
title: 플러그인
date: '2022-04-08'
tags: ['Webpack']
draft: false
summary: webpack의 플러그인 및 자주 쓰이는 플러그인 살펴보기
layout: PostSimple
authors: ['default']
---

# 플러그인

## 플러그인의 역할

- 로더
  - 파일 단위의 처리
- 플러그인
  - 번들된 결과물을 처리
    - 자바스크립트 난독화, 특정 텍스트 추출

## 커스텀 플러그인 만들기

- [Webpack Plugin Sample](https://webpack.js.org/contribute/writing-a-plugin/)
  - 공식 사이트에서 제공하는 샘플 플러그인 코드를 참고
    `my-wepack-plugin.js`

```javascript
class MyWebpackPlugin {
  apply(compiler) {
    compiler.hooks.done.tap('My Plugin', (stats) => {
      console.log('MyPlugin: done')
    })
  }
}

module.exports = MyWebpackPlugin
```

- `apply` method에 webpack은 `compiler` 객체를 주입
- 플러그인이 완료됐을 때 동작하는 callback 함수 전달
- 내보낼 MyWebpackPlugin을 `module exports` 구문으로 처리

### 커스텀 플러그인 로드

`webpack.config.js`

```javascript
const MyWebpackPlugin = require('./my-webpack-plugin')
module.exports = {
  // ...
  plugins: [new MyWebpackPlugin()],
  // ...
}
```

- 커스텀 플러그인의 경로를 Node.js require 구문으로 불러옴
- module처럼 plugins 배열을 만들고 생성자 함수를 지정

#### 내장 플러그인 알아보기

`my-wepack-plugin.js`

```javascript
class MyWebpackPlugin {
  apply(compiler) {
    compiler.plugin('emit', (compilation, callback) => {
      const source = compilation.assets['main.js'].source()
      compilation.assets['main.js'].source = () => {
        const banner = [
          '/**',
          ' * 이것은 BannerPlugin이 처리한 결과입니다.',
          ' * Build Date: 2019-10-10',
          ' */',
        ].join('\n')
        return banner + '\n\n' + source
      }

      callback()
    })
  }
}
module.exports = MyWebpackPlugin
```

- compilation 객체에 webpack이 번들링한 결과물에 접근할 수 있음
- `main.js`가 갖게될 파일의 내용을 `source` 함수로 재정의하고 `banner`의 문자열, 줄바꿈을 `source` 함수로 반환
  - `dist/main.js`에 번들 결과 확인
    `javascript /** * 이것은 BannerPlugin이 처리한 결과입니다. * Build Date: 2019-10-10 */ `
    > 실제로 플러그인을 커스텀 하는 것보다, 이미 만들어진 플러그인을 어떤 것을 활용할 지 아는것이 더 중요

# 자주 사용하는 플러그인

## BannerPlugin

- 앞서, 커스텀했던 플러그인은 이미 웹팩이 제공하고 있는 플러그인

`webpack.config.js`

```javascript
const webpack = require('webpack')

module.exports = {
  // ...
  plugins: [
    new webpack.BannerPlugin({
      banner: '이것은 배너입니다.',
    }),
  ],
  // ...
}
```

- `webpack` 모듈을 가져와서 `plugins`에 생성자 함수로 객체를 전달함
- `dist/main.js`에 번들 결과 확인
  ```javascript
  /**
   * 이것은 배너입니다
   */
  ```
  - 배너 플러그인으로 생성한 build 파일

### 응용하기

```javascript
const webpack = require('webpack')
const childProcess = require('child_process')

module.exports = {
  // ...
  plugins: [
    new webpack.BannerPlugin({
      banner: `
        Build Date: ${new Date().toLocaleDateString()}
        Commit Version: ${childProcess.execSync('git rev-parse --short HEAD')}
        Author: ${childProcess.execSync('git config user.name')}
      `,
    }),
  ],
  // ...
}
```

- build date를 동적으로 생성하기 위해 Date 함수를 작성
- node 모듈중에 `child_process` 모듈을 가져와서 터미널 명령어를 실행
  - git을 사용한다면, commit 버전을 넘겨줄 수 있음
    ```shell
    $ git rev-parse --short HEAD
    # > git commit version hash key[6]
    ```
  - git에 등록된 유저의 이름도 추가
    `shell $ git config user.name # > git user name `
    > 빌드 및 실 배포 시, 캐싱이 제대로 처리됐는지 빌드가 잘 됐는지 확인하는 용도로 사용

## DefinePlugin

- 개발환경, 운영환경에 따라 API 서버 주소를 다르게 지정하는데 휴먼 에러를 예방하기 위해 의존적인 정보를 다른 곳에서 관리할 수 있게 함
- `BannerPlugin`처럼 webpack에서 기본적으로 제공하는 플러그인

`webpack.config.js`

```javascript
//...
plugins: [
  new webpack.DefinePlugin({
    TWO: JSON.stringify('1+1'),
    'api.domain': JSON.stringify('http://dev.api.domain.com'),
  }),
]
//...
```

- 다양한 타입을 지원

`app.js`

```javascript
console.log(process.env.NODE_ENV) // 'development'
// 개발 환경 확인 가능
console.log(TWO) // 2
// webpack.config.js에서 지정한 값을 호출할 수 있음
console.log(api.domain) // http://dev.api.domain.com
```

## HtmlTemplatePlugin

- HTML 파일 후처리하는데 사용
- 서드파티 플러그인, 별도의 설치 필요

### 설치 및 운용

```shell
$ npm install html-webpack-plugin
```

- `index.html`을 소스 경로에서 관리

`webpack.config.js`

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
//...
plugins: [
  new HtmlWebpackPlugin({
    template: './src/index.html',
  }),
]
//...
```

- 설치한 `HtmlWebpackPlugin`을 가져오고 Class 생성자이므로 `plugins` 배열에 추가
- option에서 template 경로를 전달
  - 변경한 `index.html` 경로를 지정

`dist/index.html`

```html
<script type="text/javascript" src="main.js"></script>
```

- `index.html`이 빌드한 폴더 내에 생성되어 있음
- 생성한 html에 생성하지 않았던 자바스크립트 로딩 코드가 들어가있음
- 빌드 시 html코드를 의존적이지 않고도 생성 가능

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
          // publicPath: './dist',
          name: '[name].[ext]?[hash]',
          limit: 20000, // 20kb
        },
      },
    ],
  },
  // ...
}
```

- 이전엔 `index.html`가 root path에 위치했기 때문에 `./dist`와 같은 prefix를 지정해야했지만, 이미 `./dist` 폴더 내 위치했기 때문에 `publicPath`는 삭제해도 됨

### `templateParameters`를 이용한 변수 호출

`index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document <%= env %></title>
  </head>
  <body></body>
</html>
```

- `<%= env %>`
  - EJS 문법을 사용해 webpack에서 사용할 변수를 호출할 수 있음

`webpack.config.js`

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
//...
plugins: [
  new HtmlWebpackPlugin({
    template: './src/index.html',
    templateParameters: {
      env: process.env.NODE_ENV === 'development' ? '(개발용)' : '',
    },
  }),
]
//...
```

- `templateParameters`에 `env` 값을 할당해서 환경에 따라 다른 문자열을 반환

```shell
$ NODE_ENV=development npm run build
```

- `NODE_ENV` 값을 `development`로 지정해서 build 실행

`dist/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document (개발용)</title>
  </head>
  <body></body>
</html>
```

- head > title element의 값이 `Document (개발용)`으로 바뀌어 있는 것을 확인

### `minify` 응용하기

```javascript
//...
plugins: [
  new HtmlWebpackPlugin({
    template: './src/index.html',
    templateParameters: {
      env: process.env.NODE_ENV === 'development' ? '(개발용)' : '',
    },
    minify:
      process.env.NODE_ENV === 'production'
        ? {
            collapseWhitespace: true,
            removeComments: true,
          }
        : false,
  }),
]
```

- 배포 환경에따라 주석, 공백과 같은 문자열을 제어할 수 있음
- `collapseWhitespace`
  - 공백 제거
- `removeComments`
  - 주석 제거

## CleanWebpackPlugin

- 매 빌드할때마다 쌓이는 output 경로의 파일을 정리해주는 플러그인

### 설치

```shell
$ npm install -D clean-webpack-plugin
```

### 적용

```javascript
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
// ...
plugins: [new CleanWebpackPlugin()]
// ...
```

## MiniCssExtractPlugin

- 한 곳에 응집되어있는 CSS 파일을 역할에 따라 분리해주는 플러그인
- 네트워크 상으로도 큰 파일 하나를 받는 것보다, 작은 파일 여러개를 받는 것이 성능에 효과적임

### 설치

```shell
$ npm install -D mini-css-extract-plugin
```

### 적용

`webpack.config.js`

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
// ...
module: {
  rules: [
    {
      test: /\.css$/,
      use: [
        process.env.NODE_ENV === 'production' ? MiniCssExtractPlugin.loader : 'style-loader',
        'css-loader',
      ],
    },
  ]
}
plugins: [
  ...(process.env.NODE_ENV === 'production'
    ? [new MiniCssExtractPlugin({ filename: '[name].css' })]
    : []),
]
// ...
```

- spread operator(...)를 사용하면 환경 변수에 따라 쉽게 변경 가능
- 일반 플러그인과 다르게 css 파일 처리 방식을 수정해야함
  - 이 부분도 역시 개발 환경에 따라 처리여부를 결정

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
