---
title: React JSX
date: '2022-04-24'
tags: ['react', 'Clone-Coding']
draft: false
summary: VanillaJS, React 비교하면서 알아보는 JSX
layout: PostSimple
authors: ['default']
---

> ⚠️ 주의\
> 해당 방식은 React의 핵심을 짚고 넘어가기 위해서 예시로 든 예제임\
> `실제 React 개발 방식과 다름`

## VanillaJS Counter App

```html
<!DOCTYPE html>
<html>
  <body>
    <span>Total Clicks: 0</span>
    <button id="btn">Click me</button>
  </body>
  <script>
    let counter = 0
    const button = document.getElementById('btn')
    const span = document.querySelector('span')

    function handleClick() {
      counter += 1
      span.innerText = `Total Clicks: ${counter}`
      console.log(counter)
    }

    button.addEventListener('click', handleClick)
  </script>
</html>
```

- 버튼을 클릭하면 클릭한만큼의 counter 갯수를 화면에 출력해주는 코드

1. html을 정의
2. javascript를 가져옴
3. Event를 감지
4. 변경된 데이터를 업데이트

- 위와 같은 순서로 지금 당장에야 개발은 가능하지만 추후 복잡해질 코드에 대해 유연하게 대처할 수 없음
  - 계속 해당 element를 가져오고 event listener를 달다보면 비효율적인 생산성을 야기함

## ReactJS Counter App

### React, React-DOM 불러오기

```html
<!DOCTYPE html>
<html>
  <body></body>
  <script src="http://unpkg.com/react@17.0.2/umd/react.production.min.js"></script>
  <script src="http://unpkg.com/react-dom@17.0.2/umd/react-dom.production.min.js"></script>
</html>
```

- React library를 사용하기 위해서 react와 react-dom을 unpkg로 npm에 있는 package를 import함
- dev tool console 탭에 `React`를 입력하여 실제 React code가 정상적으로 로드되었는지 확인

### React.createElement()로 Element 생성하는 방식

```html
<script>
  const root = document.getElementById('root')
  const span = React.createElement(
    'span',
    { id: 'span-el', style: { color: 'red' } },
    "Hello I'm a span"
  )
  ReactDOM.render(span, root)
</script>
```

- createElement func를 가진 React obj를 만들고 span 변수에 할당
  - createElement의 대상이 되는 요소는 반드시 HTML 태그와 똑같은 이름이어야 함
  - 두번째 인자로 property(attribute)를 정의할 수 있음
  - 세번째 인자는 해당 el의 내용을 정의
- 생성한 Element를 html body에 둘 수 있도록(render) React-DOM도 반드시 의존관계에 있어야 함
- 바닐라 JS의 순서와 달리, Javascript -> HTML로 가는 순
  - ReactJS로 생성한 Element만 변동사항에 대해서 HTML을 업데이트하는 방식

### ReactJS로 Event 제어하는 방식

```html
<script>
  const root = document.getElementById('root')
  const h3 = React.createElement(
    'h3',
    {
      id: 'title',
      onMouseEnter: () => console.log('mouse enter'),
    },
    "Hello I'm a span"
  )
  const btn = React.createElement(
    'button',
    {
      onClick: () => console.log('Im clicked'),
      style: {
        backgroundColor: 'tomato',
      },
    },
    'Click me'
  )
  const container = React.createElement('div', null, [h3, btn])
  ReactDOM.render(container, root)
</script>
```

- 두번째 인자에 Property에 EventListener를 선언하고, 해당 이벤트를 정의함
  - 일일히 addEventListener를 주지 않아도 됨
- React JS는 정의된 Property가 HTML에 놓여야 한다는 것과 `on + event`는 event listener인 것을 알고 있음

### JSX로 Element 생성하는 방식

```html
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
// babel import
<script type="text/babel">
  // bable type declare
  const Title = (
    <h3 id="title" onMouseEnter={() => console.log('mouse enter')}>
      Hello I'm a span
    </h3>
  )
  const Button = (
    <button
      style={{
        backgroundColor: 'tomato',
      }}
      onClick={() => console.log('Im clicked')}
    >
      Click me
    </button>
  )
  const container = React.createElement('div', null, [Title, Button])
  ReactDOM.render(container, root)
</script>
```

- React.createElement() 방식과 동일하게 동작함
- HTML 생성방식과 유사해서 이해하기 더 쉬움
- 브라우저에서 JSX 문법을 이해하기 위해서는 Babel 구성이 선행되어야 함
- Babel에서 JSX로 변환한 문법을 다음과 같이 읽어들임

  ```javascript
  'use strict'

  const Title = /*#__PURE__*/ React.createElement(
    'h3',
    {
      id: 'title',
      onMouseEnter: () => console.log('mouse enter'),
    },
    "Hello I'm a span"
  )
  const Button = /*#__PURE__*/ React.createElement(
    'button',
    {
      style: {
        backgroundColor: 'tomato',
      },
      onClick: () => console.log('Im clicked'),
    },
    'Click me'
  )
  ```

  - React.createElement()로 생성했던 방식으로 변환해줌
  - JSX Code를 [Babel에서 제공하는 REPL 환경](https://babeljs.io/repl)에서 변환해볼 수 있음

### 컴포넌트 합성

```javascript
const Title = () => (
  <h3 id="title" onMouseEnter={() => console.log('mouse enter')}>
    Hello I'm a span
  </h3>
)
const Button = () => (
  <button
    style={{
      backgroundColor: 'tomato',
    }}
    onClick={() => console.log('Im clicked')}
  >
    Click me
  </button>
)
const Container = () => (
  <div>
    <Title />
    <Button />
  </div>
)
ReactDOM.render(<Container />, root)
```

- Title, Button 컴포넌트를 감싸고 있는 Container 또한 컴포넌트로 관리
- 컴포넌트로 선언하는 변수는 반드시 앞글자가 대문자로 시작해야 함
  - 소문자로 선언하는 변수는 html tag로 인식함
    > 다시 위로 올라가서 바닐라JS와 비교했을 때, React와 JSX 문법을 사용하는 이유에 대해서 다시 생각해 볼 필요가 있음

> **Referenced**

- [Nomad coding](https://nomadcoders.co/react-for-beginners)
