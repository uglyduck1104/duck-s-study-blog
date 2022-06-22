---
title: 타이머
date: '2022-06-22'
tags: ['javascript']
draft: false
summary: 함수를 명시적으로 호출하지 않고 일정 시간이 경과된 이후에 호출되도록 함수 호출을 예약하려면 타이머 함수를 사용하는데 이를 호출 스케줄링이라 함
layout: PostSimple
authors: ['default']
---

# 타이머

## 호출 스케줄링

- 함수를 명시적으로 호출하지 않고 `일정 시간이 경과된 이후에 호출`되도록 함수 호출을 예약하려면 타이머 함수를 사용하는데 이를 `호출 스케줄링`이라 함
- `setTimeout`, `setInterver`
  - 타이머를 `생성`할 수 있는 함수
  - 일정 시간이 경과된 후 `콜백 함수`가 호출되도록 타이머를 생성함
  - `setTimeout`
    - 함수의 콜백 함수는 타이머가 만료되면 `단 한 번` 호출
  - `setInterver`
    - 함수의 콜백 함수는 타이머가 만료될 때마다 `반복` 호출
- `clearTimeout`, `clearInterver`
  - 타이머를 `제거`할 수 있는 함수
- 브라우저 환경과 Node.js 환경에서 모두 전역 객체의 메서드로서 타이머 함수를 제공(`호스트 객체`)

> 자바스크립트 엔진은 `싱글 스레드`로 동작하므로 타이머 함수는 `비동기 처리 방식`으로 동작

## 타이머 함수

### `setTimeout` / `clearTimout`

- 두 번째 인수로 전달받은 시간으로 `단 한 번 동작`하는 타이머를 생성
- `타이머가 만료`되면 첫 번째 인수로 전달받은 `콜백 함수가 호출`됨
  - 두 번째 인수로 전달받은 시간 이후 단 한 번 실행되도록 호출 스케줄링됨

```text
const timeoutId = setTimeout(func | code[, delay, param1, param2, ...]);
```

### 매개 변수

- `func`
  - 타이머가 만료된 뒤 호출될 `콜백 함수`
    - 코드를 문자열로 전달할 수 있음
- `delay`
  - 타이머 만료 시간(`ms` 단위)으로서, 인수 전달을 생략한 경우 기본값 0이 지정됨
  - delay 시간이 설정된 타이머가 만료되면 콜백 함수가 즉시 호출되는 것은 보장되지 않음
    - `태스크 큐`에 콜백 함수를 등록하는 시간을 지연할 뿐임
- `param1`, `param2`, `…`
  - 호출 스케줄링된 콜백 함수에 전달해야 할 인수가 존재하는 경우 `세 번째 이후의 인수로 전달`할 수 있음

### 예시

```jsx
// 1초(1000ms) 후 타이머가 만료되면 콜백 함수가 호출됨.
setTimeout(() => console.log('Hi!'), 1000);

// 1초(1000ms) 후 타이머가 만료되면 콜백 함수가 호출됨
// 콜백 함수에 'Lee'가 인수로 전달됨
setTimeout(name => console.log(`Hi ${name}.`), 1000, 'Lee');

// 두 번째 인수(delay)를 생략하면 기본값 0이 지정됨
setTimeout(() => console.log('Hello'));

// setTimeout 함수는 생성된 타이머를 식별할 수 있는 고유한 타이머 id를 반환
const timerId = setTimeout(() => console.log('Hello'));

// 타이머가 취소되면 setTimeout 함수의 콜백 함수가 실행되지 않음
clearTimeout(timerId);
```

- `setTimeout` 함수는 생성된 타이머를 식별할 수 있는 고유한 `타이머 id`를 반환
  - 브라우저 환경 → `숫자`
  - Node.js 환경 → `객체`
- `setTimeout` 함수가 반환한 타이머 id를 `clearTimeout` 함수의 인수로 전달하여 타이머를 취소할 수 있음

### `setInterval` / `clearInterval`

- 두 번째 인수로 전달받은 시간(ms, 1/1000초)으로 `반복 동작`하는 타이머를 생성
  - 타이머가 만료될 때마다 첫 번째 인수로 전달받은 콜백 함수가 `반복 호출`됨
  - 타이머가 취소될 때까지 계속됨

```text
const timerId = setInterval(func | code[, delay, param1, param2, ...]);
```

- setTimeout 함수와 마찬가지로 `setInterval` 함수는 생성된 타이머를 식별할 수 있는 고유한 `타이머 id`를 반환
  - 브라우저 환경 → `숫자`
  - Node.js 환경 → `객체`
- `setInterval` 함수가 반환한 타이머 id를 `clearInterval` 함수의 인수로 전달하여 타이머를 취소할 수 있음

```jsx
let count = 1;

// 1초(1000ms) 후 타이머가 만료될 때마다 콜백 함수가 호출됨
// setInterval 함수는 생성도니 타이머를 식별할 수 있는 고유한 타이머 id를 반환
const timeoutId = setInterval(() => {
  console.log(count); // 1 2 3 4 5
  // count가 5이면 setInterval 함수가 반환한 타이머 id를 clearInterval 함수의 인수로 전달하여
  // 타이머를 취소함. 타이머가 취소되면 setInterval 함수와 콜백 함수가 실행되지 않음
  if (count++ === 5) clearInterval(timeoutId);
}, 1000);
```

## 디바운스와 스로틀

- scroll, resize, input, mousemove 같은 이벤트는 짧은 시간 간격으로 연속해서 발생하므로 과도하게 호출되어 성능에 문제를 일으킬 가능성이 높음
- `디바운스`와 `스로틀`은 짧은 시간 간격으로 연속해서 발생하는 이벤트를 그룹화해서 과도한 이벤트 핸들러의 호출을 방지하는 프로그래밍 기법

### 예시

```jsx
// HTML
/*
<button>click me</button>
<pre>일반 클릭 이벤트 카운터 <span class="normal-msg">0</span></pre>
<pre>디바운스 클릭 이벤트 카운터 <span class="debounce-msg">0</span></pre>
<pre>스로틀 클릭 이벤트 카운터 <span class="throttle-msg">0</span></pre>
*/

// Javascript
const $button = document.querySelector('.button');
const $normalMsg = document.querySelector('.normal-msg');
const $debounceMsg = document.querySelector('.debounce-msg');
const $throttleMsg = document.querySelector('.throttle-msg');

const debounce = (callback, delay) => {
  let timerId;
  return event => {
    if (timerId) clearTimeout(timerId);
    timerId = setTimeout(callback, delay, event);
  };
};

const throttle = (callback, delay) => {
  let timerId;
  return event => {
    if (timerId) return;
    timerId = setTimeout(() => {
      callback(event);
      timerId = null;
    }, delay, event);
  };
};

$button.addEventListener('click', () => {
  $normalMsg.textContent = +$normalMsg.textContent + 1;
});

$button.addEventListener('click', debounce(() => {
  $debounceMsg.textContent = +$debounceMsg.textContent + 1;
}, 500));

$button.addEventListener('click', throttle(() => {
  $throttleMsg.textContent = +$throttleMsg.textContent + 1;
}, 500));
```

### 디바운스

- 짧은 시간 간격으로 이벤트가 연속해서 발생하면 이벤트 핸들러를 호출하지 않다가 `일정 시간이 경과한 이후에 이벤트 핸들러가 한 번만 호출`되도록 함
  - 디바운스는 짧은 시간 간격으로 발생하는 이벤트를 그룹화해서 `마지막에 한 번만 이벤트 핸들러가 호출`되도록 함

```jsx
// HTML
// <input type="text"/>
// <div class="msg"></div>

// Javascript
const $input = document.querySelector('input');
const $msg = document.querySelector('.msg');

const debounce = (callback, delay) => {
  let timeId;
  // debounce 함수는 timeId를 기억하는 클로저를 반환
  return event => {
    // delay가 경과하기 이전에 이벤트가 발생하면 이전 타이머를 취소하고 새로운 타이머를 재설정
    // 따라서 delay보다 짧은 간격으로 이벤트가 발생하면 callback은 호출되지 않음
    if (timerId) clearTimeout(timerId);
    timerId = setTimeout(callback, delay, event);
  };
};

// debounce 함수가 반환하는 클로저가 이벤트 핸들러로 등록됨
// 300ms보다 짧은 간격으로 input 이벤트가 발생하면 debounce 함수의 콜백 함수는
// 호출되지 않다가 300ms 동안 input 이벤트가 더 이상 발생하지 않으면 한 번만 호출됨
$input.oninput = debounce(e => {
  $msg.textContent = e.target.value;
}, 300);
```

- 일정 시간 동안 텍스트 입력 필드에 값을 입력하지 않으면 입력이 완료된 것으로 간주
  - debounce 함수에 두 번째 인수로 `전달한 시간(delay)보다 짧은 간격으로 이벤트가 발생`하면 이전 타이머를 취소하고 `새로운 타이머를 재설정`

![javascript_timer1.png](/static/images/javascript_timer1.png)

- 만약 input의 이벤트가 사용자가 텍스트 입력 필드에 값을 `입력할 때마다 연속해서 발생`하는데 입력한 값으로 Ajax 요청과 같은 처리를 수행한다면 사용자가 아직 입력을 완료하지
  않았어도 `Ajax 요청이 전송`됨
  - 서버에 부담을 주는 불필요한 처리이므로 사용자가 입력을 완료했을 때 `한 번만 Ajax 요청`하는 것이 바람직함
- `resize` 이벤트 처리나 `input` 요소에 입력된 값으로 `ajax 요청`하는 입력 필드 `자동완성 UI 구현`, `버튼 중복 클릭 방지` 처리 등에 유용하게 사용됨
  - 실무에서는 Underscore의 debounce 함수나 Lodash의 debounce 함수 사용을 권장함

### 스로틀

- 짧은 시간 간격으로 이벤트가 연속해서 발생하더라도 일정 시간 간격으로 `이벤트 핸들러가 최대 한 번만 호출`되도록 함

```jsx
// HTML
/*
<div class="container">
  <div class="content"></div>
</div>
<div>
  일반 이벤트 핸들러가 scroll 이벤트를 처리한 횟수:
  <span class="normal-count">0</span>
</div>
<div>
  스로틀 이벤트 핸들러가 scroll 이벤트를 처리한 횟수:
  <span class="throttle-count">0</span>
</div>
 */

// Javascript
const $container = document.querySelector('.container');
const $normalCount = document.querySelector('.normal-count');
const $throttleCount = document.querySelector('.throttle-count');

const throttle = (callback, delay) => {
  let timerId;
  // throttle 함수는 timerId를 기억하는 클로저를 반환
  return event => {
    // delay가 경과하기 이전에 이벤트가 발생하면 아무것도 하지 않다가
    // delay가 경과했을 때 이벤트가 발생하면 새로운 타이머를 재설정
    // 따라서 delay 간격으로 callback이 호출됨
    if (timerId) return;
    timerid = setTimeout(() => {
      callback(event);
      timerId = null;
    }, delay, event);
  };
};

let normalCount = 0;
$countainer.addEventListener('scroll', () => {
  $normalCount.textContent = ++normalCount;
});

let throttleCount = 0;
// throttle 함수가 반환하는 클로저가 이벤트 핸들러로 등록됨
$countainer.addEventListener('scroll', throttle(() => {
  $throttleCount.textContent = ++throttleCount;
}, 100));
```

- 짧은 시간 간격으로 연속해서 발생하는 이벤트의 `과도한 이벤트 핸들러의 호출을 방지`하기 위해 throttle 함수는 이벤트를 그룹화해서 `일정 시간 단위`로 이벤트 핸들러가 호출되도록 호출 주기를 만듦
- throttle 함수가 반환한 함수는 throttle 함수에 두 번째 인수로 전달한 시간이 경과하기 이전에 이벤트가 발생하면 아무것도 하지
  않다가 `delay 시간이 경과했을 때, 이벤트가 발생하면 콜백 함수를 호출`하고 새로운 타이머를 재설정함

  ![javascript_timer2.png](/static/images/javascript_timer2.png)

- `scroll 이벤트` 처리나 `무한 스크롤 UI 구현` 등에 유용하게 사용됨
  - 실무에서는 Underscore의 throttle 함수나 Lodash의 throttle 함수 사용 권장

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 800 ~ 808p