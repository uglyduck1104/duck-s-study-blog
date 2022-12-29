---
title: React.js 및 Typescript
date: '2022-12-29'
tags: ['typescript']
draft: false
summary: React + TypeScript 프로젝트 설정하기, React와 TypeScript는 어떻게 함께 작동하는가, Props으로 작업하기 & Props의 타입, ref로 사용자 입력 받기, Cross-Component 커뮤니케이션, 상태 및 타입 작업하기, 더 나은 상태 관리하기, 더 많은 Props 및 상태 작업
layout: PostSimple
authors: ['default']
---

# React.js 및 Typescript

## React + TypeScript 프로젝트 설정하기

### CRA(Create-React-App) 설치 및 적용

```bash
npx create-react-app my-app --template typescript
```

- `CRA`와 `—template typescript option`을 사용해 프로젝트를 쉽게 구성할 수 있음
- `tsconfig.json` 파일을 비롯한 `src`, `public` 폴더의 구조를 볼 수 있음
  - `src`
    - 이전의 예제들과 달리 `.tsx` 확장자로 구성된 파일을 타입스크립트만 쓸 수 있는 것이 아니라 `jsx` 코드도 쓸 수 있음

### `App.tsx`

```tsx
import React from 'react'

const App: React.FC = () => <div className="App"></div>

export default App
```

- Dummy로 생성된 파일들의 종속성을 위와 같이, 비워주고 `App.tsx`, `index.tsx`, `index.css`와 같이 필요한 파일만 남겨둠

## React와 TypeScript는 어떻게 함께 작동하는가

- `npm start` 커맨드로 개발 서버를 시작할 수 있음
- 최초 실행 시, host를 지정하지 않았다면 localhost:3000 port로 실행되며 현재 `App.tsx` 파일에서 렌더링하고 있지 않기 때문에 보이지 않는 것이 정상임

### `App.tsx`

```tsx
import React from 'react'

const App: React.FC = () => <div className="App"></div>

export default App
```

- 리액트 타입 패키지에서 제공되는 리액트 타입 중 `FC` 타입은 `Function Component`의 약자로써, `App` 안에 저장되는 것이 `function`이어야 한다는 것을 의미함
  - JSX나 React Element를 리턴해야 함
  - 타입스크립트는 추가적인 타입 Safe를 보장하고 잘못된 컴포넌트를 실행하는 경우를 방지함
- `const App: React.ClassicComponent` 와 같이 클래스 기반 컴포넌트도 사용이 가능함

## Props으로 작업하기 & Props의 타입

### TODO List Project

`App.tsx`

```tsx
import React from 'react'

import TodoList from './components/TodoList'

const App: React.FC = () => {
  const todos = [{ id: 't1', text: 'Finish the course' }]
  return (
    <div className="App">
      {/* A Component that adds todos */}
      <TodoList items={todos} />
    </div>
  )
}

export default App
```

- `todos` 배열을 선언해 `id: string, text: string` 타입인 객체를 정의
- `TodoList` 컴포넌트에 `items` 키를 가진 `props`(`todos`)를 전달
  - TodoList 컴포넌트보다는 앱 컴포넌트에 저장하는게 더 현실적임

> 타입스크립트를 통해 자동 완성 기능 및 오타로 인해 발생하는 `human error`를 사전에 방지할 수 있으며, 정의한 `Props` 및 `Interface`의 구조를 통해 안전한 타입을 보장받을 수 있음

`component/TodoList.tsx`

```tsx
import React from 'react'

interface TodoListProps {
  items: { id: string; text: string }[]
}

const TodoList: React.FC<TodoListProps> = (props) => {
  return (
    <ul>
      {props.items.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  )
}

export default TodoList
```

- `App.tsx`에서 렌더링 될 컴포넌트를 구성
- `Props`의 타입을 지정하기 위해, `TodoListProps`에 전달 받을 키와 값을 선언
  - 제네릭 타입을 사용해 위에서 정의한 인터페이스를 주입
- `React.FC`을 만들어서 평범한 `function`이 아니라 `function component`임을 타입스크립트에 알려줌

## ref로 사용자 입력 받기

### `NewTodo.tsx`

```tsx
import React, { useRef } from 'react'

const NewTodo: React.FC = () => {
  const textInputRef = useRef<HTMLInputElement>(null)

  const todoSubmitHandler = (event: React.FormEvent) => {
    event.preventDefault()
    const enteredText = textInputRef.current!.value
    console.log(enteredText)
  }

  return (
    <form onSubmit={todoSubmitHandler}>
      <div>
        <label htmlFor="todo-text">Todo Text</label>
        <input type="text" id="todo-text" ref={textInputRef} />
      </div>
      <button type="submit">Add Todo</button>
    </form>
  )
}

export default NewTodo
```

- 사용자 입력을 받고 `App` 컴포넌트로 값을 전달하기 위해, `useRef Hook`를 사용해 `input type`이 `text`이고 `id`값이 `todo-text`인 `element`에 `ref` 속성을 추가하고 `textInputRef`와 연결 시켜 줌
  - 할당하고자 하는 `Ref` 요소의 `Element Type`을 지정하고 `DOM`이 완전히 로딩되기 전 상태인 `null` 값을 할당
- `todoSubmitHandler`
  - 값을 입력한 후, `button type`이 `submit`인 버튼을 클릭하면 `form action`에서 `onSubmit` 핸들러와 `todoSubmitHandler` 함수를 연결시켜 줌
  - `trigger`된 이벤트의 타입을 `React.FormEvent`로 정의하고 기본 이벤트 동작을 막기위해 `event.preventDefault()` 함수를 실행
  - `ref`에 할당된 요소가 반드시 존재해야 하므로 `!` 표시를 통해 `value`값을 취득

## Cross-Component 커뮤니케이션

- `NewTodo` 컴포넌트에서 작성된 텍스트를 `App` 컴포넌트에 전달해야 함

### `App.tsx`

```tsx
import React from 'react'
import NewTodo from './components/NewTodo'

import TodoList from './components/TodoList'

const App: React.FC = () => {
  const todos = [{ id: 't1', text: 'Finish the course' }]

  const todoAddHandler = (text: string) => {
    console.log(text)
  }

  return (
    <div className="App">
      <NewTodo onAddTodo={todoAddHandler} />
      <TodoList items={todos} />
    </div>
  )
}

export default App
```

- `todoAddHandler` 함수를 만들고 `NewTodo` 컴포넌트에 `onAddTodo` 이름으로 포인터로 전달

### `NewTodo.tsx`

```tsx
import React, { useRef } from 'react'

type NewTodoProps = {
  onAddTodo: (todoText: string) => void
}

const NewTodo: React.FC<NewTodoProps> = (props) => {
  const textInputRef = useRef<HTMLInputElement>(null)

  const todoSubmitHandler = (event: React.FormEvent) => {
    event.preventDefault()
    const enteredText = textInputRef.current!.value
    props.onAddTodo(enteredText)
  }

  return (
    <form onSubmit={todoSubmitHandler}>
      <div>
        <label htmlFor="todo-text">Todo Text</label>
        <input type="text" id="todo-text" ref={textInputRef} />
      </div>
      <button type="submit">Add Todo</button>
    </form>
  )
}

export default NewTodo
```

- `TodoList` 컴포넌트와 마찬가지로 `type` 혹은 `interface`를 정의해 제네릭 타입으로 컴포넌트에 주입하고, `props` 인자로 전달받은 `onAddTodo` 포인터에 `ref`로 취득한 요소의 값을 전달
  - 전달할 매개변수의 타입을 `NewTodoProps` 타입에 정의하고 반환값을 `void`로 정의

## 상태 및 타입 작업하기

- `App` 컴포넌트에서 `todos` 배열을 업데이트하면 컴포넌트가 다시 렌더링 되지 않기 때문에, `State`로 관리해야 함

### `App.tsx`

```tsx
import React, { useState } from 'react'
import NewTodo from './components/NewTodo'
import TodoList from './components/TodoList'
import { Todo } from './components/todo.model'

const App: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([])

  const todoAddHandler = (text: string) => {
    setTodos([{ id: Math.random().toString(), text: text }])
  }

  return (
    <div className="App">
      <NewTodo onAddTodo={todoAddHandler} />
      <TodoList items={todos} />
    </div>
  )
}

export default App
```

- `useState` 훅을 사용해 초기 상태를 정의하고, `todos`와 `setTodos`을 배열형태로 선언

  - `todo.model.ts` 파일을 만들어 `interface`를 정의

    ```tsx
    export interface Todo {
      id: string
      text: string
    }
    ```

- `todoAddHandler`가 호출될 때, `setTodos` 함수를 실행해 `todos`의 상태를 업데이트 함
  - 현재 상태로는 `prevState`가 정의되지 않았기 때문에, 기존의 배열 값을 덮어씌우는 문제가 예상됨

## 더 나은 상태 관리하기

```tsx
const todoAddHandler = (text: string) => {
  setTodos((prevTodos) => [...prevTodos, { id: Math.random().toString(), text: text }])
}
```

- 이론적으로 리액트는 상태 업데이트를 일정한 시간에 실행하며 업데이트가 실행될때 `Todo`값이 가장 최신 상태가 아닐 수 있으므로 보장하기 위해서는 상태 업데이트하는 함수에 함수를 전달하는 형태를 권장
- 앞서 살펴본 `todoAddHandler` 함수는 기존의 값을 덮어씌우는 문제점이 발생하므로, `prevState`값을 함수안에 사용해 가장 최신 상태 스냅샷인 것을 명시해야 함

## 더 많은 Props 및 상태 작업

### `App.tsx`

```tsx
import React, { useState } from 'react'
import NewTodo from './components/NewTodo'
import TodoList from './components/TodoList'
import { Todo } from './components/todo.model'

const App: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([])

  const todoAddHandler = (text: string) => {
    setTodos((prevTodos) => [...prevTodos, { id: Math.random().toString(), text: text }])
  }

  const todoDeleteHandler = (todoId: string) => {
    setTodos((prevTodos) => {
      return prevTodos.filter((todo) => todo.id !== todoId)
    })
  }

  return (
    <div className="App">
      <NewTodo onAddTodo={todoAddHandler} />
      <TodoList items={todos} onDeleteTodo={todoDeleteHandler} />
    </div>
  )
}

export default App
```

- `todoDeleteHandler` 함수를 추가해, 매개변수로 전달받은 `id`값을 사용해 `todos`를 업데이트
  - `todoAddHandler` 함수와 마찬가지로 전 상태값을 가져와 `filter` 메소드로 순회하여 `id`값과 매개변수로 전달받은 `id`값이 일치하지 않은 새로운 배열을 반환

### `TodoList.tsx`

```tsx
import React from 'react'

interface TodoListProps {
  items: { id: string; text: string }[]
  onDeleteTodo: (id: string) => void
}

const TodoList: React.FC<TodoListProps> = (props) => {
  return (
    <ul>
      {props.items.map((todo) => (
        <li key={todo.id}>
          <span>{todo.text}</span>
          <button onClick={props.onDeleteTodo.bind(null, todo.id)}>DELETE</button>
        </li>
      ))}
    </ul>
  )
}

export default TodoList
```

- `TodoListProps` 인터페이스에 `onDeleteTodo` props의 타입을 정의
- `button` 요소를 추가하고 `bind` 메소드를 사용해 첫번째 인자 `this` 키워드는 null로 전달하고, 두번째 인자에 `onDeleteTodo`가 받을 첫 번째 매개변수(`todo.id`)를 전달

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
