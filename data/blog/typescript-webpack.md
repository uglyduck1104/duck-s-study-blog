---
title: Typescript와 함께 Webpack 사용하기
date: '2022-12-14'
tags: ['typescript']
draft: false
summary: Webpack이란 무엇이며 왜 필요한가, Webpack 설치하기 & 중요 종속성, 입력 & 출력 구성 추가하기, ts-loader 패키지로 TypeScript 지원 추가하기, 설정 완료하기 & webpack-dev-server 추가하기, production mode로 빌드하기
layout: PostSimple
authors: ['default']
---

# Typescript**와 함께 Webpack 사용하기**

## Webpack이란 무엇이며 왜 필요한가

### 일반 프로젝트의 문제점

- 실제 프로젝트를 빌드할때, 불러오는 리소스 요청이 많으면 많을수록 베이스 오버헤드와 딜레이가 발생함
  - 파일 다운로드는 비교적 발리 진행되나, 요청을 설정하고 서버에서 수행되는 시간이 다소 소요됨
- HTTP 요청의 양만으로 인해 다량의 대기시간이 발생하고 프로젝트가 느려질 수 있음

### [Webpack](https://webpack.kr/)의 개념과 장점

- 파일을 묶고(`bundle`), 빌드하고(`building`) 통합(`orchestration`)해주는 도구로써, 여러개 파일에 걸쳐 코드를 나눌 수 있게 해줌
- 코드를 최적화하고 추가 빌드 툴을 제공함
- 가능한 짧은 변수와 함수 이름을 통해 보다, 간결한 코드로 작게 만들어 줌
  - 이는 사용자들이 리소스를 보다 빠르게 다운로드 할 수 있으며, 어플리케이션이 머신 상에서 더 빨리 시작될 수 있음

## Webpack 설치하기 & 중요 종속성

### 설치

```bash
$ npm install --save-dev webpack webpack-cli webpack-dev-server typescript ts-loader
```

- webpack을 사용하기 위해 3rd party library를 설치
- 개발을 효율적으로 할 수 있게 해주는 프로젝트, 워크플로우, 구성을 설정해줌
- `npm install —save-dev` 명령어를 통해 개발 전용 종속성이라는 것을 명시하고 종속성을 설치

### 확인 - `package.json`

```json
{
  // ...
  "devDependencies": {
    "lite-server": "^2.5.4",
    "ts-loader": "^9.4.2",
    "typescript": "^4.9.4",
    "webpack": "^5.75.0",
    "webpack-cli": "^5.0.1",
    "webpack-dev-server": "^4.11.1"
  }
}
```

- `devDependencies`의 종속성을 확인
  - `webpack`
    - 코드를 묶고 변환하기 위한 특정 기능을 플러그인 하도록 해줌
  - `webpack-cli`
    - 프로젝트에서 웹팩 명령어를 실행하기 위함
  - `webpack-dev-server`
    - 개발 서버 내에 빌트하기 위해서 개발 서버 종속성을 설치
    - 웹팩을 실행시키며, 파일의 변경사항 및 재컴파일을 시작함
  - `typescript`
    - `global`하게 설치한 `typscript` 버전과의 충돌을 방지하기 위해서, 프로젝트 마다 `typescript`를 설치해줌
  - `ts-loader`
    - 웹팩과 함께 동작할 패키지로써, 웹팩에게 어떻게 코드를 자바스크립트로 변환할 것인지 전달해줌

## 입력 & 출력 구성 추가하기

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "es2015",
    "lib": [
      "dom",
      "es6",
      "dom.iterable",
      "scripthost"
    ]
    "sourceMap": true,
    "outDir": "./dist",
    "removeComments": true,
    "noEmitOnError": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "skipLibCheck": true
  },
  "exclude": [
    "node_modules",
  ]
}
```

- 타겟은 `es6`로 설정이 되어있는 지 확인해야 함
  - 타입스크립트 로더의 웹팩이 타겟 값을 사용함
  - 코드가 구형 브라우저에서도 잘 동작하는 자바스크립트로 변환될지, 혹은 `ES6`로 신형 브라우저에서만 실행되는 코드로 변환될 지를 판단하는 기준
- 웹팩을 사용하기 위해서 `module`은 `es2015` 또는 `es6`여야 함
- `rootDir`는 필요하지 않음
  - 웹팩이 루트 파일 위치를 결정함

### `webpack.config.js`

```jsx
const path = require('path')

module.exports = {
  entry: './src/app.ts',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
}
```

- `webpack.config.js` 파일을 생성하여 웹팩이 자동으로 인식해서 프로젝트를 어떻게 동작하는지 명세
- `nodeJS`의 `export` 구문을 사용해서 `object` 형태로 `export`함
- 어느 파일에서 프로젝트가 시작하는지, 시작 지점(`entry point`)을 명시
- 프로젝트 실행 시 가장 먼저 시작되어야 하는 `app.ts`를 지정
  - `app.ts`의 로드되는 파일들의 `import`를 파악
  - `ts-loader` 패키지를 통해 파일의 모든 내용을 파악
- 웹팩을 정확하게 동작시키기 위해 모든 `import` 구문에서 확장자(`.js`)를 삭제
  - 웹팩은 확장자를 따로 추가하지 않고, 자동으로 `.js` 및 기타 확장자 파일을 찾음
  - 이중 확장자를 검색하지 않기 위해서 가급적이면 삭제하는 것을 권장
- `entry` 속성값을 사용해 오브젝트에 진입 속성을 추가함(`./src/app.ts`)
- `output` 속성값을 사용해 출력 설정 값 지정
  - 파일이름 명시
    - 브라우저내 캐시를 자동으로 생성 및 지원할 수 있음
  - `dist` 폴더는 절대 경로를 필요로하기 때문에, `nodeJS`의 `import` 구문(`require`)을 사용해 출력 지점을 설정
    - 특정 폴더로의 절대 경로를 필드하게 하기 위해 특수 상수를 사용(`__dirname`)

## ts-loader 패키지로 TypeScript 지원 추가하기

- 타입스크립트로 무엇을 할지 웹팩에 전달하기 위해서, `config` 오브젝트에 새로운 진입(`entry`)를 추가해야 함

### `webpack.config.js`

```jsx
const path = require('path')

module.exports = {
  // ...
  devtool: 'inline-source-map',
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.ts', '.js'],
  },
}
```

- 모듈도 하나의 파일이므로 `app.ts` 파일, 파일 내 `import`들을 어떻게 처리할 것인지를 전달함
- `devtool`
  - source map을 통해 개발자 도구에서 특정 코드에 `breake point`를 사용해 디버그할 수 있음
  - `tsconfig.json` 의 `sourceMap` 속성 값이 `true`가 선행되어야 함
- `rules: []`
  - 모든 파일에 적용할 여러개의 규칙들을 설정할 수 있도록 배열 형식을 추가
    - 더 복잡한 프로젝트에서는 더 많은 규칙을 사용하므로 배열 형식을 가져야 함
    - `test`
      - `ts-loader`를 사용하기 위해 특정 파일 포맷(`.ts`)을 `test` 속성을 이용해 정규표현식으로 지정
    - `use`
      - 웹팩이 사용하는 로더를 명시
      - 웹팩에게 찾은 모든 파일이 타입스크립트 로더로 처리되어야하는 것, 파일로 무엇을 하는지를 전달
    - `exclude`
      - 정규표현식을 사용해 웹팩이 `node_modules`을 찾이 않는 것을 명시

### 빌드 - `package.json`

```jsx
{
	"name": "understanding-typescript",
  "version": "1.0.0",
  "description": "Understanding TypeScript Course Setup",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "lite-server",
    "build": "webpack"
  },
	// ...
}
```

- `build` 이름의 `script`를 추가하고 `webpack`을 실행하면 `build`가 정상적으로 진행되는 것을 볼 수 있음
- `build`가 완료된 후 `npm start`를 통해 프로젝트를 실행함

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>ProjectManager</title>
    <link rel="stylesheet" href="app.css" />
    <script type="module" src="dist/bundle.js"></script>
  </head>
	<body></body>
</html>
```

- `script type`의 `src` 속성을 `dist/bundle.js` 경로 지정

## 설정 완료하기 & webpack-dev-server 추가하기

### `package.json`

```jsx
{
  //
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack serve",
    "build": "webpack"
  },
  //
}
```

- 기존의 `lite-server` 스크립트 대신 `webpack-dev-server` 혹은 `webpack serve`를 실행

### `webpack.config.js`

```jsx
const path = require('path')

module.exports = {
  mode: 'development',
  entry: './src/app.ts',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  devtool: 'inline-source-map',
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.ts', '.js'],
  },
  devServer: {
    static: {
      directory: path.join(__dirname, './'),
      watch: true,
    },
    compress: true,
    open: true,
    port: 8080,
  },
}
```

- `mode` 속성 값에 `development` 값을 할당해, `build` 시 `export` 될 모드를 지정
- `devServer` 속성 값에 `bundle.js` 리소스가 어디에 위치해있는지 지정하고, 서버 실행 시 웹 브라우저를 열지에 대한 옵션 값과 포트 번호를 추가로 지정할 수 있음

## production mode로 빌드하기

### `webpack.config.prod.js`

```jsx
const path = require('path')
const CleanPlugin = require('clean-webpack-plugin')

module.exports = {
  mode: 'production',
  entry: './src/app.ts',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.ts', '.js'],
  },
  plugins: [new CleanPlugin.CleanWebpackPlugin()],
}
```

- 개발 환경이 production일때를 관리하기 위해, 새로운 파일을 만들고 package.json에서 build script를 다음과 같이 수정
  - `"build": "webpack --config webpack.config.prod.js”`
- 개발 환경에서만 사용되는 속성인 `devServer`와 `devtool` 옵션을 제거
- 빌드 스크립트를 실행할 때마다 `dist` 폴더 내 최신 파일을 효율적으로 관리하기 위해 `3rd party library`인 `clean-webpack-plugin`을 설치하고 `plugins` 속성의 배열에 `new` 키워드와 함께 호출

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
