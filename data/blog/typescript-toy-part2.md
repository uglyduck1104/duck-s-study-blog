---
title: Drag & Drop 토이 프로젝트 part 2
date: '2022-10-16'
tags: ['typescript']
draft: false
summary: Drag & Drop 토이 프로젝트 실습
layout: PostSimple
authors: ['default']
---

# Drag & Drop 토이 프로젝트 part 2

## 렌더링 프로젝트 목록

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
    <script src="dist/app.js" defer></script>
  </head>
  <body>
    <!-- ... -->
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

- Project의 List를 출력하는 Element를 출력하는 Class를 생성
  - `<template id="project-list">`

### `app.ts`

```tsx
class ProjectList {
  templateElement: HTMLTemplateElement
  hostElement: HTMLDivElement
  element: HTMLElement

  constructor(private type: 'active' | 'finished') {
    this.templateElement = document.getElementById('project-list')! as HTMLTemplateElement
    this.hostElement = document.getElementById('app')! as HTMLDivElement

    const importedNode = document.importNode(this.templateElement.content, true)
    this.element = importedNode.firstElementChild as HTMLElement
    this.element.id = `${this.type}-projects`
    this.attach()
    this.renderContent()
  }

  private renderContent() {
    const listId = `${this.type}-projects-list`
    this.element.querySelector('ul')!.id = listId
    this.element.querySelector('h2')!.textContent = this.type.toUpperCase() + ' PROJECTS'
  }

  private attach() {
    this.hostElement.insertAdjacentElement('beforeend', this.element)
  }
}
```

- `templateElement`와 `hostElement`, `element`를 추가
  - `hostElement`는 프로젝트의 목록을 생성하는 대상
  - `element`는 별도의 섹션 구성요소가 따로 없기 때문에 일반 `HTMLElement`로 유형을 지정
- `constructor` 생성자를 정의하여, 대상이 되는 `Element`의 속성을 할당
  - `HTMLElement` 요소에 첫번째 구성 요소를 저장하고, `id`를 동적으로 부여하여 프로젝트 목록이 하나 이상임을 명시함
  - 하나는 `활성` 프로젝트용, 하나는 `비활성` 프로젝트용
- `constructor(private type: "active" | "finished") {}`
  - 매개변수 앞에 `private` 또는 `public`의 접근자를 추가해 자동으로 동일한 이름의 속성을 생성하고 동일 명칭의 속성 내 논증에 전달된 값을 전달
  - 타입 매개변수의 유형은 문자열 형식을 가져서 활성화 프로젝트 아니면 종료된 프로젝트를 가짐
  - 템플릿에서 얻은 섹션에 id를 추가
- `private attach() {}`
  - DOM에 삽입될 위치와 대상이 되는 `element`를 지정
- `private renderContent() {}`
  - `type-projects-list`의 아이디를 가지는 요소를 추가하고, `ul`요소에는 `listId`를 할당하고, `h2` 태그의 `textContent`를 설정
  - `this.type`은 `active` 혹은 `finished`이므로 `toUppercase` 메서드를 이용해 대문자로 변환하고 `+ PROJECTS`와 연산

## 싱글톤으로 애플리케이션 상태 관리하기

### `app.ts - ProjectState Class`

```tsx
// Project State Management

class ProjectState {
  private listeners: any[] = []
  private projects: any[] = []
  private static instance: ProjectState

  private constructor() {}

  static getInstance() {
    if (this.instance) {
      return this.instance
    }
    this.instance = new ProjectState()
    return this.instance
  }

  addListener(listenerFn: Function) {
    this.listeners.push(listenerFn)
  }

  addProject(title: string, description: string, numOfPeople: number) {
    const newProject = {
      id: Math.random().toString(),
      title: title,
      description: description,
      people: numOfPeople,
    }
    this.projects.push(newProject)
    for (const listenerFn of this.listeners) {
      listenerFn(this.projects.slice())
    }
  }
}

const projectState = ProjectState.getInstance()
```

- 앱 상태를 관리하는 클래스를 생성하여 앱 관리 대상이 되는 상태를 관리하고, 앱의 관련된 다른 부분의 리스너를 설정
- 다수의 프로젝트를 가지는 `projects`를 `any` 타입의 배열과 `private` 접근 제어자로 설정
- `addProject(){}`
  - `public`의 `addProject` 메서드를 정의하고 문자열인 `title`과 `description`, `people`의 수를 추가
  - 새로운 프로젝트를 push 메서드를 통해 추가
- `private static instance: ProjectState;`
  - 싱글톤 클래스임을 확실히 하고자 `private` 상수를 생성하고 `static intance`임을 명시함
- `static getInstance() {}`
  - `getInstance` 메소드를 `static`으로 추가하고 인스턴스 여부에 따라 새로운 인스턴스를 반환할지 기존 인스턴스를 반환할지 결정
- `const projectState = ProjectState.getInstance();`
  - 동일한 객체로 항상 작업할 수 있도록 `ProjectState`의 `getInstance` 메서드를 호출할 수 있음
- `private listeners: any[] = [];`
  - 구독 패턴을 설정하기 위해 리스너 목록을 관리
    - 변경사항이 있을 시, 함수 목록이 호출됨
  - `addListener(listenerFn: Function) {}`
    - 리스너 함수를 리스너 배열에 push함
- `for (const listenerFn of this.listeners) {}`
  - 새로운 프로젝트를 추가할 때 뭔가 변화가 있을 때마다, 모든 리스너 함수를 호출
  - 리스너 함수에 `this.project`를 전달하고 `slice` 메서드를 호출해서 원본 배열이 아닌 복사 배열을 반환해 원본 배열이 변경되지 않도록 해야 함
    - 배열과 객체는 자바스크립트에서 참조값이기 때문임

### `app.ts - ProjectInput Class`

```tsx
// ProjectInput Class
class ProjectInput {
  templateElement: HTMLTemplateElement
  hostElement: HTMLDivElement
  element: HTMLFormElement
  titleInputElement: HTMLInputElement
  descriptionInputElement: HTMLInputElement
  peopleInputElement: HTMLInputElement

  // ...

  @autobind
  private submitHandler(event: Event) {
    event.preventDefault()
    const userInput = this.gatherUserInput()
    if (Array.isArray(userInput)) {
      const [title, desc, people] = userInput
      projectState.addProject(title, desc, people)
      this.clearInputs()
    }
  }

  private configure() {
    this.element.addEventListener('submit', this.submitHandler)
  }

  private attach() {
    this.hostElement.insertAdjacentElement('afterbegin', this.element)
  }
}
```

- `projectState.addProject(title, desc, people);`
  - 싱글톤 생성자로 생성한 `ProjectState` 클래스의 `addProject`를 호출하고 `gatherUserInput`에서 얻은 `title`, `desc`, `people`로 전달

### `app.ts - ProjectList Class`

```tsx
// ProjectList Class
class ProjectList {
  templateElement: HTMLTemplateElement
  hostElement: HTMLDivElement
  element: HTMLElement
  assignedProjects: any[]

  constructor(private type: 'active' | 'finished') {
    this.templateElement = document.getElementById('project-list')! as HTMLTemplateElement
    this.hostElement = document.getElementById('app')! as HTMLDivElement
    this.assignedProjects = []

    const importedNode = document.importNode(this.templateElement.content, true)
    this.element = importedNode.firstElementChild as HTMLElement
    this.element.id = `${this.type}-projects`

    projectState.addListener((projects: any[]) => {
      this.assignedProjects = projects
      this.renderProjects()
    })
    this.attach()
    this.renderContent()
  }

  private renderProjects() {
    const listEl = document.getElementById(`${this.type}-projects-list`)! as HTMLUListElement
    for (const prjItem of this.assignedProjects) {
      const listItem = document.createElement('li')
      listItem.textContent = prjItem.title
      listEl?.appendChild(listItem)
    }
  }

  private renderContent() {
    const listId = `${this.type}-projects-list`
    this.element.querySelector('ul')!.id = listId
    this.element.querySelector('h2')!.textContent = this.type.toUpperCase() + ' PROJECTS'
  }

  private attach() {
    this.hostElement.insertAdjacentElement('beforeend', this.element)
  }
}
```

- `projectState.addListener((projects: any[]) => {}`
  - 프로젝트 목록에 변화가 생기면 호출하는 대상으로서, 함수를 `addListener` 화살표 함수에 전달해야 함
- `private renderProjects() {}`
  - 해당 목록에 해당하는 모든 프로젝트를 렌더링하는 메서드 정의
  - `assignedProjects`의 모든 항목을 순회하여 목록에 추가
  - 특정 타입에 해당하는 `projects-list` 요소 하위에 `assignedProjects`의 `title` property를 할당

## 더 많은 클래스 및 사용자 정의 타입

- 할당된 프로젝트를 위해 any type으로 정의된 객체, 리스너 함수들을 구체화

```tsx
// Project Type
enum ProjectStatus {
  Active,
  finished,
}
class Project {
  constructor(
    public id: string,
    public title: string,
    public description: string,
    public people: number,
    public status: ProjectStatus
  ) {}
}
```

- `Project` 클래스를 사용해 항상 동일한 구조를 갖는 객체를 생성
- `ProjectStatus`는 `enum` type으로 정의해 옵션이 정확히 두개 있음을 명시함

### `any` 타입 `Project` 객체로 정의하기

```jsx
private projects: Project[] = [];
```

- `ProjectState`의 `projects`의 `type`을 `Project` 배열임을 명시

```tsx
const newProject = new Project(
  Math.random().toString(),
  title,
  description,
  numOfPeople,
  ProjectStatus.Active
)
```

- 객체 리터럴로 정의했던 `newProject` 변수를 `Project` class의 인스턴스로 할당
  - `enum` type으로 선언했던 `ProjectStatus`의 `Active`를 명시

```tsx
class ProjectList {
  templateElement: HTMLTemplateElement
  hostElement: HTMLDivElement
  element: HTMLElement
  assignedProjects: Project[]
  //...
}
```

- 프로젝트 목록 클래스의 `assignedProjects`의 `any` 타입 배열을 `Project` 배열로 수정

```tsx
projectState.addListener((projects: Project[]) => {
  this.assignedProjects = projects
  this.renderProjects()
})
```

- `addListener` 함수를 사용하는 곳에 `any` 타입의 배열이 아닌 `Project` 배열임을 명시

### Custom Listener type 추가

```tsx
type Listener = (items: Project[]) => void
```

- 프로젝트의 배열을 매개변수로 받는 `Listener` 함수 타입을 정의하고 반환 타입을 `void`로 설정
  - 리스너로 작업하는 부분에서는 반환 타입이 필요없기 때문에 `void`로 반환

```tsx
addListener(listenerFn: Listener) {
  this.listeners.push(listenerFn);
}
```

- 리스너를 추가하는 메서드의 매개변수를 위에서 정의한 `Listener` 함수로 명시

## 열거형으로 프로젝트 필터링하기

- `filter` method를 사용해 특정 프로젝트에 추가할 때 중복되는 객체들을 필터링

```tsx
class ProjectList {
  templateElement: HTMLTemplateElement
  hostElement: HTMLDivElement
  element: HTMLElement
  assignedProjects: Project[]

  constructor(private type: 'active' | 'finished') {
    this.templateElement = document.getElementById('project-list')! as HTMLTemplateElement
    this.hostElement = document.getElementById('app')! as HTMLDivElement

    this.assignedProjects = []

    const importedNode = document.importNode(this.templateElement.content, true)
    this.element = importedNode.firstElementChild as HTMLElement
    this.element.id = `${this.type}-projects`

    projectState.addListener((projects: Project[]) => {
      const relevantProjects = projects.filter((prj) => {
        if (this.type === 'active') {
          return prj.status === ProjectStatus.Active
        }
        return prj.status === ProjectStatus.Finished
      })
      this.assignedProjects = relevantProjects
      this.renderProjects()
    })
    this.attach()
    this.renderContent()
  }

  private renderProjects() {
    const listEl = document.getElementById(`${this.type}-projects-list`)! as HTMLUListElement

    listEl.innerHTML = ''
    for (const prjItem of this.assignedProjects) {
      const listItem = document.createElement('li')
      listItem.textContent = prjItem.title
      listEl.appendChild(listItem)
    }
  }

  private renderContent() {
    //...
  }

  private attach() {
    //...
  }
}
```

- `const relevantProjects = projects.filter((prj) => {}`
  - `enum` type으로 선언한 `ProjectStatus`의 상태 값이 `active`인지 `finished`인지에 따라 해당하는 `project` 배열을 `assignedProjects`에 할당
- `private renderProjects() {}`
  - `renderProject` 메서드에 프로젝트 목록이 렌더링 될때마다 모든 목록을 없앤 후에 재생성하여 중복을 방지

## 상속 & 제네릭 추가하기

### 메인이 되는 컴포넌트 클래스 구성

```tsx
abstract class Component<T extends HTMLElement, U extends HTMLElement> {
  templateElement: HTMLTemplateElement
  hostElement: T
  element: U

  constructor(
    templateId: string,
    hostElementId: string,
    insertAtStart: boolean,
    newElementId?: string
  ) {
    this.templateElement = document.getElementById(templateId)! as HTMLTemplateElement
    this.hostElement = document.getElementById(hostElementId)! as T

    const importedNode = document.importNode(this.templateElement.content, true)
    this.element = importedNode.firstElementChild as U
    if (newElementId) {
      this.element.id = newElementId
    }

    this.attach(insertAtStart)
  }

  private attach(insertAtBeginning: boolean) {
    this.hostElement.insertAdjacentElement(
      insertAtBeginning ? 'afterbegin' : 'beforeend',
      this.element
    )
  }

  abstract configure(): void
  abstract renderContent(): void
}
```

- `DOM`에 렌더링하는 모든 클래스의 공통 기능을 관리하는 `Component` 클래스를 생성
- 템플릿 요소와, 호스트 요소, 요소 3개의 type은 HTML 템플릿으로 구성됨
- `hostElement`, `element`
  - 대상이 되는 요소는 크게 `HTMLElement`를 상속받는 제네릭 타입(`T`, `U`)으로 구성
  - HTML 요소만 있다고 제한해버리면 추가 정보를 잃게되므로 구체적인 정보를 저장하기 위해 제네릭 클래스를 만들어 구현 타입을 정함
- `constructor`
  - 문자열 타입의 효스트 요소 템플릿의 `ID`를 알면 어디에 해당 요소를 렌더링할지 알 수 있게 생성자를 추가
  - `newElemenId`를 더해 새롭게 렌더링된 요소에 `Id`를 할당하여, 선택적 요소를 명시하는 물음표(?)를 붙여줌
  - `document.getElementById(HTML Selector) as Generic<T>`
    - `templateId`, `hostElementId`를 가리키고 제네릭 타입의 `T`를 가리킴
  - `document.importNode()`
    - 노드를 임포트해서 제네릭 타입의 `U`를 가지고 있는 요소를 더함
  - `if (newElementId) {…}`
    - `newElementId` 요소는 선택적이므로 `newElementId`에 해당될때만 요소의 `Id`를 할당
  - `this.attach(insertAtStart);`
    - 컴포넌트 클래스 생성자 끝에 `attach` 메소드를 추가하고 `boolean` 값을 인수로 전달
  - `private attach(insertAtBeginning: boolean) {…}`
    - `insertAtBeginning`의 `boolean` 매개변수로 전달된 값에 따라서 호스트 요소에 더하고 싶은 위치를 정함
- `abstract class Component<T extends HTMLElement, U extends HTMLElement> {}`
  - 추상 클래스 키워드를 통해 직접 인스턴트화가 이뤄지지 않고 언젠 상속을 위해 사용됨을 정의
- `abstract configure(): void; abstract renderContent(): void;`
  - `configure` 메소드와 `renderContent` 메소드 두 가지를 추가하고 실제 구현되지 않는다는 뜻을 가짐
  - `abstract` 추상 키워들르 통해 `Component` 클래스를 상속받는 모든 클래스가 해당 메소드를 구현해야 함을 명시

### 상속받는 클래스의 제네릭 타입 구성

```tsx
// ProjectList Class
class ProjectList extends Component<HTMLDivElement, HTMLElement> {
  /* super() */
}

// ProjectInput Class
class ProjectInput extends Component<HTMLDivElement, HTMLFormElement> {
  /*...*/
}
```

- `Component`를 상속받는 구현체(클래스)의 제네릭 타입에 끼워넣을 구체적인 값을 정함
- `super(someArgs)`
  - 생성자의 베이스 클래스(`Component` 클래스)를 불러오기 위해 `super` 키워드를 추가
  - `super('project-list', 'app', false, ${type}-projects);`
    `super('project-input', 'app', true, 'user-input');`
    - 첫번째와 두번째 인수는 전과 동일
    - 세번째 매개변수에는 호스트 요소의 위치에 대한 `boolean`값을 전달
    - 네번째 매개변수에는 새로운 요소에 대한 `Id` 값을 전달
- `configure(){}; renderContent(){};`
  - `configure` 메소드와 `renderContent` 메소드를 `Component` 클래스에 만족하게 기존의 `private` 키워드를 삭제하거나 재구성하여 구현

### 프로젝트의 상태를 관리하기 위한 베이스 상태 클래스 생성

```tsx
// Project State Management
type Listener<T> = (items: T[]) => void

class State<T> {
  protected listeners: Listener<T>[] = []

  addListener(listenerFn: Listener<T>) {
    this.listeners.push(listenerFn)
  }
}
```

- 리스너 부분의 `addListener` 메소드의 경우 베이스 클래스(`State`)를 구성하여 `listener` 배열에 제네릭 타입을 추가하여 외부의 상태와 구별
- 상속 클래스의 접근을 허용하기 위해 `protected` 키워드를 사용

## 클래스로 프로젝트 항목 렌더링

### 프로젝트 항목 클래스 생성

```tsx
class ProjectItem extends Component<HTMLUListElement, HTMLLIElement> {
  private project: Project

  constructor(hostId: string, project: Project) {
    super('single-project', hostId, false, project.id)
    this.project = project

    this.configure()
    this.renderContent()
  }

  configure() {}
  renderContent() {
    this.element.querySelector('h2')!.textContent = this.project.title
    this.element.querySelector('h3')!.textContent = this.project.people.toString()
    this.element.querySelector('p')!.textContent = this.project.description
  }
}
```

- 컴포넌트 클래스를 상속받는 `ProjectItem` 클래스를 생성하여 단일 프로젝트 항목을 렌더링
- 베이스 클래스(`Component`)의 제네릭 타입으로 첫번째로 보낼 타입은 호스트 요소, 두번째로 보낼 타입은 렌더링하고자 하는 요소를 전달
  - `ULListElement`, `LIElement`를 전달해 구현 목록 항목 엘리먼트를 생성
- 컴포넌트를 상속받기 때문에 생성자에 `super`를 불러와야 함

  - 렌더링될 요소 Id가 어디에 있는지 알려주어야 함
  - `index.html`

    ```jsx
    <template id="single-project">
      <li>
        <h2></h2>
        <h3></h3>
        <p></p>
      </li>
    </template>
    ```

  - `this.project = project;`
    - 해당 프로젝트 항목에 속하는 프로젝트를 저장하기 위해 프로젝트 클래스를 프로젝트 항목 클래스에 저장

- `configure() {}; renderContent() {…}`
  - 렌더링되어야할 프로젝트 항목을 구성하기 위한 `li element`안의 내부 요소에 제목(`title`), 사람수(`people`), 내용(`description`)을 주입하기 위한 코드를 작성

### 프로젝트 목록 클래스 수정

```tsx
// ProjectList Class
class ProjectList extends Component<HTMLDivElement, HTMLElement> {
  assignedProjects: Project[]
  // ...
  private renderProjects() {
    const listEl = document.getElementById(`${this.type}-projects-list`)! as HTMLUListElement
    listEl.innerHTML = ''
    for (const prjItem of this.assignedProjects) {
      new ProjectItem(this.element.querySelector('ul')!.id, prjItem)
    }
  }
}
```

- 할당된 프로젝트 목록을 출력하는 클래스의 항목에 신규 프로젝트 항목을 인스턴스화해 첫번째 인수로는 `hostId`를 전달하고 두번째 인수로는 `prjItem`을 전달

## 게터 사용하기

```tsx
class ProjectItem extends Component<HTMLUListElement, HTMLLIElement> {
  private project: Project

  get persons() {
    if (this.project.people === 1) {
      return '1 person'
    } else {
      return `${this.project.people} persons`
    }
  }

  constructor(hostId: string, project: Project) {
    super('single-project', hostId, false, project.id)
    this.project = project

    this.configure()
    this.renderContent()
  }

  configure() {}
  renderContent() {
    this.element.querySelector('h2')!.textContent = this.project.title
    this.element.querySelector('h3')!.textContent = this.persons + ' assigned'
    this.element.querySelector('p')!.textContent = this.project.description
  }
}
```

- `get`을 사용해 할당하고자 하는 인원이 2명 이상인 경우에는 `persons`(복수)를 붙여주거나 1명인 경우에는 `person`를 붙여 보다 유용한 정보를 산출할 수 있음
- `getter`는 함수와 같아서 괄호와 중괄호를 붙여 정의하고 반드시 `return` 값을 명시해야 함

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
