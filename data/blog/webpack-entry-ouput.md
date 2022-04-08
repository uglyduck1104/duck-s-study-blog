---
title: 엔트리/아웃풋
date: '2022-04-06'
tags: ['Webpack']
draft: false
summary: 웹팩은 여러개의 파일을 하나의 파일로 합쳐주는 번들러(bundler)의 역할
layout: PostSimple
authors: ['default']
---

# Webpack

- 웹팩은 여러개의 파일을 하나의 파일로 합쳐주는 번들러(bundler)의 역할
- 특정 시작점(entry point)으로부터 의존적인 모듈을 전부 찾아내고 하나의 결과물을 만듦

## 설치

```shell
$ npm install -D webpack webpack-cli // devDependencies에 설치
```

- 설치후 node_modules/.bin 폴더에 실행 가능한 명령어 확인
- --help 옵션 참고
  - ```shell
    $ node_modules/.bin/webpack --help
    ```

### `package.json`

```json lines
{
  //...
  "dependencies": {
    "react": "^18.0.0"
  },
  "devDependencies": {
    "webpack": "^5.71.0",
    "webpack-cli": "^4.9.2"
  }
}
```

- -D option으로 개발용 패키지 설치
- `node_modules/.bin` 폴더에 설치한 `webpack`, `webpack-cli`가 있음

### `node_modules/.bin/webpack --help` 보기

- `--mode` 옵션
  - `development`, `production`, `none` 모드를 선택할 수 있음
- `--entry` 옵션
  - module의 시작점을 칭함(필수)
- `--output` 옵션
  - 하나의 파일로 합치기 위해 저장되는 경로를 지정

### Bundling

```shell
node_modules/.bin/webpack --mode development --entry ./src/app.js --output dist/main
#[webpack-cli] Error: Unknown option '--output'
#[webpack-cli] Run 'webpack --help' to see available commands and options
```

- 스펙이 바뀌었는지, 해당 옵션을 가진 명령어를 찾지 못했다고 해서 다시 --help 옵션을 확인해보니 `--ouput-path <value>`로 명시되어 있음
- 지정한 `/dist/main.js`를 확인

#### `index.html`

```html
<script src="dist/main.js"></script>
```

- webpack으로 번들링 된 경로를 수정
- 모듈 인식 키워드 `type` 제거

#### `webpack.config.js` 생성

```javascript
const path = require('path')

module.exports = {
  mode: 'development',
  entry: {
    main: './src/app.js',
  },
  output: {
    path: path.resolve('./dist'), // output directory 명 절대 경로 호출
    filename: '[name].js',
  },
}
```

> 매번 webpack cli를 통한 bundling을 할 수 없기 때문에 `--config` 옵션을 참고해 설정파일을 생성\
> default file name: `webpack.config.js`

- ES6가 아닌 node의 모듈 시스템
- `path: path.resolve('./dist')`
  - node의 path 모듈을 가져와서 절대 경로 지정
  - 최상단에 노드 모듈을 가져옴
- `filename: '[name].js'`
  - 엔트리 설정한 key 값
  - 여러 엔트리에 대한 output 이름을 동적으로 생성

#### `package.json`

- webpack을 실행하기 위한 npm build script 추가

```json lines
{
  // ...
  "scripts": {
    "build": "webpack"
  }
  // ...
}
```

- `webpack`만 설정해도 npm은 현재 프로젝트에 있는 node module을 찾아서 명령어를 실행
  > 프로젝트 내에 package.json이 없을 경우\
  > `npm init -y`으로 기본값을 설정한 package.json 파일을 생성

> **Referenced**

- [인프런 - 프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)
