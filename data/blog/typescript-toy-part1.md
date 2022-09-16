---
title: Drag & Drop 토이 프로젝트 part 1
date: '2022-09-16'
tags: ['typescript']
draft: false
summary: Drag & Drop 토이 프로젝트 실습
layout: PostSimple
authors: ['default']
---

# **Drag & Drop 토이 프로젝트 part 1**

## 시작하기

### `app.css`

```tsx
* {
  box-sizing: border-box;
}

html {
  font-family: sans-serif;
}

body {
  margin: 0;
}

label,
input,
textarea {
  display: block;
  margin: 0.5rem 0;
}

label {
  font-weight: bold;
}

input,
textarea {
  font: inherit;
  padding: 0.2rem 0.4rem;
  width: 100%;
  max-width: 30rem;
  border: 1px solid #ccc;
}

input:focus,
textarea:focus {
  outline: none;
  background: #fff5f9;
}

button {
  font: inherit;
  background: #ff0062;
  border: 1px solid #ff0062;
  cursor: pointer;
  color: white;
  padding: 0.75rem 1rem;
}

button:focus {
  outline: none;
}

button:hover,
button:active {
  background: #a80041;
  border-color: #a80041;
}

.projects {
  margin: 1rem;
  border: 1px solid #ff0062;
}

.projects header {
  background: #ff0062;
  height: 3.5rem;
  display: flex;
  justify-content: center;
  align-items: center;
}

#finished-projects {
  border-color: #0044ff;
}

#finished-projects header {
  background: #0044ff;
}

.projects h2 {
  margin: 0;
  color: white;
}

.projects ul {
  list-style: none;
  margin: 0;
  padding: 1rem;
}

.projects li {
  box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.26);
  padding: 1rem;
  margin: 1rem;
}

.projects li h2 {
  color: #ff0062;
  margin: 0.5rem 0;
}

#finished-projects li h2 {
  color: #0044ff;
}

.projects li h3 {
  color: #575757;
  font-size: 1rem;
}

.project li p {
  margin: 0;
}

.droppable {
  background: #ffe3ee;
}

#finished-projects .droppable {
  background: #d6e1ff;
}

#user-input {
  margin: 1rem;
  padding: 1rem;
  border: 1px solid #ff0062;
  background: #f7f7f7;
}
```

### `Index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>ProjectManager</title>
    <link rel="stylesheet" href="app.css" />
    <script src="dist/app.js" defer></script>
  </head>
  <body>
    <template id="project-input">
      <form>
        <div class="form-control">
          <label for="title">Title</label>
          <input type="text" id="title" />
        </div>
        <div class="form-control">
          <label for="description">Description</label>
          <textarea id="description" rows="3"></textarea>
        </div>
        <div class="form-control">
          <label for="people">People</label>
          <input type="number" id="people" step="1" min="0" max="10" />
        </div>
        <button type="submit">ADD PROJECT</button>
      </form>
    </template>
    <template id="single-project">
      <li></li>
    </template>
    <template id="project-list">
      <section class="projects">
        <header>
          <h2></h2>
        </header>
        <ul></ul>
      </section>
    </template>
    <div id="app"></div>
  </body>
</html>
```

- `index.html` 파일에서 스타일링을 위해 `app.css` 파일을 가져오고, 자바스크립트 로직을 위해 컴파일된 `app.js` 파일을 로드함
- `body` 본문의 `template` 태그를 이용해 즉시 렌더링되지 않는 `html` 코드를 지정
  - 자바스크립트와 타입스크립트를 통해 접근하며, 원할 때 렌더링하고 통제할 수 있음

## DOM 요소 선택 및 OOP 렌더링

### 실행

- `npm start` 명령어를 통해 `html` 파일을 제공하는 개발 서버를 가져옴
- 탭을 새로 열어 `tsc -w` 명령어를 통해 compliation in watch mode를 시작

### `app.ts`

```tsx
class ProjectInput {
  templateElement: HTMLTemplateElement
  hostElement: HTMLDivElement
  element: HTMLFormElement

  constructor() {
    this.templateElement = document.getElementById('project-input')! as HTMLTemplateElement
    this.hostElement = document.getElementById('app')! as HTMLDivElement

    const importedNode = document.importNode(this.templateElement.content, true)
    this.element = importedNode.firstElementChild as HTMLFormElement
    this.attach()
  }

  private attach() {
    this.hostElement.insertAdjacentElement('afterbegin', this.element)
  }
}

const prjInput = new ProjectInput()
```

- `ProjectInput`에 `constructor`를 추가하여 템플릿과 렌더링되어야 하는 위치에 접근
  - `this.templateElement`, `this.hostElement` 프로퍼티를 추가하여 템플릿 내용을 렌더링하려는 엘리먼트에 대한 참조를 취득
    - 타입스크립트는 HTMLElement를 알 수 없으므로, 해당 프로퍼티와 엘리먼트를 매칭할 수 있도록 선언해줌
    - 확실히 있는 엘리먼트에 대해서 `!`를 붙여 null이 아님을 명시
    - `getElementById`는 어떤 엘리먼트가 정확히 어떤 `HTML` 엘리먼트 버전인지 모르기때문에 타입 캐스팅(`as`)을 통해 타입을 지정해야 함
  - `const importedNode = document.importNode(this.templateElement.content, true);`
    - 전역 문서 객체에 제공되는 메서드인 `document.importNode`에 템플릿 엘리먼트의 포인터를 입력함
    - 템플릿 코드 사이의 html 코드를 참조하여 두 번째 인자로 깊은 복사를 이용해 가져올 것인지 여부를넘겨줌
  - `private attach() {}`
    - 선택 사항으로 렌더링 로직을 분리하기 위해 `attach` private 메서드를 정의
    - `this.hostElement`를 가져온 후 렌더링하고 싶은 위치를 정하고 `insertAdjacentElement`를 호출
      - `HTMLElement`를 삽입하기 위해 자바스크립트 브라우저에서 제공하는 기본 메서드로서, 대상으로 삼은 엘리먼트의 처음 부분 뒤에 삽입할 수 있음
    - `this.element`는 삽입하려는 노드를 가리키는 확실한 프로퍼티를 가리킴
- `const prjInput = new ProjectInput();`
  - 클래스 아래에 새로운 상수를 생성해 `ProjectInput` 클래스를 인스턴스화하여 호출

## **DOM 요소와 상호작용**

- 다양한 양식에 접근해, 양식이 제출되었을 때 값을 읽을 수 있도록 이벤트 리스너를 `submit`으로 설정하고 사용자 입력의 유효성을 검사해야 함
- 버튼과 `form`에 접근해 `submit`, `input` 이벤트를 통해 가장 최근의 값을 가져와야 함
- 엘리먼트에 `ID`를 할당한 후, 접근하고 클래스의 프로퍼티로 저장하여 엘리먼트를 참조

### `app.ts`

```tsx
class ProjectInput {
  templateElement: HTMLTemplateElement
  hostElement: HTMLDivElement
  element: HTMLFormElement
  titleInputElement: HTMLInputElement
  descriptionInputElement: HTMLInputElement
  peopleInputElement: HTMLInputElement

  constructor() {
    this.templateElement = document.getElementById('project-input')! as HTMLTemplateElement
    this.hostElement = document.getElementById('app')! as HTMLDivElement

    const importedNode = document.importNode(this.templateElement.content, true)

    this.element = importedNode.firstElementChild as HTMLFormElement
    this.element.id = 'user-input'

    this.titleInputElement = this.element.querySelector('#title') as HTMLInputElement
    this.descriptionInputElement = this.element.querySelector('#description') as HTMLInputElement
    this.peopleInputElement = this.element.querySelector('#people') as HTMLInputElement

    this.configure()
    this.attach()
  }

  private submitHandler(event: Event) {
    event.preventDefault()
  }

  private configure() {
    this.element.addEventListener('submit', this.submitHandler.bind(this))
  }

  private attach() {
    this.hostElement.insertAdjacentElement('afterbegin', this.element)
  }
}
const prjInput = new ProjectInput()
```

- `titleInputElement`, `descriptionInputElement`, `peopleInputElement`의 필드를 추가하고 생성자 메서드 안에 `HTMLFormElement` 하위에 있는 `HTMLInputElement`를 `querySelector`를 입력해 가져옴
- `private configure(){}`
  - 이벤트 리스너를 설정해 `this.element`와 `addEventListener`를 입력
  - `this` 키워드가 `submitHandler`에서 클래스를 가리키지 않았으므로 `bind` 메서드를 통해 `this`를 전달
- `private submitHandler(){}`
  - 이벤트 객체를 받는 메서드를 통해 이벤트 간 리스너를 바인딩해야 함
  - 입력 값에 접근하고 유효성 검사를 위해 `event.preventDefault`를 호출하여 기본 `form`의 `submit Event`를 방지

## Autobind 데코레이터 생성 및 사용

- 앱 여러 곳에서 `bind` 호출을 해야하는 경우, 데코레이터를 이용하면 작업이 훨씬 쉬워질 수 있음
- `bind`를 제거하고, `autobind decorator`를 추가

### `tsconfig.json`

```tsx
{
  "compilerOptions": {
    /* Basic Options */
    "target": "es6",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */
    "module": "commonjs",                     /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
    "lib": [
      "dom",
      "es6",
      "dom.iterable",
      "scripthost"
    ],                                         /* Specify library files to be included in the compilation. */
    "sourceMap": true /* Generates corresponding '.map' file. */,
    "outDir": "./dist" /* Redirect output structure to the directory. */,
    "rootDir": "./src" /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */,
    "removeComments": true /* Do not emit comments to output. */,
    "noEmitOnError": true,

    /* Strict Type-Checking Options */
    "strict": true,                           /* Enable all strict type-checking options. */

    /* Additional Checks */
    "noUnusedLocals": true /* Report errors on unused locals. */,
    "noUnusedParameters": true /* Report errors on unused parameters. */,
    "noImplicitReturns": true /* Report error when not all code paths in function return a value. */,
    "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */,

		/* Experimental Options */
    "experimentalDecorators": true /* Enables experimental support for ES7 decorators. */
  },
  "exclude": [
    "node_modules" // would be the default
  ]
}
```

- `decorator`를 사용하기 위해서 `tsconfig.json` 파일에서 `Experimetal Options`인 `"experimentalDecorators"` 키 값을 `true`로 설정해야 함

### `app.ts`

- `autobind decorator`

  ```tsx
  // autobind decorator
  function autobind(_: any, _2: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value
    const adjDescriptor: PropertyDescriptor = {
      configurable: true,
      get() {
        const boundFn = originalMethod.bind(this)
        return boundFn
      },
    }
    return adjDescriptor
  }
  ```

  - 데코레이터는 함수이므로 함수 선언 또는 함수 표현식으로 정의할 수 있음
  - 함수 선언을 이용해 3개의 인자를 전달 받음
    - `target`과 바인딩할 `methodName`, `descriptor`를 전달 받음
  - `descriptor.value`
    - 원래 정의했던 메서드를 정의
  - `const adjDescriptor:PropertyDescriptor = {}`
    - 조정된 디스크립터를 생성하고 `configurable`을 `true`로 설정해서 언제든 수정할 수 있게 설정
    - `getter`를 통해 함수에 접근할 때 실행되게 `bound` 함수를 설정
    - 추출한 `originalMethod`를 이용해 `bind(this)`를 호출하고 `bound` 함수를 반환
  - `return adjDescriptor;`
    - 해당 메서드 데코레이터에서 조정된 디스크립터를 반환
  - 실제로 쓰이지 않는 인수 값(target, methodName)은 두 가지 방법으로 해결할 수 있음
    1. `tsconfig`에서 `restrict` 해당하는 옵션 값(`noUnusedParameters: false`)을 조정
    2. 밑줄(`_`)을 사용해 타입스크립트와 자바스크립트에 이 값들을 당장 사용하지 않을 것이라고 암시를 줄 수 있음

- `ProjectInput Class`

  ```tsx
  class ProjectInput {
    templateElement: HTMLTemplateElement
    hostElement: HTMLDivElement
    element: HTMLFormElement
    titleInputElement: HTMLInputElement
    descriptionInputElement: HTMLInputElement
    peopleInputElement: HTMLInputElement

    constructor() {
      this.templateElement = document.getElementById('project-input')! as HTMLTemplateElement
      this.hostElement = document.getElementById('app')! as HTMLDivElement

      const importedNode = document.importNode(this.templateElement.content, true)

      this.element = importedNode.firstElementChild as HTMLFormElement
      this.element.id = 'user-input'

      this.titleInputElement = this.element.querySelector('#title') as HTMLInputElement
      this.descriptionInputElement = this.element.querySelector('#description') as HTMLInputElement
      this.peopleInputElement = this.element.querySelector('#people') as HTMLInputElement

      this.configure()
      this.attach()
    }

    @autobind
    private submitHandler(event: Event) {
      event.preventDefault()
    }

    private configure() {
      this.element.addEventListener('submit', this.submitHandler)
    }

    private attach() {
      this.hostElement.insertAdjacentElement('afterbegin', this.element)
    }
  }
  const prjInput = new ProjectInput()
  ```

  - `submitHandler`에 `@autobind`를 추가

> 수동으로 `bind`를 호출할 필요없이, 데코레이터를 통해 많은 메서드에 적용할 수 있고 시간 절약과 잠재적인 오류를 방지할 수 있음

## 사용자 입력 가져오기

- 필수 입력 값을 취합해서 `council`에 제출할 수 있음
- 사용자의 모든 입력을 취합해서, 자유 튜플값으로써의 반환 타입을 규정

### `app.ts`

```tsx
class ProjectInput {
  // ...
  private gatherUserInput(): [string, string, number] | void {
    const enteredTitle = this.titleInputElement.value
    const enteredDescription = this.descriptionInputElement.value
    const enteredPeople = this.peopleInputElement.value

    if (
      enteredTitle.trim().length === 0 ||
      enteredDescription.trim().length === 0 ||
      enteredPeople.trim().length === 0
    ) {
      alert('Invalid input, please try again!')
      return
    } else {
      return [enteredTitle, enteredDescription, +enteredPeople]
    }
  }

  private clearInputs() {
    this.titleInputElement.value = ''
    this.descriptionInputElement.value = ''
    this.peopleInputElement.value = ''
  }

  @autobind
  private submitHandler(event: Event) {
    event.preventDefault()
    const userInput = this.gatherUserInput()
    if (Array.isArray(userInput)) {
      const [title, desc, people] = userInput
      console.log(title, desc, people)
    }
  }
}
```

- `private gatherUserInput(): [string, string, number] | void{}`
  - 튜플은 세개의 구성요소를 가짐
  - 첫 번째와 두 번째 구성요소는 열이고 세번째 구성요소는 숫자를 가져옴
  - 값의 타이틀 입력 요소에서 모든 입력 항목을 얻음(`title`, `description`, `people`)
    - 취득한 `title`, `description`, `people` 값의 여백을 제거하여 각각의 값이 길이가 0일 경우에 유효하지 않은 `input` 값임을 알려줌
    - 값이 유효하다면 튜플을 반환함
      - `enteredPeople` 값의 경우 `+`를 이용해 `number type`으로 변환
- `if (Array.isArray(userInput)){}`
  - 사용자 입력 타입이 튜플과 등가인지 확인할 방법이 없음
    - 튜플은 자바스크립트가 아니므로 튜플인지 여부를 확인하려면 결국 튜플은 배열임을 기억해야함
    - 타입스크립트에서는 특별하지만 자바스크립트 영역에 있으면 배열임
  - `Array.isArray(userInput)`
    - 튜플은 배열이므로 `destructuring`으로 `userInput`의 값을 추출
- `private clearInputs() {}`
  - 모든 입력값을 비우는 함수를 정의

## 재사용 가능한 검증 기능 생성

### 검증 객체 구조 정의

```tsx
// validation
interface Validatable {
  value: string | number
  required?: boolean
  minLength?: number
  maxLength?: number
  min?: number
  max?: number
}
```

- 검증 가능한 객체 구조를 정의
  - 검증 함수가 대상을 인지하고 속성 등을 정확하게 추출하기 위해서 인터페이스를 정의
- `required`
  - 필수 명제로는 `boolean` value를 취함
- `minLength, maxLength`
  - `input value`(문자열)의 길이
- `min, max`
  - 수치 값이 특정 숫자 이싱인지, 특정 최대값 이하인지 확인
- 수치외에는 모두 선택 사항이어야함
  - ?(`Optional Operator`)를 사용

### 검증 함수 정의

```tsx
function validate(validatableInput: Validatable) {
  let isValid = true
  if (validatableInput.required) {
    isValid = isValid && validatableInput.value.toString().trim().length !== 0
  }
  if (validatableInput.minLength != null && typeof validatableInput.value === 'string') {
    isValid = isValid && validatableInput.value.length >= validatableInput.minLength
  }

  if (validatableInput.maxLength != null && typeof validatableInput.value === 'string') {
    isValid = isValid && validatableInput.value.length <= validatableInput.maxLength
  }

  if (validatableInput.min != null && typeof validatableInput.value === 'number') {
    isValid = isValid && validatableInput.value > validatableInput.min
  }

  if (validatableInput.max != null && typeof validatableInput.value === 'number') {
    isValid = isValid && validatableInput.value < validatableInput.max
  }
  return isValid
}
```

- 위에서 정의한 인터페이스의 구조를 가지는 `validatable`한 `validatableInput`를 인자로 받음
- 초기에 참(`true`) 값인 유효한 변수를 생성
  - 디폴트는 `true` 값이지만 검증 중 하나라도 실패하면 `false`로 설정
- `필수 값(required) 검증`

  ```
  if (validatableInput.required) {
    isValid = isValid && validatableInput.value.toString().trim().length !== 0;
  }
  ```

  - `input value`가 공란임을 검증하는 조건식
  - `isValid`는 초기값으로 `true`이므로, `validatableInput.value` 값을 검증
    - 문자열인지 명확하지 않으므로, `toString()` 메서드를 통해 문자열로 변환하고 `trim()` 메서드와 문자열의 길이인 `length`가 0이 아님을 검증

- `minLength`, `maxLength` 검증

  ```tsx
  if (validatableInput.minLength != null && typeof validatableInput.value === 'string') {
    isValid = isValid && validatableInput.value.length >= validatableInput.minLength
  }

  if (validatableInput.maxLength != null && typeof validatableInput.value === 'string') {
    isValid = isValid && validatableInput.value.length <= validatableInput.maxLength
  }
  ```

  - `input value`의 최소, 최대 길이가 `null`이 아니고, `string`인 `type`임을 검증하는 조건식
  - 특정 최소 혹은 최대 길이가 `null`이나 `undefined`가 같지 않음을 검증
    - `validatableInput.minLength != null`, `validatableInput.maxLength != null`
  - 특정 최소 길이와, 최대 길이가 문자열의 길이 조건에 부합해야 함
    - 타입 가드를 추가해서 문자열과 등가인지 확인
    - `validatableInput.value`가 최소 길이보다 작거나, 최대 길이보다 큰지 확인
      - `isValid = isValid && validatableInput.value.length <= validatableInput.minLength;`
      - `isValid = isValid && validatableInput.value.length <= validatableInput.maxLength;`

- `min`, `max` 값 검증

  ```tsx
  if (validatableInput.min != null && typeof validatableInput.value === 'number') {
    isValid = isValid && validatableInput.value > validatableInput.min
  }

  if (validatableInput.max != null && typeof validatableInput.value === 'number') {
    isValid = isValid && validatableInput.value < validatableInput.max
  }
  ```

  - 위의 검증 로직과 유사하게 최소, 최대 값이 `number` `type`이며, `null` 또는 `undefined`를 가지지 말아야 함

### 검증 가능 객체 구성

```tsx
class ProjectInput {
  // ...
  private gatherUserInput(): [string, string, number] | void {
    const enteredTitle = this.titleInputElement.value
    const enteredDescription = this.descriptionInputElement.value
    const enteredPeople = this.peopleInputElement.value

    const titleValidatable: Validatable = {
      value: enteredTitle,
      required: true,
    }
    const descriptionValidatable: Validatable = {
      value: enteredDescription,
      required: true,
      minLength: 5,
    }
    const peopleValidatable: Validatable = {
      value: +enteredPeople,
      required: true,
      min: 1,
      max: 5,
    }

    if (
      !validate(titleValidatable) ||
      !validate(descriptionValidatable) ||
      !validate(peopleValidatable)
    ) {
      alert('Invalid input, please try again!')
      return
    } else {
      return [enteredTitle, enteredDescription, +enteredPeople]
    }
  }
}
```

- `titleValidatable`, `descriptionValidatable`, `peopleValidatable` 를 선언하고 Validatable 인터페이스의 구조를 가지는 값을 할당

```tsx
if (
  !validate(titleValidatable) ||
  !validate(descriptionValidatable) ||
  !validate(peopleValidatable)
) {
  alert("Invalid input, please try again!");
  return;
} else {
  return [enteredTitle, enteredDescription, +enteredPeople];
}
```

- `validate` 함수에 검증하고자 하는 값을 매개변수로 넘겨줌
- validate 함수 조건문 중 하나라도 `false`라면 `Invalid input, please try again!` 경고창을 노출

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
