---
title: Webpack Optimized
date: '2022-04-15'
tags: ['Webpack']
draft: false
summary: 웹팩 최적화의 필요성, 최적화 방법
layout: PostSimple
authors: ['default']
---

# 최적화

- 소스 코드가 많아지면 번들링된 결과물도 커짐
  - 파일 다운로드 시간이 많이 소요되므로, 브라우저 성능에 영향을 줄 수 있음

## Production 모드

- 웹팩의 내장되어 있는 최적화 방법은 mode 값을 설정하는 방식이 가장 기본
- `development`는 디버깅 편의를 위해 두 개의 플러그인 사용

  - NamedChunksPlugin
  - NamedModulesPlugin
    > DefinePlugin을 사용하면 process.env.NODE_ENV 값이 `development`로 설정되어 애플리케이션에 전역변수로 주입

- `production`으로 설정하면 자바스크립트 결과물을 최소화 하기 위해 다음 일곱 개 플러그인 사용
  - FlagDependencyUsagePlugin
  - FlagIncludedChunksPlugin
  - ModuleConcatenationPlugin
  - NoEmitOnErrorsPlugin
  - OccurrenceOrderPlugin
  - SideEffectsFlgPlugin
  - TerserPlugin
    > DefinePlugin을 사용하면 process.env.NODE_ENV 값이 `production`로 설정되어 애플리케이션에 전역변수로 들어감

### 설정

`webpack.config.js`

```javascript
const mode = process.env.NODE_ENV || 'development' // 기본값을 development로 설정
module.exports = {
  mode,
}
```

- 빌드 시에 운영 모드로 설정하여 실행하도록 npm script 추가
  `package.json`

```json lines
{
  "scripts": {
    "build": "NODE_ENV=production webpack --progress"
  }
}
```

- 빌드가 완료된 후 `dist/main.js`를 가보면 production에서 제공하는 플러그인으로 인해서 코드가 난독화되어 있는 것을 확인할 수 있음

## OptimizeCssAssetsPlugin으로 최적화

- HtmlWebpackPlugin이 html 파일을 압축한 것처럼 css 파일도 빈칸을 없애는 압축을 하려면 `optimize-css-assets-webpack-plugin`을 사용

### 설치

```shell
$ npm install -D optimize-css-assets-webpack-plugin
```

### 적용

`webpack.config.js`

```javascript
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin')
module.exports = {
  optimization: {
    minimizer: mode === 'production' ? [new OptimizeCssAssetsPlugin()] : [],
  },
}
```

- 일반 plugin처럼 plugins에 적용하지않고, optimization에서 설정
- minimizer에는 여러개의 플러그인을 설정할 수 있기 때문에 배열로 설정
- webpack version 4 기준으로 해당 플러그인을 설정하면, 운영버전 시 js파일의 난독화가 해제되는 문제가 발생
  - 아래의 TerserWebpackPlugin으로 해결

## TerserWebpackPlugin으로 최적화

```shell
$ npm install -D terser-webpack-plugin
```

`webpack.config.js`

```javascript
const TerserPlugin = require('terser-webpack-plugin')
module.exports = {
  optimization: {
    minimizer:
      mode === 'production'
        ? [
            new TerserPlugin({
              terserOptions: {
                compress: {
                  drop_console: true, // 콘솔 로그 제거
                },
              },
            }),
            new OptimizeCssAssetsPlugin(),
          ]
        : [],
  },
}
```

- `production` mode에서 제공하는 `TerserPlugin`과 별개로 `TerserWebpackPlugin`을 설치하고 minimizer 배열에 추가함
  - optimization.minimizer를 다시 정의할 경우 웹팩에서 설정한 기본 플러그인 들을 덮어씀
  - TerserWebpackPlugin은 자바스크립트 코드를 난독화하고, debugger 구문을 제거함
    - 기본 설정외에도 콘솔 로그를 제거하는 옵션도 있음
  - [webpack v4 참고 사이트](https://v4.webpack.js.org/configuration/optimization/#optimizationminimizer)
  - [webpack v5 참고 사이트](https://webpack.js.org/configuration/optimization/#optimizationminimizer)
    - 웹팩 v5에서는 '...' 인자를 배열에 추가하면 기본 옵션을 사용하면서 플러그인을 추가

## 코드 스플리팅

- 코드 압축 이외에 결과물을 여러개로 쪼개면 좀 더 브라우저 다운로드 속도를 높일 수 있음

### 엔트리 분리

`webpack.config.js`

```javascript
module.exports = {
  entry: {
    main: './src/app.js',
    result: './src/result.js',
  },
  optimization: {
    // minimizer: ...,
    splitChunks: {
      chunks: 'all',
    },
  },
}
```

- 가장 단순한 것은 엔트리 포인트를 여러개로 분리
- `splitChunks.chunks`
  - 중복되는 코드를 제거하고 엔트리 포인트를 다시 빌드
  - `dist` 경로에 가면 엔트리 포인트간의 중복되는 코드를 `vendors` prefix 파일이 따로 생성되어 관리함
    > 개발자가 일일히 수동으로 작업하는 것은 비효율 적임, 다른 방법으로 자동화를 구성해야 함

### Dynamic Import 사용

`app.js`

```javascript
import form from './form'
// import result from "./result";
import './app.css'

let resultEl, formEl

document.addEventListener('DOMContentLoaded', async () => {
  formEl = document.createElement('div')
  formEl.innerHTML = form.render()
  document.body.appendChild(formEl)

  import(/* webpackChunkName: "result" */ './result.js').then(async (m) => {
    const result = m.default
    resultEl = document.createElement('div')
    resultEl.innerHTML = await result.render()
    document.body.appendChild(resultEl)
  })
})
```

- import 구분에 `/* webpackChunkName: "result" */` 주석을 달면 webpack이 자동으로 인식해서 따로 번들링 함
- 위에 엔트리 분리 방법보다는 해당 방법을 사용
- `dist` 경로에 가면 위의 방법과는 다르게 `ChunkName` 주석으로 지정해준 이름의 `vender~[filename]` 파일이 생성됨

### externals

- axios 같은 서드 파티 라이브러리를 빌드 프로세서에서 제외시키는 방법
- 웹팩 설정 중 externals를 사용
  `webpack.config.js`

```javascript
module.exports = {
  externals: {
    axios: 'axios',
  },
}
```

- 웹팩으로 빌드를 할 때 'axios' 모듈을 사용하는 부분이 있으면 전역변수 'axios' 사용으로 인식함
- axios를 전역 변수로 간주하려면 node_modules 안에 있는 axios 파일을 가져와야 함(복사)

#### copy-webpack-plugin 설치

```shell
$ npm install copy-webpack-plugin
```

#### copy-webpack-plugin 적용

`webpack.config.js`

```javascript
const CopyWebpackPlugin = require('copy-webpack-plugin')
module.exports = {
  plugins: [
    new CopyWebpackPlugin([
      {
        from: './node_modules/axios/dist/axios.min.js',
        to: './axios.min.js',
      },
    ]),
  ],
}
```

- 배열로 객체를 넘겨줌
  - from: 어디에 있는 것을 복사할 건지
  - to: output(./dist) 폴더에 저장
    `src/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document <%= env %></title>
  </head>
  <body>
    <script type="text/javascript" src="axios.min.js"></script>
  </body>
</html>
```

- `copy-webpack-plugin`으로 설정한 `to` 경로를 script 태그로 지정
- output 경로에 `axios.min.js`가 정상적으로 복사되었는지 확인
  > 웹팩에 빌드할 필요없는 외부 라이브러리는 `externals`로 빼는 것이 좋고 운영 환경 뿐만아니라, 개발 환경에서도 웹팩 빌드 시간을 줄여줌

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
