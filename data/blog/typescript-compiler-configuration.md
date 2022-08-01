---
title: TypeScript 컴파일러 및 구성
date: '2022-08-01'
tags: ['typescript']
draft: false
summary: 타입스크립트 컴파일러와 컴파일러를 구성하는 옵션에 대한 내용
layout: PostSimple
authors: ['default']
---

# TypeScript 컴파일러 및 구성

## Watch Mode

- 변경 사항이 있을 때마다 대상이 되는 파일의 tsc 커맨드를 일일히 실행하는 것은 번거로운 점이 있음
  - `Watch Mode`를 통해 파일을 관찰하고 파일에 변경 사항이 있을때마다 다시 컴파일함

```bash
$ tsc app.ts --watch
```

- 대상이 되는 파일 뒤에 `—watch` 혹은 `-w` 옵션을 통해 `watch mode`를 실행할 수 있음
- 해당 부분도 역시 파일을 구체적으로 지정해야 하므로 규모가 비교적 큰 프로젝트에서는 사용하지 않음

## 전체 프로젝트 컴파일 / 다수의 파일

- 위에서 본 바와 같이 `tsc` 커맨드는 파일을 지정하지 않고도 사용이 가능한데, 특정 파일을 지정하지 않은 경우 `관찰 모드`로 전체 프로젝트 폴더를 확인하고 변경사항이 적용될 수 있는 `모든 타입스크립트 파일을 다시 컴파일` 함

### 타입스크립트 프로젝트 지정

```bash
$ tsc --init
```

- 프로젝트를 타입스크립트 프로젝트라고 `최초 지정`한 후 한 번만 실행하면 되는데 즉, 커맨드가 실행되는 폴더의 `모든 항목을 알려주는 역할`을 함
- 실행 한 후에는 `tsconfig.json` 파일이 생성됨

### tsconfig.json

- 타입 스크립트가 관리해야 하는 파일이 포함된 프로젝트와 폴더의 모든 하위 폴더를 참고하기 위한 파일

  ```json
  {
    "compilerOptions": {
      /* Visit https://aka.ms/tsconfig to read more about this file */

      /* Projects */
      // "incremental": true,                              /* Save .tsbuildinfo files to allow for incremental compilation of projects. */
      // "composite": true,                                /* Enable constraints that allow a TypeScript project to be used with project references. */
      // "tsBuildInfoFile": "./.tsbuildinfo",              /* Specify the path to .tsbuildinfo incremental compilation file. */
      // "disableSourceOfProjectReferenceRedirect": true,  /* Disable preferring source files instead of declaration files when referencing composite projects. */
      // "disableSolutionSearching": true,                 /* Opt a project out of multi-project reference checking when editing. */
      // "disableReferencedProjectLoad": true,             /* Reduce the number of projects loaded automatically by TypeScript. */

      /* Language and Environment */
      "target": "es2016" /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */,
      // "lib": [],                                        /* Specify a set of bundled library declaration files that describe the target runtime environment. */
      // "jsx": "preserve",                                /* Specify what JSX code is generated. */
      // "experimentalDecorators": true,                   /* Enable experimental support for TC39 stage 2 draft decorators. */
      // "emitDecoratorMetadata": true,                    /* Emit design-type metadata for decorated declarations in source files. */
      // "jsxFactory": "",                                 /* Specify the JSX factory function used when targeting React JSX emit, e.g. 'React.createElement' or 'h'. */
      // "jsxFragmentFactory": "",                         /* Specify the JSX Fragment reference used for fragments when targeting React JSX emit e.g. 'React.Fragment' or 'Fragment'. */
      // "jsxImportSource": "",                            /* Specify module specifier used to import the JSX factory functions when using 'jsx: react-jsx*'. */
      // "reactNamespace": "",                             /* Specify the object invoked for 'createElement'. This only applies when targeting 'react' JSX emit. */
      // "noLib": true,                                    /* Disable including any library files, including the default lib.d.ts. */
      // "useDefineForClassFields": true,                  /* Emit ECMAScript-standard-compliant class fields. */
      // "moduleDetection": "auto",                        /* Control what method is used to detect module-format JS files. */

      /* Modules */
      "module": "commonjs" /* Specify what module code is generated. */,
      // "rootDir": "./",                                  /* Specify the root folder within your source files. */
      // "moduleResolution": "node",                       /* Specify how TypeScript looks up a file from a given module specifier. */
      // "baseUrl": "./",                                  /* Specify the base directory to resolve non-relative module names. */
      // "paths": {},                                      /* Specify a set of entries that re-map imports to additional lookup locations. */
      // "rootDirs": [],                                   /* Allow multiple folders to be treated as one when resolving modules. */
      // "typeRoots": [],                                  /* Specify multiple folders that act like './node_modules/@types'. */
      // "types": [],                                      /* Specify type package names to be included without being referenced in a source file. */
      // "allowUmdGlobalAccess": true,                     /* Allow accessing UMD globals from modules. */
      // "moduleSuffixes": [],                             /* List of file name suffixes to search when resolving a module. */
      // "resolveJsonModule": true,                        /* Enable importing .json files. */
      // "noResolve": true,                                /* Disallow 'import's, 'require's or '<reference>'s from expanding the number of files TypeScript should add to a project. */

      /* JavaScript Support */
      // "allowJs": true,                                  /* Allow JavaScript files to be a part of your program. Use the 'checkJS' option to get errors from these files. */
      // "checkJs": true,                                  /* Enable error reporting in type-checked JavaScript files. */
      // "maxNodeModuleJsDepth": 1,                        /* Specify the maximum folder depth used for checking JavaScript files from 'node_modules'. Only applicable with 'allowJs'. */

      /* Emit */
      // "declaration": true,                              /* Generate .d.ts files from TypeScript and JavaScript files in your project. */
      // "declarationMap": true,                           /* Create sourcemaps for d.ts files. */
      // "emitDeclarationOnly": true,                      /* Only output d.ts files and not JavaScript files. */
      // "sourceMap": true,                                /* Create source map files for emitted JavaScript files. */
      // "outFile": "./",                                  /* Specify a file that bundles all outputs into one JavaScript file. If 'declaration' is true, also designates a file that bundles all .d.ts output. */
      // "outDir": "./",                                   /* Specify an output folder for all emitted files. */
      // "removeComments": true,                           /* Disable emitting comments. */
      // "noEmit": true,                                   /* Disable emitting files from a compilation. */
      // "importHelpers": true,                            /* Allow importing helper functions from tslib once per project, instead of including them per-file. */
      // "importsNotUsedAsValues": "remove",               /* Specify emit/checking behavior for imports that are only used for types. */
      // "downlevelIteration": true,                       /* Emit more compliant, but verbose and less performant JavaScript for iteration. */
      // "sourceRoot": "",                                 /* Specify the root path for debuggers to find the reference source code. */
      // "mapRoot": "",                                    /* Specify the location where debugger should locate map files instead of generated locations. */
      // "inlineSourceMap": true,                          /* Include sourcemap files inside the emitted JavaScript. */
      // "inlineSources": true,                            /* Include source code in the sourcemaps inside the emitted JavaScript. */
      // "emitBOM": true,                                  /* Emit a UTF-8 Byte Order Mark (BOM) in the beginning of output files. */
      // "newLine": "crlf",                                /* Set the newline character for emitting files. */
      // "stripInternal": true,                            /* Disable emitting declarations that have '@internal' in their JSDoc comments. */
      // "noEmitHelpers": true,                            /* Disable generating custom helper functions like '__extends' in compiled output. */
      // "noEmitOnError": true,                            /* Disable emitting files if any type checking errors are reported. */
      // "preserveConstEnums": true,                       /* Disable erasing 'const enum' declarations in generated code. */
      // "declarationDir": "./",                           /* Specify the output directory for generated declaration files. */
      // "preserveValueImports": true,                     /* Preserve unused imported values in the JavaScript output that would otherwise be removed. */

      /* Interop Constraints */
      // "isolatedModules": true,                          /* Ensure that each file can be safely transpiled without relying on other imports. */
      // "allowSyntheticDefaultImports": true,             /* Allow 'import x from y' when a module doesn't have a default export. */
      "esModuleInterop": true /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */,
      // "preserveSymlinks": true,                         /* Disable resolving symlinks to their realpath. This correlates to the same flag in node. */
      "forceConsistentCasingInFileNames": true /* Ensure that casing is correct in imports. */,

      /* Type Checking */
      "strict": true /* Enable all strict type-checking options. */,
      // "noImplicitAny": true,                            /* Enable error reporting for expressions and declarations with an implied 'any' type. */
      // "strictNullChecks": true,                         /* When type checking, take into account 'null' and 'undefined'. */
      // "strictFunctionTypes": true,                      /* When assigning functions, check to ensure parameters and the return values are subtype-compatible. */
      // "strictBindCallApply": true,                      /* Check that the arguments for 'bind', 'call', and 'apply' methods match the original function. */
      // "strictPropertyInitialization": true,             /* Check for class properties that are declared but not set in the constructor. */
      // "noImplicitThis": true,                           /* Enable error reporting when 'this' is given the type 'any'. */
      // "useUnknownInCatchVariables": true,               /* Default catch clause variables as 'unknown' instead of 'any'. */
      // "alwaysStrict": true,                             /* Ensure 'use strict' is always emitted. */
      // "noUnusedLocals": true,                           /* Enable error reporting when local variables aren't read. */
      // "noUnusedParameters": true,                       /* Raise an error when a function parameter isn't read. */
      // "exactOptionalPropertyTypes": true,               /* Interpret optional property types as written, rather than adding 'undefined'. */
      // "noImplicitReturns": true,                        /* Enable error reporting for codepaths that do not explicitly return in a function. */
      // "noFallthroughCasesInSwitch": true,               /* Enable error reporting for fallthrough cases in switch statements. */
      // "noUncheckedIndexedAccess": true,                 /* Add 'undefined' to a type when accessed using an index. */
      // "noImplicitOverride": true,                       /* Ensure overriding members in derived classes are marked with an override modifier. */
      // "noPropertyAccessFromIndexSignature": true,       /* Enforces using indexed accessors for keys declared using an indexed type. */
      // "allowUnusedLabels": true,                        /* Disable error reporting for unused labels. */
      // "allowUnreachableCode": true,                     /* Disable error reporting for unreachable code. */

      /* Completeness */
      // "skipDefaultLibCheck": true,                      /* Skip type checking .d.ts files that are included with TypeScript. */
      "skipLibCheck": true /* Skip type checking all .d.ts files. */
    }
  }
  ```

  - 무수히 많은 주석과 간단한(?) 설명이 포함되어 있음
  - 컴파일러 옵션에 대한 상세 내용은 [타입 스크립트 공식 사이트](https://www.typescriptlang.org/ko/tsconfig)에서 확인할 수 있음

### 전역 파일 watch mode 진입

- 앞서 본 바와 같이, 특정 파일을 지정하지 않아도 `tsc —watch` 커맨드를 실행하면 프로젝트에서의 모든 타입스크립트 파일(`.ts`)은 컴파일의 대상이 되고, 관찰 모드도 함께 적용됨

## 파일 포함 및 제외하기

### index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Understanding TypeScript</title>
    <script src="app.js" defer></script>
    <script src="analytics.js" defer></script>
  </head>
  <body></body>
</html>
```

- 바닐라 자바스크립트 방식으로 `index.html` 파일에서 컴파일의 대상이되는 타입스크립트의 파일들을 불러와서 해당 파일들을 프로젝트로서 관리할 수 있음
- `tsconfig.json`의 파일을 통해 프로젝트 관리를 어떻게 할 것인가에 대한 내용을 명시함

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true
  },
  "exclude": ["node_modules"]
}
```

- `제외하기(exclude)`
  - `compilerOptions`의 중괄호 다음에 추가할 수 있는 몇 가지 옵션중에, `exclude` 옵션을 통해 타입스크립트가 컴파일하는 대상이 되는 폴더를 `제외`할 수 있음
    - 특정 파일이 될 수도 있음
    - `wildcard`(\*)를 통해 특정 파일 패턴을 적용할 수 있음
      - ex) `"*.dev.ts"` : dev.ts가 포함된 모든 파일 무시
    - `node_modules` 폴더는 `package.json`에 설치한 모든 종속성과 변경하지 않아야 할 타사 라이브러러를 가져오는 위치이므로, 지정하지 않아도 기본 설정되어 있음
      - **주의! `exclude 옵션을 아예 지정하지 않았을 경우`에만 node_modules는 제외됨**
- `포함하기(include)`
  - `exclude`와는 반대로 컴파일 과정에 포함시킬 파일을 타입스크립트에게 알려줌
- `혼합 사용(exclude + include)`
  - `include`가 필터링되므로, `exclude`를 제외한 채 `include`를 컴파일 함
- `files`
  - 개별 파일을 지정할 수 있음
  - include와 다소 비슷하지만 컴파일하고자 하는 `개별 파일`만 지정 가능
  - 비교적 규모가 작은 프로젝트에 사용하므로, 사용 빈도수가 많이 낮은 편

## 컴파일 대상 설정하기

### compilerOptions를 통해 파일 관리

- `compilerOptions`는 타입스크립트 코드가 컴파일되는 방식을 관리
- 주로 어떤 파일을 컴파일할 지, 컴파일되는 파일이 타입스크립트로 어떻게 처리되어야 하는지 설정 가능

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5"
  },
  "exclude": ["node_modules"]
}
```

- `target`
  - 어떤 자바스크립트 버전을 대상으로 코드를 컴파일 할 것인지 결정
  - 어떤 브라우저가 디컴파일된 코드를 지원하는지 정의
  - `ES5`(default value) 이외에도 다른 ES 버전의 코드를 에디터 자동 완성 기능을 통해 기입할 수 있음
    - 컴파일된 파일의 변수 선언부를 보면 `var` 키워드, `ES6`로 지정하여 컴파일하면 `let`, `const` 키워드로 선언되어 있는 것을 확인할 수 있음

## TypeScript 핵심 라이브러리 이해하기

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs"
    /*
		 ** "target": "es6" 설정과 동일
			"lib": [
			"dom",
			"es6",
			"dom.iterable",
			"scripthost"
		],
		*/
  }
}
```

- `target`
  - 어떤 자바스크립트 버전을 대상으로 코드를 컴파일 할 것인지 결정
  - 어떤 브라우저가 디컴파일된 코드를 지원하는지 정의
  - `ES5`(default value) 이외에도 다른 ES 버전의 코드를 에디터 자동 완성 기능을 통해 기입할 수 있음
    - 컴파일된 파일의 변수 선언부를 보면 `var` 키워드, `ES6`로 지정하여 컴파일하면 `let`, `const` 키워드로 선언되어 있는 것을 확인할 수 있음
- `lib`
  - dom으로 작업을 수행하는 항목들, 기본 객체, 기능, 타입스크립트 노드를 지정하게 해주는 옵션
  - 타입스크립트는 브라우저를 위해 작성하는 것이 아니므로, `document`, `var`, `const`, `let`과 같은 키워드를 알지 못하지만 기본적으로 제공하는 `lib` 옵션의 도움을 받음
    - lib 옵션이 설정되지 않으면, 기본 설정은 자바스크립트의 `target`에 따라 달라짐
      - 브라우저 작동에 필요한 사항들(`DOM API`)을 포함
    - 이 역시, 주석 처리를 해야만 타입스크립트가 기본 설정을 적용하여 작동하게 됨

## 추가 구성 및 컴파일 옵션

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5"
    // "allowJs": true,
    // "checkJs": true,
  }
}
```

- `allowJs`
  - 컴파일 시 checkJS와 함께 자바스크립트 파일에 포함할 수 있음
    - 타입스크립트가 자바스크립트로 컴파일 할 수 있게함
      - 파일이 `.ts`로 끝나지 않아도 컴파일이 가능
      - 컴파일을 수행하지 않더라도 구문을 검사하고 잠재적 에러 보고
  - 주로 타입스크립트를 사용하지 않고 일부 기능의 장점을 취하고자 할 때 유용
  - 타입스크립트 프로젝트는 `.ts` 파일에서 비롯된 `.js` 파일에 대해 컴파일을 두 번 수행할 필요가 없으므로 필요하지 않은 옵션임
    - 주로, 타입스크립트를 전혀 사용하지 않는 프로젝트 혹은 타입스크립트 파일과 바닐라 자바스크립트 파일을 함께 사용하면서 바닐라 자바스크립트 파일도 함께 검사하고 싶은 경우 사용

## 소스 맵 작업

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "sourceMap": true
  }
}
```

- `디버깅` 작업과 개발에 유용한 옵션(디버깅 단순화)
  - 복잡한 방식의 코드가 있어서 타입스크립트 코드와 디컴파일된 자바스크립트 코드를 디버깅할 때 사용
- tsc 커맨드를 사용하면 `.js.map` 파일들이 같이 생성됨
- 자바스크립트 파일과 개발자 도구간의 브릿지 역할

## rootDir 및 outDir

- 프로젝트가 커짐에 따라 파일을 제대로 정리해야 하는 필요성이 생기는 데, `rootDir`과 `outDir` 옵션으로 `src` 폴더와 `dist` 폴더로 관심사를 나누어 관리

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "outDir": "./dist",
    "rootDir": "./src",
    "removeComments": true
    // "noEmit": true
  }
}
```

- `outDir`

  - 생성된 파일이 저장되는 위치를 타입크립트 컴파일러에 알릴 수 있음
  - `tsc` 커맨드 실행 시, 자바스크립트 파일은 `dist`(모든 출력값을 보관하는 작업)에 저장됨

    - `index.html`

      ```json
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Understanding TypeScript</title>
        <script src="dist/app.js" defer></script>
        <script src="dist/analytics.js" defer></script>
      </head>
      <body>

      </body>
      </html>
      ```

      - `outDir` 경로가 바뀜에 따라, 지정한 `dist` 경로를 설정

- `rootDir`
  - 루트 디렉토리를 설정하고 타입스크립트 컴파일러가 폴더에서 보이지 않도록 지정
  - src 폴더도 확인할 뿐만 아니라, 설정한 프로젝트 구조가 `dist` 폴더에서 유지되는지도 확인
- `removeComments`
  - 타입스크립트 파일의 모든 주석이 컴파일된 자바스크립트 파일에서 제거됨
  - 파일 크기를 줄이는 데 좋은 옵션이 될 수 있음
- `noEmit`
  - 자바스크립트 파일을 생성하지 않을 경우 설정
  - 비교적 규모가 큰 프로젝트에서 시간을 절약하기 위해 파일을 생성하지 않을 때 사용
    - 컴파일러가 파일을 검사하지 않고, `잠재적 에러`를 보고하는 목적

## 컴파일 오류 시 컴파일 중단하기

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "noEmitOnError": true
  }
}
```

- 에러를 수정해야만 타입스크립트를 사용하여 파일을 컴파일 할 수 있음
  - 타입스크립트 파일에 에러가 있는 경우 자바스크립트 파일을 생성하지 않음

## Strict 컴파일

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "strict": true /* Enable all strict type-checking options. */
    // "noImplicitAny": true,                            /* Enable error reporting for expressions and declarations with an implied 'any' type. */
    // "strictNullChecks": true,                         /* When type checking, take into account 'null' and 'undefined'. */
    // "strictFunctionTypes": true,                      /* When assigning functions, check to ensure parameters and the return values are subtype-compatible. */
    // "strictBindCallApply": true,                      /* Check that the arguments for 'bind', 'call', and 'apply' methods match the original function. */
    // "strictPropertyInitialization": true,             /* Check for class properties that are declared but not set in the constructor. */
    // "noImplicitThis": true,                           /* Enable error reporting when 'this' is given the type 'any'. */
    // "useUnknownInCatchVariables": true,               /* Default catch clause variables as 'unknown' instead of 'any'. */
    // "alwaysStrict": true,                             /* Ensure 'use strict' is always emitted. */
    // "noUnusedLocals": true,                           /* Enable error reporting when local variables aren't read. */
    // "noUnusedParameters": true,                       /* Raise an error when a function parameter isn't read. */
    // "exactOptionalPropertyTypes": true,               /* Interpret optional property types as written, rather than adding 'undefined'. */
    // "noImplicitReturns": true,                        /* Enable error reporting for codepaths that do not explicitly return in a function. */
    // "noFallthroughCasesInSwitch": true,               /* Enable error reporting for fallthrough cases in switch statements. */
    // "noUncheckedIndexedAccess": true,                 /* Add 'undefined' to a type when accessed using an index. */
    // "noImplicitOverride": true,                       /* Ensure overriding members in derived classes are marked with an override modifier. */
    // "noPropertyAccessFromIndexSignature": true,       /* Enforces using indexed accessors for keys declared using an indexed type. */
    // "allowUnusedLabels": true,                        /* Disable error reporting for unused labels. */
    // "allowUnreachableCode": true,                     /* Disable error reporting for unreachable code. */
  }
}
```

- `strict` 옵션이 `true` 인 경우, 하위 항목들에 대한 `strict mode`가 활성화
- `false`인 경우, 하위 항목들을 개별 옵션으로 설정할 수 있음

### 일부 옵션 설명

- `noImplicitAny`
  - 코드에서 작업하고 있는 매개변수와 값을 명확히 할 수 있도록 도와줌
- `strictNullChecks`
  - null 값을 잠재적으로 가질 수 있는 값에 접근하고 작업하는 방식을 타입스크립트에게 엄격하게 알려줌
- `strictBindCallApply`
  - 기본적으로 호출하려는 함수가 bind, call, apply 중 무엇에 해당하는지 확인하고, 함수를 제대로 설정했는지를 확인

## 코드 품질 옵션

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "noUnusedLocals": true /* Report errors on unused locals. */,
    "noUnusedParameters": true /* Report errors on unused parameters. */,
    "noImplicitReturns": true /* Report error when not all code paths in function return a value. */
    // "noFallthroughCasesInSwitch": true,    /* Report errors for fallthrough cases in switch statement. */
  }
}
```
- `noUnusedLocals`
  - 기본적으로 사용되지 않은 변수가 있으면 타입스크립트가 에러를 표시함
- `noUnusedParameters`
  - 기본적으로 사용되지 않은 파라미터가 있으면 타입스크립트가 에러를 표시함
- `noImplicitReturns`
  - 함수가 반환하는 값이 없는 경우 타입스크립트가 에러를 표시함
    - 단, 아무것도 반환하지 않는 함수(void)가 있다면 허용
- `noFallthroughCasesInSwitch`
  - switch 문에서 break 키워드를 잊어버린 경우 타입스크립트가 에러를 표시함

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
