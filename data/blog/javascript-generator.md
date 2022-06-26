---
title: 제너레이터와 async/await
date: '2022-06-26'
tags: ['javascript']
draft: false
summary: ES6에서 도입된 제너레이터는 코드 블록의 실행을 일시 중지했다가 필요한 시점에 재개할 수 있는 특수한 함수
layout: PostSimple
authors: ['default']
---

# 제너레이터와 async/await

## 제너레이터란?

- ES6에서 도입된 제너레이터는 **코드 블록의 실행을 일시 중지했다가 필요한 시점에 재개할 수 있는 특수한 함수**

### 제너레이터 특징

1. 제너레이터 함수는 `함수 호출자`에게 `함수 실행의 제어권`을 양도할 수 있음
   - 일반 함수
     - 함수 호출자는 함수를 호출한 이후 함수 실행을 제어할 수 없음
   - `제너레이터 함수`
     - 함수 실행을 `함수 호출자가 제어할 수 있음`
     - `함수의 제어권`을 함수가 독점하는 것이 아니라 `함수 호출자에게 양도`할 수 있음

2. 제너레이터 함수는 `함수 호출자`와 `함수의 상태`를 주고받을 수 있음
   - 일반 함수
     - 함수가 실행되고 있는 동안에는 함수 외부에서 함수 내부로 값을 전달하여 함수의 상태를 변경할 수 없음
   - `제너레이터 함수`
     - 함수 호출자에게 `상태를 전달`할 수 있고 함수 호출자로부터 `상태를 전달받을 수 있음`

3. 제너레이터 함수를 호출하면 `제너레이터 객체`를 반환
   - 일반 함수
     - 함수 코드를 일괄 실행하고 값을 반환
   - `제너레이터 함수`
     - 함수 코드를 실행하는 것이 아니라 이터러블이면서 동시에 이터레이터인 제너레이터 객체를 반환

## 제너레이터 함수의 정의

- `function*` 키워드로 선언
- 하나 이상의 `yield 표현식` 포함

```jsx
// 제너레이터 함수 선언문
function* genDecFunc() {
  yield 1;
}

// 제너레이터 함수 표현식
const getExpFunc = function* () {
  yield 1;
};

// 제너레이터 메서드
const obj = {
  * getnObjMethod() {
    yield 1;
  }
};

// 제너레이터 클래스 메서드
class MyClass {
  * getClsMethod() {
    yield 1;
  }
};
```

- 에스터리스크(*)의 `위치`는 function `키워드`와 `함수 이름 사이`라면 언제든지 상관없음
- 일관성을유지하기 위해 `function` 키워드 `바로 뒤에 붙임`
- `화살표 함수`로 정의하거나 `new 연산자`와 함께 생성자 함수로 호출이 `불가능`

## 제너레이트 객체

- 제너레이터 함수를 호출하면 `제너레이터 객체를 생성해 반환함`
  - `Symbol.iterator` 메서드를 상속받는 `이터러블`
  - `value`, `done` 프로퍼티를 갖는 이터레이터 리절트 객체를 반환하는 `next 메서드를 소유`하는 이터레이터

### 이터레이터 리절트 객체 반환

| 구분         | value 프로퍼티                     | done 프로퍼티 |
|------------|--------------------------------|-----------|
| next 메서드   | yield된 값                       | false     |
| return 메서드 | 인수로 전달받은 값                     | true      |
| throw 메서드  | 인수로 전달받은 에러를 발생시키고 undefined 값 | true      |

```jsx
function* getFunc() {
  try {
    yield 1;
    yield 2;
    yield 3;
  } catch (e) {
    console.error(2);
  }
}

const generator = genFunc();

console.log(generator.next()); // {value: 1, done: false}
console.log(generator.return('End!')); // {value: "End!", done: true}
console.log(generator.throw('Error!')); // {value: undefined, done: true}
```

## 제너레이터의 일시 중지와 재개

- 제너레이터는 `yield` 키워드와 `next` 메서드를 통해 실행을 `일시 중지`했다가 필요한 시점에 `다시 재개`할 수 있음
- **yield 표현식까지만 실행하므로 함수의 실행을 일시 중지시키거나 yield 키워드 뒤에 오는 표현식의 평가 결과를 제너레이터 함수 호출자에게 반환**

```jsx
// 제너레이터 함수
function* genFunc() {
  yield 1;
  yield 2;
  yield 3;
}

// 제너레이터 함수를 호출하면 제너레이터 객체를 반환함
// 이터러블이면서 동시에 이터레이터인 제네레이터 객체는 next 메서드를 가짐
const generator = genFunc();

// 처음 next 메서드를 호출하면 첫 번째 yield 표현식까지 실행되고 일시 중지됨
// next 메서드는 이터레이터 리절트 객체({value, done})를 반환
// value 프로퍼티에는 첫 번째 yield 표현식에서 yield된 값 1이 할당됨
// done 프로퍼티에는 제너레이터 함수가 끝까지 실행되었는지를 나타내는 false가 할당됨
console.log(generator.next()); // {value: 1, done: false}

// 처음 next 메서드를 호출하면 첫 번째 yield 표현식까지 실행되고 일시 중지됨
// next 메서드는 이터레이터 리절트 객체({value, done})를 반환
// value 프로퍼티에는 첫 번째 yield 표현식에서 yield된 값 2이 할당됨
// done 프로퍼티에는 제너레이터 함수가 끝까지 실행되었는지를 나타내는 false가 할당됨
console.log(generator.next()); // {value: 2, done: false}

// 처음 next 메서드를 호출하면 첫 번째 yield 표현식까지 실행되고 일시 중지됨
// next 메서드는 이터레이터 리절트 객체({value, done})를 반환
// value 프로퍼티에는 첫 번째 yield 표현식에서 yield된 값 3이 할당됨
// done 프로퍼티에는 제너레이터 함수가 끝까지 실행되었는지를 나타내는 false가 할당됨
console.log(generator.next()); // {value: 3, done: false}

// 처음 next 메서드를 호출하면 첫 번째 yield 표현식까지 실행되고 일시 중지됨
// next 메서드는 이터레이터 리절트 객체({value, done})를 반환
// value 프로퍼티에는 제너레이터 함수의 반환값 undefined가 할당됨
// done 프로퍼티에는 제너레이터 함수가 끝까지 실행되었음을 나타내는 true가 할당됨
console.log(generator.next()); // {value: undefined, done: true}
```

```
generator.next() -> yield -> generator.next() -> yield -> ... -> generator.next() -> return
```

- 제너레이터 객체의 `next 메서드`에 전달한 `인수`는 제너레이터 함수의 `yield 표현식`을 할당받는 `변수에 할당됨`

## 제너레이터의 활용

### 이터러블의 구현

- 이터레이션 프로토콜을 준수해 이터러블을 생성하는 방식보다 더 간단히 `이터러블`을 구현할 수 있음

```jsx
// 무한 이터러블을 생성하는 제너레이터 함수
const infiniteFibonacci = (function* () {
  let [pre, cur] = [0, 1];

  while (true) {
    [pre, cur] = [cur, pre + cur];
    yield cur;
  }
}());

// infiniteFibonacci는 무한 이터러블
for (const num of infiniteFibonacci) {
  if (num > 10000) break;
  console.log(num); // 1 2 3 5 8 ... 2584 4181 6765
}
```

### 비동기 처리

- `next` 메서드와 `yield` 표현식을 통해 함수 호출자와 함수의 `상태`를 주고받을 수 있기때문에 `프로미스`를 사용한 비동기 처리를 `동기 처리`처럼 구현할 수 있음
  - 프로미스의 후속 처리 메서드 `then`/`catch`/`finally` 없이 `비동기 처리 결과를 반환`하도록 구현할 수 있음
- `제너레이터 실행기`(`co` 라이브러리 사용)

    ```jsx
    const fetch = require('node-fetch');
    // https://github.com/tj/co
    const co = require('co');
    
    co(function* fetchTodo() {
    	const url = 'https://jsonplaceholder.typicode.com/todos/1';
    	
    	const response = yield fetch(url);
    	const todo = yield response.json();
    	console.log(todo);
    	// { userId: 1, id: 1, title: 'delectus aut autem', completed: false }
    });
    ```

## async/await

- `ES8`에서는 제너레이터보다 `간단`하고 `가독성` 좋게 비동기 처리를 `동기처럼 동작하도록 구현`할 수 있는 `async`/`await`가 도입됨
- 프로미스를 기반으로 동작하므로 `then`/`catch`/`finally` 후속 처리 메서드에 `콜백 함수를 전달`하여 동기처럼 프로미스를 사용할 수 있음

    ```jsx
    const fetch = require('node-fetch');
    
    async function fetchTodo(){
    	const url = 'https://jsonplaceholder.typicode.com/todos/1';
    	
    	const response = await fetch(url);
    	const todo = await response.json();
    	console.log(todo);
    	// {userId: 1, id: 1, title: 'delectus aut autem', completed: false}
    }
    
    fetchTodo();
    ```

### async 함수

- `await` 키워드는 반드시 `async 함수 내부`에서 사용해야 함
- `async` 함수는 암묵적으로 반환값을 `resolve`하는 프로미스를 반환

    ```jsx
    // async 함수 선언문
    async function foo(n) { return n; }
    foo(1).then(v => console.log(v)); // 1
    
    // async 함수 표현식
    const bar = async function (n) { return n; };
    bar(2).then(v => console.log(v)); // 2
    
    // async 화살표 함수
    const baz = async n => n;
    baz(3).then(v => console.log(v)); // 3
    
    // async 메서드
    const obj = {
    	async foo(n) { return n; }
    };
    obj.foo(4).then(v => console.log(v)); // 4
    
    // async 클래스 메서드
    class MyClass {
    	async bar(n) { return n; }
    }
    const myClass = new MyClass();
    myClass.bar(5).then(v => console.log(v)); // 5
    ```

### await 키워드

- 프로미스가 `settled` 상태가 될 때까지 대기하다가 `settled` 상태가 되면 프로미스가 `resolve`한 처리 결과를 반환함

```jsx
const fetch = require('node-fetch');

const getGithubUserName = async id => {
  const res = await fetch(`https://api.github.com/users/${id}`); // (1)
  const {name} = await res.json(); // (2)
  console.log(name); // Ungmo Lee
};

getGithubUserName('ungomo2');
```

- (1)의 fetch 함수가 수행한 HTTP 요청에 대한 서버의 응답이 도착해서 fetch 함수가 반환한 프로미스가 `settled` 상태가 될때까지 (1)은 대기함
  - **프로미스가 settled 상태가 되면 프로미스가 resolve한 처리 결과가 res 변수에 할당됨**

> 다음 실행을 `일시 중지`시켰다가 프로미스가 `settled` 상태가 되면 다시 재개

### 에러 처리

- 콜백 패턴의 단점 중 가장 심각한 것은 `에러 처리`가 곤란하는 것
- `호출자 방향(caller)`으로 전파
  - 비동기 함수의 콜백 함수를 호출한 것은 비동기 함수가 아니므로 `try…catch` 문을 사용해 에러를 캐치할 수 없음
- `async/await`에서 에러 처리는 `try…catch` 문을 사용할 수 있음
  - `try…catch`문을 사용한 에러 처리

      ```jsx
      const fetch = require('node-fetch');
      
      const foo = async () => {
        try{
          const wrongUrl = 'https://wrong.url';
          
          const response = await fetch(wrongUrl);
          const data = await response.json();
          console.log(data);
        } catch (err) {
          console.log(err); // TypeError: Failed to fetch
        }
      };
      
      foo();
      ```

    - **async 함수 내에서 `catch 문을 사용`해서 에러 처리를 하지 않으면 async 함수는 발생한 에러를 reject하는 프로미스를 반환함**
  - `Promise.prototype.catch` 후속 처리 메서드 사용

      ```jsx
      const fetch = require('node-fetch');
      
      const foo = async () => {
        const wrongUrl = 'https://wrong.url';
        
        const response = await fetch(wrongUrl);
        const data = await response.json();
        return data;
      };
      
      foo()
        .then(console.log)
        .catch(console.error); // Typeerror: Failed to fetch
      ```
    
> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 870 ~ 885p