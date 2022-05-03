---
title: React Simple ToDos Example
date: '2022-05-03'
tags: ['react']
draft: false
summary: ToDos 예제로 알아보는 State 개념
layout: PostSimple
authors: ['default']
---

# Simple ToDos

- 사용자의 입력을 받아, 특정 버튼을 누르면 저장된 todo list 목록을 화면에 출력

```javascript
import { useState } from 'react'

function App() {
  const [toDo, setToDo] = useState('')
  const [toDos, setToDos] = useState([])
  const onChange = (event) => setToDo(event.target.value)
  const onSubmit = (event) => {
    event.preventDefault()
    if (toDo === '') {
      return
    }
    setToDos((currentArray) => [toDo, ...currentArray])
    setToDo('')
  }
  return (
    <div>
      <h1>My To Dos ({toDos.length})</h1>
      <form onSubmit={onSubmit}>
        <input onChange={onChange} value={toDo} type="text" placeholder="Write your to do..." />
        <button>Add To Do</button>
      </form>
      <hr />
      <ul>
        {toDos.map((toDo, index) => (
          <li key={index}>{toDo}</li>
        ))}
      </ul>
    </div>
  )
}
export default App
```

## Execute Sequences

1. input field에 텍스트를 입력
2. 입력한 텍스트를 `onChange` 이벤트가 감지
3. 감지한 이벤트의 값을 `toDo` state에 저장
4. 'Add To Do' 버튼을 클릭하면 form 태그의 `onSubmit` 이벤트 감지
5. `toDo` state의 값이 없다면 그대로 리턴, 값이 있다면 현재 배열(currentArray)에 `toDo`값을 대입하고 `toDos`의 배열에 저장
   - `setToDos(currentState => [])`, `setToDos(function(currentArray){ return ""})`
     - 함수를 넣어, 현재의 state를 받아와서 새로운 array를 return 하는 방식
     - 반드시 새로운 값이어야 함
     - `hello`를 입력하고 submit이 실행된 후 `bye bye`를 입력했을 때,
       ```javascript
       setToDos((["hello"]) => ["bye bye", ...["hello"]]);
       ```
       - 이전의 element 값을 불러와서 새로운 array의 element를 추가함
   - `setToDo("")`
     - 값을 직접 대입하는 방식
6. map 고차함수를 이용해 첫번째 argument로 현재 순서에 맞는 item(toDos)을 가져와서 `새로운 배열을 반환`함
   - 첫번째 argument는 `value`, 두번째 argument는 `index`를 정의함
   - 리액트가 기본적으로 list에 있는 모든 item들을 인식하기 때문에 같은 컴포넌트의 list를 render할 때 key라는 prop을 넣어줘야 함
     - `key`는 반드시 `고유한 값`이어야 함
       - index argument로 key props를 대체할 수 있지만, 웬만하면 고유한 `key(id)`를 넘겨주는 것을 권장함

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
