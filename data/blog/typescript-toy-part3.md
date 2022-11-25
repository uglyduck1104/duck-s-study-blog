---
title: Drag & Drop 토이 프로젝트 part 3
date: '2022-11-25'
tags: ['typescript']
draft: false
summary: Drag & Drop 토이 프로젝트 실습
layout: PostSimple
authors: ['default']
---

# Drag & Drop 토이 프로젝트 part3

## 드래그 앤 드롭 구현을 위한 인터페이스 활용하기

- 드래그와 드롭 인터페이스를 정의해서 특정 메소드를 실행할 수 있도록 구성해야 함

### 인터페이스 정의

```tsx
interface Draggable {
  dragStartHandler(event: DragEvent): void
  dragEndHandler(event: DragEvent): void
}

interface DragTarget {
  dragOverHander(event: DragEvent): void
  dropHandler(event: DragEvent): void
  dragLeaveHander(event: DragEvent): void
}
```

- `Draggable`
  - 드래그 가능한 인터페이스(두 핸들러가 필요)
    1. `dragStartHandler`
       - 드래그 앤 드롭의 `시작`을 실행할 이벤트
    2. `dragEndHandler`
       - 드래그 앤 드롭의 `끝`을 실행할 이벤트
  - 드래그 이벤트를 실행하기 때문에 별도의 리턴값을 가지지 않음
- `DragTarget`
  - 드래그 대상이 되는 요소
    1. `dragOverHandler`
       - 드래그 앤 드롭과 자바 스크립트를 실행할 때 드래그 오버 핸들러를 실행함
    2. `dropHandler`
       - 실제 일어나는 드롭에 대응
       - 주로 데이터와 UI를 업데이트 함
    3. `dropLeaveHandler`
       - 사용자에게 비주얼 피드백을 주고자 할 때 유용함
       - 드롭이 일어나지 않고 취소가 될 경우에는 비주얼 업데이트를 되돌릴 수 있음
  - `Draggable` 인터페이스와 마찬가지로 아무것도 반환하지 않음

### 인터페이스 확장

```tsx
class ProjectItem extends Component<HTMLUListElement, HTMLLIElement> implements Draggable {
  //...
  @autobind
  dragStartHandler(event: DragEvent) {
    console.log(event)
  }

  dragEndHandler(_: DragEvent) {
    console.log('DragEnd')
  }

  configure() {
    this.element.addEventListener('dragstart', this.dragStartHandler)
    this.element.addEventListener('dragend', this.dragEndHandler)
  }
  //...
}
```

- `ProjectItem`을 확장해서 `Draggable` 인터페이스를 `implements` 함
- `Draggable`에서 구현해야할 메소드를 선언
  - `dragStartHandler(event: DragEvent)`, `dragEndHandler(event: DragEvent)`
  - 드래그 가능한 컴포넌트나 드래그 클래스를 일관되게 쓸 수 있게 도와줌
  - `@autobind`
    - 데코레이터를 추가해 트리거를 볼 수 있게 함
- `configure()`
  - 해당 요소에 `dragstart`, `dragend` 이벤트 리스너를 추가해서 렌더링된 요소에 접근할 수 있음

### 드래그되는 대상 지정

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- ... -->
  </head>
  <body>
    <!-- ... -->
    <template id="single-project">
      <li draggable="true">
        <h2></h2>
        <h3></h3>
        <p></p>
      </li>
    </template>
    <!-- ... -->
    <div id="app"></div>
  </body>
</html>
```

- 드래그 되는 대상에 `draggable` 속성을 추가

## 드래그 이벤트 및 UI의 현재 상태 반영하기

```tsx
class ProjectList extends Component<HTMLDivElement, HTMLElement> implements DragTarget {
  //...

  @autobind
  dragOverHandler(_: DragEvent) {
    const listEl = this.element.querySelector('ul')!
    listEl.classList.add('droppable')
  }

  @autobind
  dropHandler(_: DragEvent) {}

  @autobind
  dragLeaveHandler(_: DragEvent) {
    const listEl = this.element.querySelector('ul')!
    listEl.classList.remove('droppable')
  }

  configure() {
    this.element.addEventListener('dragover', this.dragOverHandler)
    this.element.addEventListener('dragleave', this.dragLeaveHandler)
    this.element.addEventListener('drop', this.dropHandler)
    //...
  }
  //...
}
```

- `ProjectList` 클래스에 드래그 타겟 역할을 하는 인터페이스를 추가 (`implements DragTarget`)
- 인터페이스에서 요구하는 핸들러 메서드 3개를 구현
  - `dragOverHandler(event: DragEvent) {}`, `dropHandler(event: DragEvent) {}`, `dragLeaveHandler(event: DragEvent) {}`
    1. `dragOverHandler(event: DragEvent) {}`
       - 목록 요소(`ul`)에 접근해 클래스에 `droppable` 클래스를 추가
       - 렌더링된 섹션 위에 드래그를 하고자할 때 실행
    2. `dragLeaveHandler(event: DragEvent) {}`
       - `dragOverHandler`와 반대로 목록 요소(`ul`)에 접근해 클래스에 `droppable` 클래스를 삭제
       - 렌더링된 섹션 위에 드래그 이벤트가 떠날 때 실행
  - `@autobind`
    - 데코레이터를 추가해 트리거를 볼 수 있게 함
- `app.css`에서 선언한 `droppable` 클래스 속성을 명시
- `configure` 메서드에 핸들러 등록
  - `this.element.addEventListener("dragover", this.dragOverHandler);`
  - `this.element.addEventListener("dragleave", this.dragLeaveHandler);`
  - `this.element.addEventListener("drop", this.dropHandler);`

## 드롭할 수 있는 영역 추가하기

- 자바스크립트는 무엇이, 어디로 드래그되는지, 상태를 어떻게 업데이트할 지 현재로써는 모르므로, 드래깅 가능한 항목(`ProjectItem`)에서 `dragStartHandler`에 이벤트 객체를 활용

```tsx
class ProjectItem extends Component<HTMLUListElement, HTMLLIElement> implements Draggable {
  // ...

  @autobind
  dragStartHandler(event: DragEvent) {
    event.dataTransfer!.setData('text/plain', this.project.id)
    event.dataTransfer!.effectAllowed = 'move'
  }
  //...
}
```

- `event.dataTransfer` 프로퍼티를 사용해 `text/plain` 형태의 `project` 요소(`DOM`)의 `id` 속성에 추가
- 데이터 전송에 데이터를 설정할 수 있으나, `null` 상태임을 `!` 표시로 명시
  - 드래그 이벤트는 언제나 같은 종류의 이벤트지만 어떤 리스너가 발생시키는지 또는 어떤 이벤트를 처리하고 있는지에 따라 데이터 전송이 불가능하므로 `null` 일 수 있음
  - `setData`는 두 개의 매개변수와 인수를 필요로 함
    - 첫번째 인수 → 데이터 포맷의 식별자
    - 두번째 인수 → 전송하고자 하는 데이터
- `effectAllowed = 'move'`
  - `allowed` 프로퍼티가 움직이도록 효과 설정

```tsx
class ProjectList extends Component<HTMLDivElement, HTMLElement> implements DragTarget {
  // ...

  @autobind
  dragOverHandler(event: DragEvent) {
    if (event.dataTransfer && event.dataTransfer.types[0] === 'text/plain') {
      event.preventDefault()
      const listEl = this.element.querySelector('ul')!
      listEl.classList.add('droppable')
    }
  }

  @autobind
  dropHandler(event: DragEvent) {
    console.log(event.dataTransfer!.getData('text/plain'))
  }

  // ...
}
```

- `dragOverHandler` 에서 `dataTranfer` 객체에 전달된 데이터 전송 `type`의 첫번째 인수값이면 `draoppable` 클래스를 추가하는 로직을 구현
- `event.preventDefault()` 메서드를 통해 요소 하나에만 드롭 이벤트가 허용하도록 함
  - 드래그 앤 드롭 이벤트 디폴트는 드로핑을 허용하지 않기때문에 막아줘야 함
  - 해당 메서드를 사용하지 않으면 사용자가 놓더라도 드롭이벤트가 실행되지 않음
- `dropHandler`에서 `console.log` 메서드를 통해 `dataTransfer` 객체로 전달된 데이터(`text/plain`)를 정상적으로 불러오는지 확인

## 드래그 앤 드롭 마무리하기

```tsx
class ProjectState extends State<Project> {
  private projects: Project[] = []
  private static instance: ProjectState

  // ...

  addProject(title: string, description: string, numOfPeople: number) {
    const newProject = new Project(
      Math.random().toString(),
      title,
      description,
      numOfPeople,
      ProjectStatus.Active
    )
    this.projects.push(newProject)
    this.updateListeners()
  }

  moveProject(projectId: string, newStatus: ProjectStatus) {
    const project = this.projects.find((prj) => prj.id === projectId)
    if (project && project.status !== newStatus) {
      project.status = newStatus
      this.updateListeners()
    }
  }

  private updateListeners() {
    for (const listenerFn of this.listeners) {
      listenerFn(this.projects.slice())
    }
  }
}
```

- 이벤트 핸들러에 따라 프로젝트의 상태를 관리하기 위해, `addProject` 메소드를 추가해 프로젝트 Id 값을 기준으로, 상태 값을 변경
- find 메서드를 통해 프로젝트의 요소에 접근해서 찾는 요소가 맞는지 Id값을 비교함
  - 해당하는 요소가 존재하면 상태값을 `newStatus`로 할당
  - 불필요한 리렌더링을 방지하기 위해 상태값이 `newStatus`가 아닐 경우에만 렌더링하게 처리
- `updateListeners` 함수를 만들어서 모든 리스너들을 순회

```tsx
class ProjectList extends Component<HTMLDivElement, HTMLElement> implements DragTarget {
  assignedProjects: Project[]

  // ...

  @autobind
  dropHandler(event: DragEvent) {
    const prjId = event.dataTransfer!.getData('text/plain')
    projectState.moveProject(
      prjId,
      this.type === 'active' ? ProjectStatus.Active : ProjectStatus.Finished
    )
  }

  // ...
}
```

- `dropHandler(event: DragEvent){}`
  - 드롭 핸들러에 moveProject 메소드를 호출해서 프로젝트의 상태를 불러와서, 상태(`active | finished`)에 따라 할당함

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
