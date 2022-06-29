---
title: 에러 처리
date: '2022-06-29'
tags: ['javascript']
draft: false
summary: 에러에 대해 대처하지 않고 방치하면 프로그램은 강제 종료되므로 try…catch 문을 사용해 발생한 에러에 적절하게 대응하면 프로그램이 강제 종료되지 않고 계속해서 코드를 실행시킬 수 있음
layout: PostSimple
authors: ['default']
---

# 에러 처리

## 에러 처리의 필요성

- 에러에 대해 대처하지 않고 방치하면 프로그램은 강제 종료되므로 try…catch 문을 사용해 발생한 에러에 적절하게 대응하면 프로그램이 강제 종료되지 않고 계속해서 코드를 실행시킬 수 있음

    ```jsx
    console.log('[Start]');
    
    try{
    	foo();
    }	catch (error) {
    	console.error('[에러 발생', error);
    	// [에러 발생] ReferenceError: foo is not defined
    }
    // 발생한 에러에 적절한 대응을 하면 프로그램이 강제 종료되지 않음
    console.log('[End]');
    ```

  - 직접적으로 에러를 발생하지는 않는 예외적인 상황이 발생할 수도 있음

    ```jsx
    // DOM에 button 요소가 존재하는 경우 querySelector 메서드는 에러를 발생시키지 않고 null을 반환함
    const $button = document.querySelector('button'); // null
    $button?.classList.add('disabled');
    ```

> 우리가 작성한 코드에서는 언제나 에러나 예외적인 상황이 발생할 수 있다는 것을 전제하고 이에 대응하는 코드를 작성해야 함

## `try…catch…finally` 문

- 에러 처리 구현하는 방법은 조건문, 단축 평가, 옵셔널 체이닝 연산자 등을 통해 확인해서 처리하는 방법과 에러 처리 코드를 미리 등록해 두고 에러가 발생하면 에러 처리 코드로 점프하도록 하는 방법이 있음
- `try..catch…finally` 문은 에러 처리를 통해 3개의 코드 블럭을 구성함
  - `finally`문은 생략 가능

    ```jsx
    try {
    	// 실행할 코드(에러가 발생할 가능성이 있는 코드)
    } catch (err) {
    	// try 코드 블록에서 에러가 발생하면 이 코드 블록의 코드가 실행됨
    	// err에는 try 코드 블록에서 발생한 Error 객체가 전달
    } finally {
    	// 에러 발생과 상관없이 반드시 한 번 실행
    }
    ```

1. `try` 코드 블록 실행

- try문의 에러발생 시 catch 문의 err 변수에 전달

2. `catch` 코드 블록 실행

- catch문의 err 변수는 에러가 발생되면 생성
- catch 코드 블록에만 유효함

3. `finally` 코드 블록 실행

- 에러 발생과 상관없이 `반드시 한 번 실행`

## Error 객체

- Error 생성자 함수에는 에러를 상세히 설명하는 에러 메시지를 인수로 전달할 수 있음

```jsx
const error = new Error('invalid');
```

- `message` 프로퍼티와 `stack` 프로퍼티를 가짐
  - `message`
    - Error 생성자 함수에 인수로 전달한 `에러 메시지`
  - `stack`
    - 에러를 발생시킨 콜스택의 호출 정보를 나타내는 문자열, `디버깅 용도`

| 생성자 함수        | 인스턴스                                                  |
|---------------|-------------------------------------------------------|
| Error         | 일반적 에러 객체                                             |
| SyntaxError   | 자바스크립트 문법에 맞지 않는 문을 해석할 때 발생하는 에러 객체                  |
| RefernceError | 참조할 수 없는 식별자를 참조했을 때 발생하는 에러 객체                       |
| TypeError     | 피연산자 또는 인수의 데이터 타입이 유효하지 않을 때 발생하는 에러 객체              |
| RangeError    | 숫자값의 허용 범위를 벗어났을 때 발생하는 에러 객체                         |
| URIError      | encodeURI 또는 decodeURI 함수에 부적절한 인수를 전달했을 때 발생하는 에러 객체 |
| EvalError     | eval 함수에서 발생하는 에러 객체                                  |

## throw 문

- 에러 객체 생성과 에러 발생은 의미가 다름

```jsx
try {
  // 에러 객체를 생성한다고 에러가 발생하는 것은 아님
  new Error('something wrong');
} catch (error) {
  console.log(error);
}
```

- try 코드 블록에서 throw 문으로 에러 객체를 던져야 함

```
throw 표현식;
```

- 어떤 값이라도 상관없지만 일반적으로 에러 객체를 지정
  - 에러를 던지면 catch 문의 에러 변수가 생성되고 던져진 에러 객체가 할당됨

    ```jsx
    try{
    	// 여러 객체를 던지면 catch 코드 블록이 실행되기 시작함
    	throw new Error('something wrong');
    } catch (error) {
    	console.log(error);
    }
    ```

### 예제

```jsx
// 외부에서 전달받은 콜백 함수를 n번만큼 반복 호출
const repeat = (n, f) => {
  if (typeof f !== 'function') throw new TypeError('f must be a function');

  for (var i = 0; i < n; i++) {
    f(i); // i를 전달하면서 f를 호출
  }
};

try {
  repeat(2, 1); // 두 번째 인수가 함수가 아니므로 TypeError가 발생함
} catch (err) {
  console.error(err); // TypeError: f must be a function
}
```

## 에러의 전파

- 에러는 호출자 방향(콜 스택의 아래 방향)으로 전파됨

```jsx
const foo = () => {
  throw Error('foo에서 발생한 에러'); // (4)
};

const bar = () => {
  foo(); // (3)
};

const baz = () => {
  bar(); // (2)
};

try {
  baz(); // (1)
} catch (err) {
  console.error(err);
}

```

1. `baz` 함수 호출
2. `bar` 함수 호출
3. `foo` 함수 호출
4. `foo` 함수가 에러를 throw 함

<img alt="javascript_error" src="/static/images/javascript_error.png" width="300"/>

- 에러를 캐치하지 않으면 `호출자 방향`으로 전파됨
  - `throw된 에러를 캐치`하여 적절히 대응하면 프로그램을 강제 종료시키지 않고 `코드의 실행 흐름을 복구`할 수 있음

> `비동기 함수`, `프로미스 후속 처리 메서드의 콜백 함수`는 호출자가 없으므로 `태스크 큐`나 `마이크로태스크 큐`에 일시 저장되었다가 콜 스택이 비면 `이벤트 루프`에 의해 콜 스택으로 푸시되어 실행되고,
> 이때 콜 스택의 푸시된 콜백 함수의 `실행 컨텍스트`는 `콜 스택에 가장 하부에 존재`하기 때문에 에러를 전파할 호출자가 존재하지 않음

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 800 ~ 808p