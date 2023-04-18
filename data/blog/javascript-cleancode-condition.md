---
title: 분기 다루기
date: '2023-04-18'
tags: ['javascript']
draft: false
summary: 값식문, 삼항 연산자 다루기, Truthy & Falsy, 단축평가, else if 피하기, else 피하기, Early Return, 부정 조건문 지양하기, Default Case 고려하기
layout: PostSimple
authors: ['default']
---

복습: No

# 분기 다루기

## 값식문

### Case 1 - ReactJSX로 생각해보기

```jsx
// This JSX:
ReactDOM.render(<div id="msg">Hello World!</div>, mountNode)

// Is transformed to this JS:
ReactDOM.render(React.createElement('div', { id: 'msg' }, 'Hello World!'), mountNode)
```

- `ReactDOM.render(JSX, mountNode)`
  - 주로 함수와 관련되어 있는 것으로 JSX와 mountNode 인자를 넘기고 있음
  - `Babel`을 만나 `Transfile` 되면서 특성 속성과 태그, 문자열이 객체 구조로 넘어가고 있음

### Case 2 - 값식문의 사용

```jsx
// This JSX:
// <div id={if (condition) { 'msg' }}>Hello World!</div>

// Is transformed to this JS:
// React.createElement("div", {id: if (condition) { 'msg' }}, "Hello World!");

ReactDOM.render(<div id={condition ? 'msg' : null}>Hello World!</div>, mountNode)
```

- `if` 문을 객체 키의 값으로 대입했음(값 → 문)

  - 해당 코드는 동작해서되지 않음
  - 값이 들어와야하는데 문이 들어왔으므로 문법적 오류가 발생함

  > 그렇다면 식은 무엇일까?

  - `condition ? 'msg' : null` 처럼 식은 값으로 귀결될 수 있는 것

### Case 3 - 즉시실행함수(`IIFE`)로 보는 switch 문

```jsx
function ReactComponent() {
  return (
    <section>
      <h1>Color</h1>
      <h3>Name</h3>
      <p>{this.state.color || 'white'}</p>
      <h3>Hex</h3>
      <p>
        {(() => {
          switch (this.state.color) {
            case 'red':
              return '#FF0000'
            case 'green':
              return '#00FF00'
            case 'blue':
              return '#0000FF'
            default:
              return '#FFFFFF'
          }
        })()}
      </p>
    </section>
  )
}
```

- `{this.state.color || "white"}`
  - 논리 연산자를 통해 값과 식으로도 분기문 없이 사용이 가능
- `switch` 문 자체로는 동작하지 않지만, 즉시실행함수(`IIFE`)를 통해 내부에서 값을 리턴하므로 사용이 가능함

### Case 4 - 즉시실행함수(`IIFE`)로 보는 안티패턴 1

```jsx
function ReactComponent() {
  return (
    <tbody>
      {(() => {
        const rows = [];

        for (let i = 0; i < objectRows.length; i++) {
          rows.push(<ObjectRow key={i} data={objectRows[i]} />);
        }
        return rows;
      })()}
    </tbody>
  );
}
```

- 임시 변수에 빈 배열을 할당해서 초기화한 후, `for`문을 돌려 임시 변수에 누적하여 `IIFE`로 반환

```jsx
function ReactComponent() {
  return (
    <tbody>
      {objectRows.map((obj, i) => (
        <ObjectRow key={i} data={obj} />
      ))}
    </tbody>
  )
}
```

- 고차함수를 사용
  - `map` 함수를 사용해 값과 식으로 이루어진 코드로 수정할 수 있음

### Case 5 - 즉시실행함수(`IIFE`)로 보는 안티패턴 2

```jsx
function ReactComponent() {
  return (
    <div>
      {(() => {
        if (conditionOne) return <span>One</span>
        if (conditionTwo) return <span>Two</span>
        else conditionOne
        return <span>Three</span>
      })()}
    </div>
  )
}
```

- Case 4와 마찬가지로, 즉시 실행 함수로 구현된 소스코드

```jsx
function ReactComponent() {
  return (
    <div>
      {conditionOne && <span>One</span>}
      {conditionTwo && <span>Two</span>}
      {!conditionTwo && <span>Three</span>}
    </div>
  )
}
```

- 위의 예제 코드로 논리 연산자를 사용해 리팩터링을 진행할 수 있음

> `Babel`을 통해서 트랜스파일링된 자바스크립트 코드는 객체로 바뀌는데, 객체로 바뀌는 모든 건 내부에 값과 식으로만 넣을 수 있으므로 고차함수를 활용해 코드를 작성해야 함

## 삼항 연산자 다루기

- 3개의 피연산자를 취하는 삼항 연산자(`조건 ? 참 : 거짓`)
  - 참과 거짓에는 `식`이 들어감

### Case 1 - 가독성과 유지보수의 관점

```jsx
function example1() {
  return condition1 ? value1 : condition2 ? value2 : condition3 ? value3 : value4
}

function example2() {
  if (condition1) {
    return value1
  } else if (condition2) {
    return value2
  } else if (condition3) {
    return value3
  } else {
    return value4
  }
}
```

- `reformat`을 사용하고도 가독성과 유지보수 측면에서 봤을 때, `return` 값을 읽기 어려운 경우 `switch` 문을 사용

### Case 2 - 중첩 삼항 연산자

```jsx
const example = condition1 ? (a === 0 ? 'zero' : 'positive') : 'negative'
```

- 3개의 피연산자 이외의 중첩 피연산자로 인해 의미가 불분명함
- `()` 소괄호는 연산자 중에 우선순위를 선점하므로 활용해서 가독성을 높이는 방안도 고려할 수 있음

### Case 3 - 기본값 할당의 활용으로 보는 삼항 연산자

```jsx
const welcomeMessage = (isLogin) => {
  const name = isLogin ? getName() : '이름없음'

  return `안녕하세요 ${name}`
}
```

- 매개변수로 `isLogin`를 받아 조건에 따라서 `getName` 함수를 실행하거나, 이름없음 문자열을 na`m`e 변수에 할당하여 반환
  - `isLogin`은 `nullable`한 값이 넘어올 수 있으므로 기본값의 의미로 활용할 수 있음
- `if` 문으로도 가능하지만, 삼항연산자로 코드를 더 간결하게 사용할 수 있으며 일정 중복 코드를 방지할 수 있음

  ```jsx
  const welcomeMessage = (isLogin) => {
    if (isLogin) return `안녕하세요 ${getName()}`

    return '안녕하세요 이름없음'
  }
  ```

### Case 4 - 안티 패턴(Bad Case)

```jsx
function alertMessage(isAdult) {
  isAdult ? alert('입장이 가능합니다.') : alert('입장이 불가능합니다.')
}
```

- `alert`은 `void`(반환 값이 없음) `type`이므로, 겉으로는 다르게 느껴지지만 어떠한 값이 들어와도 같음
  - `undefined`가 할당될 수 있음
- 억지로 삼항연산자를 사용하는 것보다는 `if`, `else` 문을 사용

### 정리 - 삼항연산자를 사용하는 경우

- 삼항연산자를 사용하는 경우는 결국 어떠한 조건으로 인해 값을 만들고 변수에 할당할 때 사용
- 함수가 값을 반환하는 경우
- 명확한 조건을 기준으로 로직을 분리하는 경우
- `3개`의 피연산자를 명확하게 사용할 경우

## Truthy & Falsy

- 자바스크립트 언어 특성상 암묵적인 형변환이 일어나는 경우가 빈번함

### Truthy(참 같은 값)

```text
if (true)
if ({})
if ([])
if (42)
if ("0")
if ("false")
if (new Date())
if (-42)
if (12n)
if (3.14)
if (-3.14)
if (Infinity)
if (-Infinity)
```

### Falsy(거짓 같은 값)

```text
if (false)
if (null)
if (undefined)
if (0)
if (-0)
if (0n)
if (NaN)
if ("")
```

### Case - falsy 활용

```jsx
function printName(name) {
  if (name === undefined || name === null) {
    return '사람이 없네요'
  }

  return '안녕하세요 ' + name + '님'
}

printName()
```

- `undefined`와 `null`은 `falsy`한 값이므로 `name`이 문자열이 있을 경우에는 `truthy` 없을 경우에는 falsy로 평가됨
  - `if (name === undefined || name === null)` 구문을 `if(!name)`으로 `falsy`값을 사용해 개선이 가능함

## 단축평가 (short-circuit evaluation)

```jsx
/**
 * AND
 */
true && true && '도달 O'
true && false && '도달 X'

/**
 * OR
 */
false || false || '도달 O'
true || true || '도달 X'
```

- `AND`
  - 좌항에서 우항으로 모든 항이 참이어야 함
- `OR`
  - 좌항에서 우항으로 되는 것은 맞지만, 그 중에 하나라도 참인 경우 다음 항에 도달하지 않음

### Case 1 - OR 연산자로 단축 평가 응용하기

```jsx
function fetchData() {
  //   if (state.data) {
  //     return state.data;
  //   } else {
  //     return "Fetching...";
  //   }

  return state.data || 'Fetching...'
}
```

- `state.data`가 `false` 인경우 `true`를 찾기 위해, 우항(`"Fetching…"`)으로 이동
  - `Fetching`이라는 문자열은 참으로 평가되므로 기본값으로 사용할 수 있음

### Case 2 - OR 연산자로 분기문 개선하기

```jsx
function favoriteDog(someDog) {
  //   let favoriteDog;
  //   if (someDog) {
  //     favoriteDog = dog;
  //   } else {
  //     favoriteDog = "냐옹";
  //   }

  //   return (favoriteDog) + "입니다";
  return (someDog || '냐옹') + '입니다'
}

favoriteDog() // => 냐옹
favoriteDog('포메') // => 강아지 입니다.
```

- `someDog`가 `falsy`한 값(`undefined` 혹은 `null`)일 경우 냐옹으로 평가되므로, 코드를 줄일 수 있음

### Case 3 - OR 연산자로 기본값 처리하기

```jsx
function getActiveUserName(user, isLogin) {
  //   if (isLogin && user) {
  //     if (user.name) {
  //       return user.name;
  //     } else {
  //       return "이름없음";
  //     }
  //   }
  if (isLogin && user) {
    return user.name || '이름없음'
  }
}
```

- 활성화된 유저의 이름을 반환하는 코드로써, 중첩된 조건문으로 인해 결과 값을 예측하기 어려움
- `AND` 논리 연산자를 사용해 `isLogin`과 `user`가 `truthy` 한 값이여야만 조건문이 실행됨
- 앞에 살펴본 예제처럼 단축 평가를 활용해 기본값을 할당함과 더불어 조건문도 개선할 수 있음

## else if 피하기

```jsx
const x = 1

if (x >= 0) {
  console.log('x는 0과 같거나 크다')
} else if (x > 0) {
  console.log('x는 0보다 크다 ')
}
```

- `else if` → `else`를 한번 더 실행하고 `if`를 처리하므로 `if`로 분리하는게 더 명확함

  ```jsx
  const x = 1

  if (x >= 0) {
    console.log('x는 0과 같거나 크다')
  }
  if (x > 0) {
    console.log('x는 0보다 크다 ')
  }
  ```

- `else if` 가 늘어나면 차라리 `switch` 문을 사용

## else 피하기

### Case 1 - else 문 대신 return 문

```jsx
function getUserName(user) {
  /*
	if (user.name) {
		return user.name;
	} else {
		return '이름없음';
	}
	*/
  if (user.name) {
    return user.name
  }

  return '이름없음'
}
```

- 자바스크립트는 `return`을 만나면 함수 자체가 종료되므로, `if` 문으로 조건문만 처리 할 수 있음

### Case 2 - if ~ else 문의 문제점

```jsx
/**
 * age가 20 미만시 특수 함수 실행
 */
function getHelloCustomer(user) {
  /*
		if (user.age < 20) {
			report(user);
		} else {
			return '안녕하세요';
		}
	*/
  if (user.age < 20) {
    report(user)
  }

  return '안녕하세요'
}
```

- `user` 매개변수로 성인이 아닌 경우에 `else`문에 도달하지 못하기 때문에, `else`문을 지우고 특수 함수가 실행된 후 `안녕하세요` 문자열을 반환할 수 있음

## Early Return

### Case 1. 로그인 검증

```jsx
function loginService(isLogin, user) {
  if (!isLogin) {
    if (checkToken()) {
      if (!user.nickName) {
        return registerUser(user)
      } else {
        refreshToken()

        return '로그인 성공'
      }
    } else {
      throw new Error('No Token')
    }
  }
}
```

- 흐름
  1. 로그인 여부 확인
  2. 토큰 존재 여부 확인
  3. 기가입유저 확인
  - 가입이 안되어있으면 가입 혹은 리프레스 토큰 생성

```jsx
function loginService(isLogin, user) {
  /* Early Return
   * 함수를 미리 종료
   * 사고하기 편함
   */
  if (!isLogin) return

  if (!checkToken()) throw new Error('No Token')

  if (!user.nickName) return registerUser(user)

  refreshToken()
  return '로그인 성공'
}
```

- `Early Return`을 통해 예외케이스를 if문으로 나누어서 미리 종료시킴
- `refreshToken` 함수와 로그인 성공 문자열 반환하는 부분은 위의 조건문이 걸리지 않은 상태로 실행

## 부정 조건문 지양하기

```jsx
const isCondition = true
const isNotCondition = false

if (isCondition) {
  console.log('참인 경우에만 실행') // 참인 경우에만 실행(전제 조건)
} else {
  console.log('거짓인 경우에만 실행')
}

// 숫자일때만
function isNumber(num) {
  return !Number.isNaN(num) && typeof num === 'number'
}

if (isNumber(3)) {
  // if (!isNaN(3)) {
  console.log('숫자입니다')
}
```

- `isNaN`의 경우 해당 인수가 숫자일때 참인지 거짓인지 명확하지 않기 때문에, 숫자일때만 참인 헬퍼 함수를 만들어서 조건문으로 사용하는 것이 부정 연산자를 사용하는 것보다 의미를 이해하기 쉬움
- 부정 조건문 지양해야 하는 이유
  1. 생각을 여러번 해야할 수 있음
  2. 프로그래밍 언어 자체로 `if`문이 처음부터오고 참인 경우부터 실행시킴
- 부정 조건문 예외
  1. Early Return 사용
  2. Form Validation 검증 로직
  3. 보안 혹은 검사하는 로직

> 헷갈리는 코드보다는 명시적인 코드를 작성하기 위해서는 부정 연산자는 지양해야 함

## Default Case 고려하기

- 사용자로부터 값을 얻지 못하거나, 전달되는 값이 없는 경우를 예외를 두어 안전하게 프로그래밍해야 함(정책)

### Case 1. OR 연산자로 예외 처리 1

```jsx
function sum(x, y) {
  x = x || 1
  y = y || 1

  return x + y
}

sum()
```

- `x`, `y`값이 존재하지 않은 경우 우항인 1로 할당한 후, 더한 값을 반환

### Case 2. OR 연산자로 예외 처리 2

```jsx
function createElement(type, height, width) {
  const element = document.createElement(type || 'div')

  element.style.height = height || 100
  element.style.width = width || 100

  return element
}

createElement()
```

- 코어 라이브러리 함수나, 유틸리티 함수를 구현하는 경우에 항상 `defaultCase`를 염두해 구현하면 안전하고 확장성 높은 프로그래밍을 할 수 있음

### Case 3. switch로 defaultCase 처리

```jsx
function registerDay(userInputDay) {
  switch (userInputDay) {
    case '월요일': // some code
    case '화요일': // some code
    case '수요일': // some code
    case '목요일': // some code
    case '금요일': // some code
    case '토요일': // some code
    case '일요일': // some code
    default:
      throw Error('입력값이 유효하지 않습니다.')
  }
}

e.target.value = '월ㄹ요일'
registerDay(e.target.value)
```

- 인수로 넘겨주는 문자열에 오타와 같은 휴먼에러 발생 시, `defaultCase`를 예외로 설정
- 사용자의 값을 넘겨받는 경우에는 더욱이 중요하게 고려해야 함

### Case 4. React Switch 컴포넌트

```jsx
const Root = () => (
  <Router history={browserHistory}>
    <Switch>
      <Route exact path="/" component={App} />
      <Route path="/welcome" component={Welcome} />
      <Route component={NotFound} />
    </Switch>
  </Router>
)
```

- 누구나 `Switch` 케이스 문을 사용하면 마지막에는 `Edge Case`를 고려할 수 있음

> **Referenced**

- [클린코드 자바스크립트](https://www.udemy.com/course/clean-code-js/)
