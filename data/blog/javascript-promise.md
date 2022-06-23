---
title: 프로미스
date: '2022-06-23'
tags: ['javascript']
draft: false
summary: 전통적인 콜백 패턴의 단점(콜백 헬)으로 인해 ES6에서 도입된 비동기 처리 패턴
layout: PostSimple
authors: ['default']
---

# 프로미스

- 전통적인 콜백 패턴의 단점(콜백 헬)으로 인해 `ES6`에서 도입된 `비동기 처리 패턴`
  - `가독성`, `에러 처리`, `여러 개의 비동기 처리`를 명확하게 표현할 수 있음

## 비동기 처리를 위한 콜백 패턴의 단점

### 비동기와 콜백 함수

- 비동기 함수를 호출하면 함수 내부의 비동기로 동작하는 코드가 완료되지 않았다 해도 기다리지 않고 즉시 종료됨
  - `비동기로 동작하는 코드`는 `함수가 종료된 이후에 완료`되므로, 처리 결과를 외부로 반환하거나 상위 스코프의 변수에 할당하면 기대하는 대로 동작하지 않음
- 비동기 함수는 **비동기 처리 결과를 외부에 반환할 수 없고**, **상위 스코프의 변수에 할당할 수도 없기 때문에** 비동기 함수의 처리 결과에 대한 `후속 처리는 비동기 함수 내부에서 수행`하는 `콜백 함수`
  를 전달하는 것이 일반적임

### 콜백 헬

- 콜백 함수를 통해 비동기 처리 결과에 대한 후속처리를 수행하는 비동기 함수가 결과를 가지고 또 다시 비동기 함수를 호출해야 한다면 콜`백 함수 중첩`으로 인해 `복잡도가 높아지는 현상`을 `콜백 헬`이라 함

```jsx
// GET 요청을 위한 비동기 함수
const get = (url, callback) => {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.send();

  xhr.onload = () => {
    if (xhr.status === 200) {
      // 서버의 응답을 콜백 함수에 인수로 전달하면서 호출하여 응답에 대한 후속 처리를 함
      callback(JSON.parse(xhr.response));
    } else {
      // 에러 정보를 콜백 함수에 인수로 전달하면서 호출하여 에러 처리를 함
      console.error(`${xhr.status} ${xhr.statusText}`);
    }
  };
};

const url = 'https://jsonplaceholder.typicode.com';
// id가 1인 post의 userId를 취득
get(`${url}/posts/1`, ({userId}) => {
  console.log(userId); // 1
  // post의 userId를 사용하여 user 정보를 취득
  get(`${url}/users/${userId}`, userInfo => {
    console.log(userInfo); // {id: 1, name: "Leanne Graham", username: "Bret",...}
  });
});
```

- GET 요청을 통해 서버로부터 응답을 취득하고 데이터를 사용하여 또다시 GET 요청을하여 `콜백 헬`을 발생

### 에러 처리의 한계(프로미스의 등장 배경)

```jsx
try {
  setTimeout(() => {
    throw new Error('Error!');
  }, 1000);
} catch (e) {
  console.error('캐치한 에러', e);
}
```

- try 코드 블록 내에서 호출한 함수는 1초 후에 콜백 함수가 실행되도록 타이머를 설정하고 콜백 함수는 에러를 발생시키지만 의도하지 않게 `catch 코드 블록에서 캐치되지 않음`
  1. setTimeout 함수의 실행 컨텍스트 생성 및 실행(`콜 스택에 푸시`)
  2. setTimeout은 `비동기 함수`이므로 콜백 함수가 호출되는 것을 기다리지 않고 즉시 종료(`콜 스택 팝`)
  3. `타이머가 만료`되면 setTimeout 함수의 콜백 함수는 `태스크 큐로 푸시`
  4. 콜 스택이 비어있다면 `이벤트 루프에 의해 콜 스택으로 푸시`
- 에러는 `호출자 방향으로 전파`되므로  `setTimeout` 함수의 콜백 함수가 발생시킨 에러는 catch 블록에서 캐치되지 않음
  - `호출자 방향`
    - 실행 중인 실행 컨텍스트가 푸시되기 직전에 푸시된 실행 컨텍스트 방향

> 비동기 처리를 위한 콜백 패턴의 문제점 중에서 가장 심각한 것은 `에러 처리`가 곤란하는 것

## 프로미스의 생성

- Promise 생성자 함수를 new 연산자와 함께 호출
- `ES6`에서 도입된 Promise는 ECMAScript 사양에 정의된 `표준 빌드인 객체`
- 비동기 처리를 수행할 `콜백 함수`를 인수로 전달받음
  - 콜백 함수는 `resolve`와 `reject` 함수를 인수로 전달받음

```jsx
// 프로미스 생성
const promise = new Promise((resolve, reject) => {
  // Promise 함수의 콜백 함수 내부에서 비동기 처리를 수행
  if (/* 비동기 처리 성공 */) {
    resolve('result');
  } else { /* 비동기 처리 실패 */
    reject('failure reason');
  }
});
```

- Promise 생성자 함수가 인수로 전달받은 `콜백 함수 내부`에서 `비동기 처리를 수행`
  - 비공기 처리 `성공`
    - `resolve` 함수 호출
  - 비동기 처리 `실패`
    - `reject` 함수 호출

### 프로미스 상태 정보

- 프로미스는 현재 비동기 처리가 어떻게 진행되고 있는지를 나타내는 `상태 정보`를 가짐

| 프로미스의 상태 정보 | 의미                    | 상태 변경 조건           |
|-------------|-----------------------|--------------------|
| pending     | 비동기 처리가 아직 수행되지 않은 상태 | 프로미스가 생성된 직후 기본 상태 |
| fulfilled   | 비동기 처리가 수행된 상태(성공)    | resolve 함수 호출      |
| rejected    | 비동기 처리가 수행된 상태(실패)    | reject 함수 호출       |

- 생성된 직후의 프로미스는 기본적으로 `pending` 상태
- 처리 결과에 따른 상태 변경
  - **비동기 처리 성공** - resolve 함수를 호출해 프로미스를 `fulfilled` 상태로 변경
  - **비동기 처리 실패** - reject 함수를 호출해 프로미스를 `rejected` 상태로 변경
- 프로미스의 상태는 `resolve` 또는 `reject` 함수를 호출하는 것으로 결정됨

  <img alt="javascript_promise" src="/static/images/javascript_promise.png" width="700"/>

- `fulfilled`, `reject` 상태를 `settled` 상태라고 함
  - 비동기 처리가 `수행된 상태`
  - 다른 상태로 변화할 수 없음
- 비동기 처리가 `실패`하면 프로미스는 `pending` 상태에서 `rejected` 상태로 변화함
  - 비동기 처리 결과는 Error 객체를 값으로 가짐

> `프로미스는` 비동기 처리 상태와 처리 결과를 관리하는 객체
>

## 프로미스의 후속 처리 메서드

- 프로미스의 비동기 처리 상태가 변화하면 후속 처리 메서드에 인수로 전달한 콜백 함수가 선택적으로 호출됨

### Promise.prototype.then

- `then` 메서드는 `두 개의 콜백 함수`를 인수로 전달받음
  - 첫 번째 콜백 함수
    - 비동기 처리가 `성공`했을 때
    - 프로미스가 `fulfilled` 상태
  - 두 번째 콜백 함수
    - 비동기 처리가 `실패`했을 때
    - 프로미스가 `rejected` 상태

```jsx
// fulfilled
new Promise(resolve => resolve('fulfilled'))
  .then(v => console.log(v), e => console.error(e)); // fulfilled

// rejected
new Promise((_, reject) => reject(new Error('rejected')))
  .then(v => console.log(v), e => console.error(2)); // Error: rejected
```

- `then` 메서드는 언제나 `프로미스를 반환함`
  - 콜백 함수가 프로미스가 아닌 값을 반환하면 그 값을 암묵적으로 `resolve` 또는 `reject`하여 프로미스를 생성해 반환함

### Promise.prototype.catch

- `catch` 메서드는 `한 개의 콜백 함수`를 인수로 전달받음
- 프로미스가 `rejected` 상태인 경우에만 호출

```jsx
// rejected
new Promise((_, reject) => reject(new Error('rejected')))
  .catch(e => console.log(e)); // Error: rejected
```

- then(undefined, onRejected) 메서드와 동일하게 동작함
- then 메서드와 동일하게 언제나 `프로미스를 반환`

### Promise.prototype.finally

- `finally` 메서드는 `한 개의 콜백 함수`를 인수를 전달받음
- 프로미스의 성공 또는 실패와 상관없이 `무조건 한 번 호출`됨

```jsx
new Promise(() => {
})
  .finally(() => console.log('finally')); // finally
```

## 프로미스 체이닝

- then , catch, finally 후속 처리 메서드는 언제나 프로미스를 반환하므로 연속적으로 호출(chaining)할 수 있음

```jsx
const url = 'https://jsonplaceholder.typicode.com';

// id가 1인 post의 userId를 취득
promiseGet(`${url}/posts/1`)
  // 취득한 post의 userId로 user 정보를 취득
  .then(({userId}) => promiseGet(`${url}/users/${userId}`))
  .then(userInfo => console.log(userInfo))
  .catch(err => console.error(err));
```

| 후속 처리 메서드 | 콜백 함수의 인수                                                      | 후속 처리 메서드의 반환값                         |
|-----------|----------------------------------------------------------------|----------------------------------------|
| then      | promiseGet 함수가 반환한 프로미스가 resolve 한 값(id가 1인 post)              | 콜백 함수가 반환한 프로미스                        |
| then      | 첫 번째 then 메서드가 반환한 프로미스가 resolve한 값(post의 userId로 취득한 user 정보) | 콜백 함수가 반환한 값(undefined)을 resolve한 프로미스 |
| catch     | promiseGet 함수 또는 앞선 후속 처리 메서드가 반환한 프로미스가 reject한 값             | 콜백 함수가 반환한 값(undefined)을 resolve한 프로미스 |

- `프로미스 체이닝`을 통해 비동기 처리 결과를 전달받아 후속 처리를 하므로 콜백 헬이 발생하지 않음
  - 그렇지만, 프로미스도 `콜백 패턴`을 사용하므로 `콜백 함수`를 사용함
- 콜백 패턴은 가독성이 좋지 않으므로 `ES8`에서 도입된 프로미스 기반 `async/await`를 통해 해결할 수 있음

## 프로미스의 정적 메서드

### Promise.resolve / Promise.reject

- `Promise.resolve`

    ```jsx
    // 배열을 resolve하는 프로미스를 생성
    const resolvedPromise = Promise.resolve([1, 2, 3]);
    resolvedPromise.then(console.log); // [1, 2, 3]
    ```

- `Promise.reject`

    ```jsx
    // 에러 객체를 reject하는 프로미스를 생성
    const rejectedPromise = Promise.reject(new Error('Error!'));
    rejectedPromise.catch(console.log); // Error: Error!
    ```

### Promise.all

- 여러 개의 비동기 처리를 모두 병렬 처리할 때 사용

    ```jsx
    const requestData1 = () =>
    	new Promise(resolve => setTimeout(() => resolve(1), 3000));
    const requestData2 = () =>
    	new Promise(resolve => setTimeout(() => resolve(2), 2000));
    const requestData3 = () =>
    	new Promise(resolve => setTimeout(() => resolve(3), 1000));
    
    // 세 개의 비동기 처리를 순차적으로 처리
    Promise.all([requestData1(), requestData2(), requestData3()])
    	.then(console.log) // [ 1, 2, 3 ] => 약 3초 소요
    	.catch(console.error);
    ```

  - 첫 번째 프로미스는 3초 후 1을 resolve함
  - 두 번째 프로미스는 2초 후 2을 resolve함
  - 세 번째 프로미스는 1초 후 3을 resolve함
- 프로미스를 요소로 갖는 배열 등의 `이터러블을 인수로 전달`받음
- `fulfilled` 상태가 되면 모든 처리 결과를 `배열에 저장해 새로운 프로미스를 반환함`
  - 처리 순서가 보장됨

### Promise.race

- `Promise.all`과는 다르게 가장 먼저 `fulfilled` 상태가 된 프로미스의 처리 결과를 `resolve`하는 새로운 프로미스를 반환함

```jsx
Promise.race([
  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
  new Promise(resolve => setTimeout(() => resolve(3), 1000)), // 3
])
  .then(console.log) // 3
  .catch(console.log);
```

- 프로미스가 `rejected` 상태가 되면 `Promise.all` 메서드와 동이하게 처리됨

### Promise.allSettled

- `ES11`(ECMAScript 2020)에 도입됨
- 프로미스를 `요소로 갖는 배열` 등의 `이터러블`을 인수로 전달받음
- 전달 받은 프로미스가 모든 `settled` 상태가 되면 처리 결과를 배열로 반환

```jsx
Promise.allSettled([
  new Promise(resolve => setTimeout(() => resolve(1), 2000)),
  new Promise((_, reject) => setTimeout(() => reject(new Error('Error!')), 1000)) // 2
]).then(console.log);
/*
[
	{status: "fulfilled", value: 1},
	{status: "rejected", reason: Error! at <anonymouse>:3:54}
]
*/
```

- 프로미스가 `fulfilled` 상태인 경우 비동기 처리 상태를 나타내는 status 프로퍼티와 `처리 결과`를 나타내는 `value` 프로퍼티를 가짐
- 프로미스가 `rejected` 상태인 경우 비동기 처리 상태를 나타내는 status 프로퍼티와 `에러`를 나타내는 `reason` 프로퍼티를 가짐

```text
[
  // 프로미스가 fulfilled 상태인 경우
  {status: "fulfilled", value: 1},
  // 프로미스가 rejected 상태인 경우
  {status: "reject", reason: Error:Error!at < anonymous >:3:60}
]
```

## fetch

- HTTP 요청 전송 기능을 제공하는 `클라이언트 사이드 Web API`
- 콜백 패턴의 단점에서 자유로움
- HTTP `요청`을 전송한 URL과 HTTP 요청 메서드, HTTP 요청 헤더, 페이로등 등을 설정한 객체를 전달

    ```jsx
    const promise = fetch(url, [,options])
    ```

- `HTTP 응답`을 나타내는 `Response` 객체를 래핑한 `Promise` 객체를 반환
  - 후속 처리 메서드 `then`을 통해 프로미스가 `resolve`한 `Response` 객체를 전달받을 수 있음

```jsx
fetch('https://jsonplaceholder.typicode.com/todos/1')
  // response는 HTTP 응답을 나타내는 Response 객체
  // json 메서드를 사용하여 Response 객체에서 HTTP 응답 몸체를 취득하여 역질렬화함
  .then(response => response.json())
  // json은 역직렬화된 HTTP 응답 몸체
  .then(json => console.log(json));
// {userId: 1, id: 1, title: "delectus aut autem", completed: false}
```

### fetch Error 처리

- fetch 함수가 반환하는 프로미스는 404, 500 같은 HTTP 에러가 발생해도 에러를 reject하지 않고 불리언 타입의 `ok 상태를 false`로 설정한 `Response` 객체를 `resolve` 함
- 오프라인 등의 `네트워크 장애`나 `CORS 에러`에 의해 요청이 완료되지 못한 경우에만 프로미스를 `reject`함

```jsx
const wrongUrl = 'https://jsonplaceholder.typicode.com/xxx/1';

// 부적절한 URL이 지정되었으므로 404 Not Found 에러가 발생함
fetch(wrongUrl)
  // response는 HTTP 응답을 나타내는 Response 객체
  .then(response => {
    if (!response.ok) throw new Error(response.statusText);
    return response.json();
  })
  .then(todo => console.log(todo))
  .catch(err => console.error(err));
```

- `fetch` 함수가 반환한 프로미스가 `resolve`한 불리언 타입의 `ok 상태`를 확인해 명시적으로 에러를 처리할 필요가 있음
- 참고로 `axios`는 모든 HTTP 에러를 `reject`하는 프로미스를 반환함
  - 또한 axios는 `인터셉터`, `요청 설정` 등 fetch 보다 다양한 기능을 지원함

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 842 ~ 867p