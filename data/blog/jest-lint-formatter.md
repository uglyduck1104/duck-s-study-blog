---
title: 테스팅 라이브러리가 포함된 ESLint와 Prettier
date: '2022-08-16'
tags: ['Jest']
draft: false
summary: Javascript에서 흔히 사용되는 린터로서 정적 텍스트를 분석하고 린터 규칙을 위반하는 구문을 마킹하는 도구
layout: PostSimple
authors: ['default']
---

# 테스팅 라이브러리가 포함된 ESLint와 Prettier

## ESLint와 Prettier

### ESLint

- Javascript에서 흔히 사용되는 린터로서 정적 텍스트를 분석하고 린터 규칙을 위반하는 구문을 마킹하는 도구
- 코드가 어떻게 실행되는지에 대해서는 분석이 이루어지지 않음
- 코드의 `스타일`을 일관적으로 유지하는 데 아주 유용하며 협업시 많은 도움이 됨
- 코드의 `오류`를 잡아 낼 수 있음

### Prettier

- 포맷터로 코드를 자동으로 포맷팅해줌
- 들여쓰기와 중괄호내의 공백 일관성을 유지해 줌

### ESLint 플러그인

- ESLint의 규칙을 확장하여 jest-dom도 지원하며 선호하는 단언 메서드를 사용할 수 있게 해줌

## 테스팅 라이브러리와 Jest-DOM을 위한 ESLint

### 설치

```bash
$ npm install eslint-plugin-testing-library eslint-plugin-jest-dom
```

### 구성( `.eslintrc.json`)

```json
{
  "plugins": ["testing-library", "jest-dom"],
  "extends": [
    "react-app",
    "react-app/jest",
    "plugin:testing-library/react",
    "plugin:jest-dom/recommended"
  ]
}
```

- `package.json`내의 `eslintConfig` 구성을 삭제한 후, 별로도 `.eslintrc.json`파일을 프로젝트의 rootPath에 생성

1. 사용하고자 하는 `plugins`를 배열에 담아 추가
2. 추가한 플러그인의 규칙을 `extends` 배열에 추가

- `package.json`에서 삭제한 규칙을 추가
  - `react-app`, `react-app/jest`
- `testing-library` 플러그인과, `jest-dom` 플러그인의 규칙을 추가
- 이외의 추가적인 규칙은 아래 링크에서 확인 가능함
  - [https://github.com/testing-library/eslint-plugin-testing-library](https://github.com/testing-library/eslint-plugin-testing-library)
  - [https://github.com/testing-library/eslint-plugin-jest-dom](https://github.com/testing-library/eslint-plugin-testing-library)

## VSCode에서 ESLint 구성

- VSCode내 확장 프로그램에서 Prettier을 검색한 후 설치하고, MAC OS 기준으로 `Command + Shift + p`의 조합으로 작업 영역 설정을 연 후, 아래와 같이 설정

```json
{
  "eslint.options": {
    "configFile": ".eslintrc.json"
  },
  "eslint-validate": ["javascript", "javascriptreact"],
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
	"editor.defaultFormatter": "esbenp.prettier-vscode",
	"editor.formatOnSave": true
}
```

> **Referenced**

- [Testing React with Jest and React Testing Library (RTL)](https://www.udemy.com/course/understanding-typescript/)
