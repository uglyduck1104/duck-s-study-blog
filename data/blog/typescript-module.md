---
title: 모듈 및 네임스페이스
date: '2022-11-29'
tags: ['typescript']
draft: false
summary: 여러개의 코드 쪼개기, 네임스페이스 작업하기, 파일 및 폴더 정리하기, 네임스페이스 가져오기 문제, ES 모듈 사용하기, 다양한 import, export 구문 이해하기
layout: PostSimple
authors: ['default']
---

# 모듈 및 네임스페이스

## 여러개의 코드 쪼개기

- 코드를 여러개의 파일로 나누는 3가지 방법
  1. 단순하게 여러개의 코드 파일, 여러개의 타입스크립트 파일과 폴더를 작성하고 모든 코드 파일을 자동으로 소스 디렉토리에 컴파일한 후 수동으로 컴파일된 자바스크립트 파일을 HTML로 가져오기
     - 모든 불러오는 항목을 수동으로 관리해야하는 번거로움이 발생
     - 특정 유형의 경우 타입 지원이 되지 않은 이슈가 발생
  2. 네임스페이스(`namespace`)와 파일 번들링(`file bundling`) 사용하기(Typescript 기능)
     - 네임스페이스(`namespace`) 신택스(`syntax`) 코드를 사용해 그룹화
     - `HTML` 파일 내의 서로 다른 `import`를 수동으로 관리할 필요가 없음
  3. ES6 module `import`, `export` 사용
     - `import`, `export` 신택스(`syntax`) 코드를 사용
     - 단 `1`개의 스크립트 `import`만 필요함
       - 모던 브라우저가 모든 의존관계를 불러옴
     - `namespace`와 마찬가지로 `import`를 수동으로 관리하지 않아도 됨

## 네임스페이스 작업하기

### `src/app.ts`

```tsx
// Drag & Drop Interfaces
interface Draggable {
  dragStartHandler(event: DragEvent): void
  dragEndHandler(event: DragEvent): void
}

interface DragTarget {
  dragOverHandler(event: DragEvent): void
  dropHandler(event: DragEvent): void
  dragLeaveHandler(event: DragEvent): void
}
//...
```

- `Drag & Drop Interfaces`를 `app.ts`에서 분리

### `src/drag-drop-interfaces.ts`

```tsx
// Drag & Drop Interfaces
namespace App {
  export interface Draggable {
    dragStartHandler(event: DragEvent): void
    dragEndHandler(event: DragEvent): void
  }

  export interface DragTarget {
    dragOverHandler(event: DragEvent): void
    dropHandler(event: DragEvent): void
    dragLeaveHandler(event: DragEvent): void
  }
}
```

- 동일한 경로에 `drag-drop-interfaces.ts` 파일을 생성하고 `namespace App {}`으로 감싸줌
- 타입스크립트는 기본적으로 `object`로 컴파일되며, `namespace`에 입력하는 것들은 속성에 저장됨
- 오직 `App namespace` 내부에서만 이용이 가능함
  - 네임스페이스에서의 기능을 `export` 키워드를 통해 외부에서 사용할 수 있게 해야 함

### `app.ts` - reference syntax

```tsx
/// <reference path="drag-drop-interfaces.ts" />

namespace App {
  //...
}
```

- `app.ts` 파일의 최상단에 슬래시(`/`)를 3개 추가해 타입스크립트가 이해하고 인식할 수 있게 해야 함
- 또한, `import`된 네임스페이스(App)에 있는 값을 사용하려면, `app.ts`도 동일한 이름의 `namespace`로 감싸줘야 함

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es6" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */,
    "module": "amd" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */,
    "lib": [
      "dom",
      "es6",
      "dom.iterable",
      "scripthost"
    ]
    "sourceMap": true /* Generates corresponding '.map' file. */,
    "outFile": "./dist/bundle.js",                       /* Concatenate and emit output to single file. */
    "outDir": "./dist" /* Redirect output structure to the directory. */,
    "rootDir": "./src" /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */,
  },
  "exclude": [
    "node_modules", // would be the default
  ]
}
```

- `namespace`에서 `import` 한 것은 단지 타입스크립트에 어디서 해당 타입을 찾아야 하는지 전달해준 것뿐, 자바스크립트로 컴파일되면 연결이 무너지므로 `tsconfig`에서 `outFile` 옵션을 지정해줘야함
- `outFile` 옵션은 타입스크립트가 네임스페이스와 연결되도록 함
  - 여러개 자바스크립트 파일을 컴파일하는 대신에 컴파일 중에 연결되는 참조(reference)들을 하나의 자바스크립트 파일로 연결
    - `./disk/bundle.js`
    - `index.html`
      - `<script src="dist/bundle.js" defer></script>`
  - 더불어 `module` 키 값을 `amd`로 설정해야 함
- 설정값을 제거하기 위해 `dist` 폴더안의 모든 파일을 삭제한 후, `tsc -w` 옵션으로 재시작해 컴파일을 진행하여 에러를 해결

## 파일 및 폴더 정리하기

### `app.ts`

```tsx
/// <reference path="models/drag-drop.ts" />
/// <reference path="models/project.ts" />
/// <reference path="state/project-state.ts" />
/// <reference path="util/validation.ts" />
/// <reference path="decorators/autobind.ts" />
/// <reference path="components/project-input.ts" />
/// <reference path="components/project-list.ts" />

namespace App {
  new ProjectInput()
  new ProjectList('active')
  new ProjectList('finished')
}
```

- `subfolder`를 두어, 관심사가 같은 타입스크립트끼리 추상화
- `Component`를 상속받는 클래스 `project-input`, `project-item`, `project-list`는 파일마다 참조값을 지정
  - `/// <reference path="base-component.ts" />`

## 네임스페이스 가져오기 문제

- 모든 의존성을 `app.ts(global)`에서 참조하기 때문에 각각의 파일이 공통적으로 요구하는 `project.ts`, `drag-drop.ts`, `project-state` 등과 같은 파일을 로드할 수 있는데, 만약에 의존성을 글로벌 파일로 관리하지 않고 각각의 파일로 관리한다면 네임스페이스는 최선의 방법은 아닐 수 있음
  - 또한, 의존성이 누락됐을 경우 에러 없이 컴파일 되므로 무엇이 잘못됐는지 직관적으로 알기 어려움

## ES 모듈 사용하기

### `tsconfig.json`

```tsx
{
  "compilerOptions": {
    "target": "es6" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */,
    "module": "es2015" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */,
    "lib": [
      "dom",
      "es6",
      "dom.iterable",
      "scripthost"
    ]
    "sourceMap": true /* Generates corresponding '.map' file. */,
    // "outFile": "./dist/bundle.js",                       /* Concatenate and emit output to single file. */
    "outDir": "./dist" /* Redirect output structure to the directory. */,
    "rootDir": "./src" /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */,
  },
  "exclude": [
    "node_modules", // would be the default
  ]
}
```

- `outFile` Option은 ES 모듈로 지원되지 않으므로, 주석처리하고 `module` 키 값을 `es2015`로 변경
- `index.html`
  - `<script type="module" src="dist/app.js"></script>`
  - `script type`을 `module`로 지정해서, 모듈 외부에서 `import` 표현식을 사용할 수 있게 해야 함

### 모든 파일

1. namespace로 감싸고 있는 중괄호를 제거해줌
2. `import {} from “./<file_name>.js`

- ES 모듈 `syntax`인 `import` 표현식을 사용해 의존성을 추가해주고 꼭 파일명 뒤에 `js` 확장자를 붙여 이미 컴파일된 `js` 파일을 대상으로하게 명시해야 함

## 다양한 import, export 구문 이해하기

- `import * as Validation from '../util/validation.js';`
  - 별칭(`alias`)을 사용해 파일로부터 모든 것을 가져와 `.` 표기법으로 로드할 수 있음
- `import { autobind as Autobind } from '../decorators/autobind.js';`
  - 별칭을 사용해 특정 이름을 사용해 이름간의 충돌을 방지할 수 있음
- `export default`
  - 파일당 1개만 `export`하는 경우 사용하며, import시, 중괄호로 감싸지 않고 가져올 수 있음

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
