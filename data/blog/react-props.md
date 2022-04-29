---
title: React Props
date: '2022-04-27'
tags: ['react', 'Clone-Coding']
draft: false
summary: 부모 컴포넌트로부터 자식 컴포넌트에 데이터를 보낼 수 있게 해주는 방법
layout: PostSimple
authors: ['default']
---

# 예제로 알아보는 React Props

## 컴포넌트 합성 및 Props 전달

- 부모 컴포넌트로부터 자식 컴포넌트에 데이터를 보낼 수 있게 해주는 방법

```javascript
function Btn({ text, big }) {
  return (
    <button
      style={{
        backgroundColor: 'tomato',
        color: 'white',
        padding: '10px 20px',
        border: 0,
        borderRadius: 10,
        fontSize: big ? 18 : 16,
      }}
    >
      {text}
    </button>
  )
}
function App() {
  return (
    <div>
      <Btn text="Save Change" big={true} />
      <Btn text="Continue" big={false} />
    </div>
  )
}
const root = document.getElementById('root')
ReactDOM.render(<App />, root)
```

- Btn 컴포넌트는 `함수`이기 때문에 argument(인자)를 전달할 수 있음
  - props는 첫 번째이자 유일한 인자임
- Btn 컴포넌트에 `text`, `big`이라는 이름의 props key와 value를 전달
  - props에 담겨있는 `text`와 `big`을 object 형태로 추출
    - props 객체를 넘겨서 `props.text, props.big`로 처리해도 동일하게 동작
    - 대신 넘기는 props의 이름과 받는 props의 이름은 반드시 일치해야 함
  - text: 화면에 출력될 텍스트 노드
  - big: `big` props의 boolean값에 따라 fontSize를 다르게 지정

## Memo

- state가 갱신되면 컴포넌트를 다시 렌더링하는데, 컴포넌트 합성 구조에서 자식 컴포넌트까지 같이 재생성되는 경우가 있음
  - 의도치 않은 컴포넌트가 같이 리렌더링 되는 경우 발생
  - 추후에 어플리케이션이 느려지는 원인이 될 수 있음
- Memo를 사용하여 지정한 컴포넌트는 렌더링 되지 않도록 처리할 수 있음

```javascript
function Btn({ text, onClick }) {
  return <button onClick={onClick}>{text}</button>
}
const MemorizedBtn = React.memo(Btn)
function App() {
  const [value, setValue] = React.useState('Save Change')
  const changeValue = () => setValue('Revert Change')
  return (
    <div>
      <MemorizedBtn text={value} onClick={changeValue} />
      <MemorizedBtn text="Continue" />
    </div>
  )
}
const root = document.getElementById('root')
ReactDOM.render(<App />, root)
```

- Props에는 함수도 전달 가능함
- Btn 컴포넌트(Custom Component)에 전달하는 함수는 이벤트리스너(onClick)와 동일한 이름이어도 `props`로써 전달됨
- Btn 첫 번째 버튼의 props는 state와 연결되어 있음
  - Btn 컴포넌트에 changeValue 함수가 호출되면서 부모의 컴포넌트(App)는 자식 요소들을 리렌더링 함
- `memorizedBtn`에 Memo를 적용한 Btn 컴포넌트를 할당
  > 부모의 컴포넌트에 state 변동사항에 따라 자식 컴포넌트는 모두 다시 렌더링됨

## Prop Types

- props를 전달하고자 하는 컴포넌트에 잘못된 형태(type casting)로 전달하는 경우 오류 사항에 대해 인지하지 못하는 문제 발생
- PropType은 어떤 타입의 prop를 받고 있는지 체크해줌

### prop-type.js 불러오기

```html
<script src="https://unpkg.com/prop-types@15.7.2/prop-types.js"></script>
```

- PropType이라고 불리는 오브젝트에 접근 가능
- 각기 다른 타입들을 검사하는게 가능함
  - [참고 링크](https://reactjs.org/docs/typechecking-with-proptypes.html)

### 예제

```javascript
function Btn({ text, fontSize = 14 }) {
  return (
    <button
      style={{
        backgroundColor: 'tomato',
        color: 'white',
        padding: '10px 20px',
        border: 0,
        borderRadius: 10,
        fontSize,
      }}
    >
      {text}
    </button>
  )
}
Btn.prototype = {
  text: PropTypes.string.isRequired, // 필수값으로 string type이 전달되야 함
  fontSize: PropTypes.number, // isRequired가 아니면 필수는 아님, 대신 기본 값을 할당해야함
}
function App() {
  return (
    <div>
      <Btn text="Save Changes" fontSize={18} />
      <Btn text="Continue" />
    </div>
  )
}
const root = document.getElementById('root')
ReactDOM.render(<App />, root)
```

- Btn 컴포넌트(custom component)에 전달할 props키들에 대한 type을 지정
- isRequired는 반드시 전달되어야 하는 필수 값
  - 필수값으로 string type이 전달되야 함
  - isRequired가 아닌 값은 예제처럼 props를 넘겨받을 때, 기본 값을 할당해주는 게 좋음(Javascript 문법)

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
