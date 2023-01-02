---
title: Node.js + Express 및 Typescript
date: '2023-01-02'
tags: ['typescript']
draft: false
summary: Node.js로 TypeScript 코드 실행하기, 프로젝트 설정하기, 설정 완료 및 타입 작업하기(Node + Express), 미들웨어 및 타입 추가하기, 컨트롤러 작업 및 요청 본문 구문 분석하기, 더 많은 CRUD 작업
layout: PostSimple
authors: ['default']
---

# Node.js + Express 및 Typescript

## Node.js로 TypeScript 코드 실행하기

### `app.ts`

```tsx
console.log('Something...')
```

- 콘솔에 log를 출력하는 간단한 스크립트는 브라우저 안에서도 동작하지만 노드 JS에서도 동일하게 동작함
- `tsc app.ts` 커맨드로 실행하면 `app.js` 파일이 생성되고 실행되는 모습을 볼 수 있음

### NodeJS는 타입스크립트를 실행할 수 없음

- 위와 같은 예제를 봤을때는 당연히 동작한다고 생각할 수 있지만, 아래의 예시를 보면 오류를 발생함

  ```tsx
  let age: number

  age = 30

  console.log(age)
  ```

  - 타입스크립트에서만 사용하는 타입 구문이 실행되지 않는 이유는 `NodeJS`는 자바스크립트 코드만 실행할 수 있기 때문이고 `ts-node`라는 패키지를 추가로 설치해서 컴파일러와 노드 실행이 가능한 코드를 만들 수 있음
  - 결국은 `tsc` 커맨드와 노드 커맨드를 하나로 합치는 결과를 얻을 수 있지만 프로덕션 환경에서는 파일들을 웹 서버에 호스팅하는 경우가 생길 수 있으므로, 추가적인 컴파일 단계가 코드 실행마다 발생할 경우에는 매번 실행 되는 것이 버거워 질 수 있음

## 프로젝트 설정하기

### 노드 모듈 초기화

- `npm init` 커맨드를 사용해 노드 앱 프로젝트의 의존성 초기화를 진행 → `package.json` 생성
  - 간단한 웹서버와 익스프레스 JS를 사용하기 위한 의존성 설지
  - `npm install express`
    - NodeJS의 프레임워크인 익스프레스 설치
  - `npm install body-parser`
    - `request body`를 전달하기 위한 의존성 설치
  - `npm install -D nodemon`
    - 노드 코드가 있는 파일을 실행하고 같은 폴더에 있는 파일들의 변화를 모니터링하여 노드 서버를 자동으로 재시작해 줌

### 타입 스크립트 초기화

- `tsc —init` 커맨드를 실행해서 타입스크립트 프로젝트로 초기화 → `tsconfig.json` 생성
- `tsconfig.json`

  ```json
  {
    "compilerOptions": {
      /* Basic Options */
      // "incremental": true,                   /* Enable incremental compilation */
      "target": "es2018" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019' or 'ESNEXT'. */,
      "module": "commonjs",
      "moduleResolution": "node" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */,
      // "lib": [],                             /* Specify library files to be included in the compilation. */
      // "allowJs": true,                       /* Allow javascript files to be compiled. */
      // "checkJs": true,                       /* Report errors in .js files. */
      // "jsx": "preserve",                     /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */
      // "declaration": true,                   /* Generates corresponding '.d.ts' file. */
      // "declarationMap": true,                /* Generates a sourcemap for each corresponding '.d.ts' file. */
      // "sourceMap": true,                     /* Generates corresponding '.map' file. */
      // "outFile": "./",                       /* Concatenate and emit output to single file. */
      "outDir": "./dist" /* Redirect output structure to the directory. */,
      "rootDir": "./src" /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */,
      // "composite": true,                     /* Enable project compilation */
      // "tsBuildInfoFile": "./",               /* Specify file to store incremental compilation information */
      // "removeComments": true,                /* Do not emit comments to output. */
      // "noEmit": true,                        /* Do not emit outputs. */
      // "importHelpers": true,                 /* Import emit helpers from 'tslib'. */
      // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
      // "isolatedModules": true,               /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

      /* Strict Type-Checking Options */
      "strict": true /* Enable all strict type-checking options. */,
      // "noImplicitAny": true,                 /* Raise error on expressions and declarations with an implied 'any' type. */
      // "strictNullChecks": true,              /* Enable strict null checks. */
      // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
      // "strictBindCallApply": true,           /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
      // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
      // "noImplicitThis": true,                /* Raise error on 'this' expressions with an implied 'any' type. */
      // "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */

      /* Additional Checks */
      // "noUnusedLocals": true,                /* Report errors on unused locals. */
      // "noUnusedParameters": true,            /* Report errors on unused parameters. */
      // "noImplicitReturns": true,             /* Report error when not all code paths in function return a value. */
      // "noFallthroughCasesInSwitch": true,    /* Report errors for fallthrough cases in switch statement. */

      /* Module Resolution Options */
      // "moduleResolution": "node",            /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
      // "baseUrl": "./",                       /* Base directory to resolve non-absolute module names. */
      // "paths": {},                           /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
      // "rootDirs": [],                        /* List of root folders whose combined content represents the structure of the project at runtime. */
      // "typeRoots": [],                       /* List of folders to include type definitions from. */
      // "types": [],                           /* Type declaration files to be included in compilation. */
      // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
      "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */,
      // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */
      // "allowUmdGlobalAccess": true,          /* Allow accessing UMD globals from modules. */

      /* Source Map Options */
      // "sourceRoot": "",                      /* Specify the location where debugger should locate TypeScript files instead of source locations. */
      // "mapRoot": "",                         /* Specify the location where debugger should locate map files instead of generated locations. */
      // "inlineSourceMap": true,               /* Emit a single file with source maps instead of having a separate file. */
      // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

      /* Experimental Options */
      // "experimentalDecorators": true,        /* Enables experimental support for ES7 decorators. */
      // "emitDecoratorMetadata": true,         /* Enables experimental support for emitting type metadata for decorators. */

      /* Advanced Options */
      "forceConsistentCasingInFileNames": true /* Disallow inconsistently-cased references to the same file. */
    }
  }
  ```

  - 프로젝트에서 사용할 `ECMA` 설정과 `rootDir`, `outDir`를 필수적으로 지정함
    - 타입스크립트 코드와 노드로 실행될 출력되는 실제 자바스크립트 코드 분리
  - `moduleResolution` 옵션을 사용해 필히 노드로 설정해야 타입스크립트가 프로젝트 파일과 import가 어떻게 작동하는지 알 수 있음

## 설정 완료 및 타입 작업하기(Node + Express)

### `src/app.ts`

```tsx
import express from 'express'
// const express = require('express')

const app = express()

app.listen(3000)
```

- 서버를 시작하는 루트 엔트리 포인트로써, `express`를 `import` 한 후, `function`으로 실행할 수 있음
- 앱 오브젝트가 생긴 후 특정한 포트에서 들어오는 요청을 `listen()` 메소드를 호출할 수 있음
- `require`는 `NodeJS`에서는 존재하는 `function`지만, 브라우저 측면에서는 없으므로(`tsconfig.json → lib: []` 확인) 타입스크립트는 인지하지 못함
  - 추가 적인 타입 설치 필요
    - `npm install -D @types/node`를 추가해 `NodeJS`로 작업하는데 필요한 타입을 모두 설치
- `app function`에 커서를 올려대면 타입스크립트는 `any`로 추론하는 문제 발생
  - `npm install -D @types/express`를 설치한 후, `require` 구문에서 `import` 구문으로 사용해서 `express`를 가져옴
  - `NodeJS`에는 `commonJS`를 사용하는 것이 `import`와 `export`를 추가하는 가장 흔한 방법임
    - 정식으로 지원하지는 않음

## 미들웨어 및 타입 추가하기

### `routes/todo.ts`

```tsx
import { Router } from 'express'

const router = Router()

router.post('/')

router.get('/')

router.patch('/:id')

router.delete('/:id')

export default router
```

- 들어오는 요청을 `route` 하거나 `point`해서 특정한 로직을 실행 할 수 있음
- 각 실행될 `HTTP` 메소드 요청을 먼저 정의(`endpoint`)

### `app.ts`

```tsx
import express, { Request, Response, NextFunction } from 'express'
import todoRoutes from './routes/todos'

const app = express()

app.use('/todos', todoRoutes)

app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  res.status(500).json({ message: err.message })
})

app.listen(3000)
```

- 정의한 라우트를 `import`하고 `app.use()`를 사용해 실행되는 익스프레스 앱을 연결 및 요청에 따라 `todoRoutes`로 포워딩
- `요청`, `응답`, `에러`, `NextFunction` 등 미들웨어가 작동하는 데 필요한 매개 변수
- 넓은 타입에서 좁히기 위해 `express` 프레임워크에서 제공하는 타입들을 지정시켜 줌

## 컨트롤러 작업 및 요청 본문 구문 분석하기

- `router`에 따른 로직을 구성하기 위해 `controller` 폴더를 만들고 분리

### `controller/todos.ts`

```tsx
import { RequestHandler } from 'express'

import { Todo } from '../models/todo'

const TODOS: Todo[] = []

export const createTodo: RequestHandler = (req, res, next) => {
  const text = (req.body as { text: string }).text
  const newTodo = new Todo(Math.random().toString(), text)

  TODOS.push(newTodo)

  res.status(201).json({ message: 'Created the todo.', createTodo: newTodo })
}
```

- `RequestHandler`로 `export` 하고자 하는 `method`에 타입 지정을 하면 `req`, `res`, `next`를 따로 타입으로 지정하지 않아도 됨
  - `import`는 단순히 데이터와 타입을 가져오므로 컴파일 과정에서는 사라지는 구문
- `Todo model` 파일을 만들어 `todo`가 어떻게 생겼는지 정의

  - `models/todo.ts`

    ```tsx
    export class Todo {
      constructor(public id: string, public text: string) {}
    }
    ```

- `TODOS`가 `Todo` 배열 타입임을 타입스크립트에게 알려줌
- `const newTodo = new Todo(Math.random().toString(), text);`
  - 새로운 할일을 인스턴스화를 통해 만들고, 전달할 아이디를 무작위로 생성한 후 두번째 매개변수에 요청 바디에 있을 `text`를 전달
- `const text = (req.body as { text: string }).text;`
  - 타입스크립트가 `request.body.text`에 저장될 값을 `any`로 보기 때문에 타입캐스팅을 통해 문자열임을 알려줌
- `TODOS.push(newTodo);`
  - 새로운 할일을 배열에 추가함
- `res.status(201).json({message: 'Created the todo.', createTodo: newTodo});`
  - 상태 코드(`201`)와 `message`, `createTodo`를 클라이언트에 전달

### `app.ts`

```tsx
import express, { Request, Response, NextFunction } from 'express'
import { json } from 'body-parser'

import todoRoutes from './routes/todos'

const app = express()

app.use(json())

app.use('/todos', todoRoutes)

app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  res.status(500).json({ message: err.message })
})

app.listen(3000)
```

- `app.use(json());`
  - 우리가 추출하고 싶은 body가 실제로 존재하는지 body-parser를 통해 파싱(분석)해야함
  - 요청에서 찾는 `JSON` 데이터를 추출하고 `JSON` 데이터를 요청 `body`에 채움

## 더 많은 CRUD 작업

### `routes/todos.ts`

```tsx
import { Router } from 'express'

import { createTodo, getTodos, updateTodo, deleteTodo } from '../controllers/todos'

const router = Router()

router.post('/', createTodo)

router.get('/', getTodos)

router.patch('/:id', updateTodo)

router.delete('/:id', deleteTodo)

export default router
```

### `controllers/todos.ts`

```tsx
import { RequestHandler } from 'express'

import { Todo } from '../models/todo'

const TODOS: Todo[] = []

export const createTodo: RequestHandler = (req, res, next) => {
  const text = (req.body as { text: string }).text
  const newTodo = new Todo(Math.random().toString(), text)

  TODOS.push(newTodo)

  res.status(201).json({ message: 'Created the todo.', createTodo: newTodo })
}

export const getTodos: RequestHandler = (req, res, next) => {
  res.json({ todos: TODOS })
}

export const updateTodo: RequestHandler<{ id: string }> = (req, res, next) => {
  const todoId = req.params.id

  const updateText = (req.body as { text: string }).text

  const todoIndex = TODOS.findIndex((todo) => todo.id === todoId)
  if (todoIndex < 0) {
    throw new Error('Could not find todo!')
  }

  TODOS[todoIndex] = new Todo(TODOS[todoIndex].id, updateText)

  res.json({ message: 'Updated!', updatedTodo: TODOS[todoIndex] })
}

export const deleteTodo: RequestHandler = (req, res, next) => {
  const todoId = req.params.id

  const todoIndex = TODOS.findIndex((todo) => todo.id === todoId)
  if (todoIndex < 0) {
    throw new Error('Could not find todo!')
  }

  TODOS.splice(todoIndex, 1)

  res.json({ message: 'Todo deleted!' })
}
```

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
