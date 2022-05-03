---
title: React State
date: '2022-04-25'
tags: ['react']
draft: false
summary: 예제로 알아보는 React State
layout: PostSimple
authors: ['default']
---

# 예제로 알아보는 React State

## Counter App - Not Good Way 🙄

```javascript
let counter = 0
function countUp() {
  counter = counter + 1
  render()
}
function render() {
  ReactDOM.render(<Container />, root)
}
const Container = () => (
  <div>
    <h3>Total clicks: {counter}</h3>
    <button onClick={countUp}>Click me</button>
  </div>
)
const root = document.getElementById('root')
render()
```

### Execute sequences

1. 어플리케이션이 시작될 때 Container를 렌더링함
2. Container가 렌더링 한 `counter`는 0
3. button을 클릭했을 때, 이벤트 리스너로 `countUp` 함수를 호출
4. `countUp` 함수는 `counter`를 1증가 시킴
5. 변경된 UI 사항에 대해서 `render` 함수가 호출되어 `Container`를 다시 렌더링함
6. 렌더링된 Container는 초깃값 0이 아닌, countUp 함수로 증가된 숫자인 1을 보여줌
   > 변경된 사항에 대해서 매번 render 함수를 호출해야 하므로, 비효율적임

#### Vanilla JS와 비교

- Vanilla JS 예제와 비교하여 개발자 도구에서 확인
  - Vanilla JS
    - countUp 함수 호출 시, 변경된 사항의 인접해있는 요소(element)들까지 갱신
  - React - countUp 함수 호출 시, 리액트는 UI에서 변경되는 부분만 갱신
    > React는 변경된 사항에 모든 요소를 갱신하지 않고, 해당되는 요소만 감지하여 갱신

## Counter App - Best Way 😁

- 위에 예제에서 확인했듯이, 변경되는 사항에 대해서 매번 렌더링을 해줘야하는 번거로움이 있음
- React.js는 어플 내에서 데이터를 관리하는 방법인 `State`를 사용할 수 있음

```javascript
function App() {
  const [counter, setCounter] = React.useState(0)
  const onClick = () => {
    setCounter(counter + 1)
  }
  return (
    <div>
      <h3>Total clicks: {counter}</h3>
      <button onClick={onClick}>Click me</button>
    </div>
  )
}
const root = document.getElementById('root')
ReactDOM.render(<App />, root)
```

- useState를 사용해서 초기값을 지정하고, 초기값을 넘겨받은 counter는 컴포넌트에 변수로 전달
- 버튼을 클릭했을 때, 이벤트 리스너로 전달된 onClick 함수가 실행
- setCounter 함수에 counter + 1한 값이 전달되면, 변경된 counter 값과 함께 `바뀐 부분(Component)만` UI를 리렌더링 함
  > 데이터가 바뀔때마다 컴포넌트를 리렌더링하고 UI를 refresh함

### React.useState()

- data 변수에 할당되는 React.useState()를 console.log에서 보면 `[undefined, func]`로 출력
  - 첫번째 요소는 초기값을 지정할 수 있음, 초기값을 지정하지 않으면 undefined를 확인할 수 있음
  - 두번째 요소는 data 값을 바꿀 때 사용할 함수

### State Functions

- setState 함수에서 직접 변경되는 값을 할당할 수 있으나, 이전 값을 바탕으로 현재 값을 설정할 수 있음

```javascript
const onClick = () => {
  // setCounter(counter + 1);
  setCounter((current) => current + 1)
}
```

- 현재 값을 argument로 가져와서 함수로 return 할 수 있음
  - 함수가 어떤 것을 return 하던 지, return 값은 새로운 state가 됨
- 주석처리한 부분과 동작은 동일하나, 리액트가 전달하는 current 값(counter)은 확실하게 현재 값이라는 것을 보장함

## Minute-Hour Converter App

- 분과 시를 변환해주는 앱

```javascript
function App() {
  const [amount, setAmount] = React.useState(0)
  const [flipped, setFlipped] = React.useState(false)
  const onChange = (event) => {
    setAmount(event.target.value)
  }
  const reset = () => {
    setAmount(0)
  }
  const onFlip = () => {
    reset()
    setFlipped((current) => !current)
  }
  return (
    <div>
      <h1 className="hi">Super Converter</h1>
      <div>
        <label htmlFor="minutes">Minutes</label>
        <input
          value={flipped ? amount * 60 : amount}
          id="minutes"
          placeholder="Minutes"
          type="number"
          onChange={onChange}
          disabled={flipped}
        />
      </div>
      <div>
        <label htmlFor="hours">Hours</label>
        <input
          value={flipped ? amount : Math.round(amount / 60)}
          id="hours"
          placeholder="Hours"
          type="number"
          onChange={onChange}
          disabled={!flipped}
        />
      </div>
      <button onClick={reset}>Reset</button>
      <button onClick={onFlip}>Flip</button>
    </div>
  )
}
const root = document.getElementById('root')
ReactDOM.render(<App />, root)
```

### Amount state

- 초기값이 0인 분(minutes)과 시(hours) input field의 value 값으로 컴포넌트에 전달

### Flipped state

- 초기값이 `false`인 flag state
- `flipped state`에 따라서 input의 value값과 `disabled` 여부가 달라짐
  - flipped가 `false`일 때
    - 초기값이 0인 분(minutes) input field가 활성화됨
    - 분(minuted)값에 따라 60초를 나눈 값(시)이 계산됨
  - flipped가 `true`일 때
    - 초기값이 0인 시(hours) input field가 활성화됨
    - 시(hours)값에 따라 60초를 곱한 값(분)이 계산됨

#### onChange func

- input field의 value가 바뀔때마다 onChange 이벤트로 감지하여 해당 함수를 호출
- event 인자의 taget > value 값(input value)을 amount state에 갱신하고 리렌더링

#### reset func

- React는 전체 페이지가 refresh되지 않고, 변경사항이 있는 요소만 리렌더링하므로 input field의 값을 초기화 처리

#### onFlip func

- flipped button을 눌렀을 때 호출되는 함수
- 시(hours)와 분(minutes) State의 현재 값을 반대 값(!)으로 뒤집고 flipped state를 갱신하여 리렌더링

> modifier 함수(setter function)를 이용해서 컴포넌트의 state 변경 시, 컴포넌트는 새로운 값을 가지고 다시 렌더링됨(컴포넌트 재생성)

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
