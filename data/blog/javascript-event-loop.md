---
title: 이벤트 루프
date: '2022-06-10'
tags: ['javascript']
draft: false
summary: 태스크가 들어오길 기다렸다가 태스크가 들어오면 처리하고, 없을 경우에는 끊임없이 돌아가는 자바스크립트 내 루프
layout: PostSimple
authors: ['default']
---

# Event Loop

## 정의

- 태스크가 들어오길 기다렸다가 `태스크가 들어오면 처리`하고, `없을 경우에는 끊임없이 돌아가는` 자바스크립트 내 루프
1. 처리해야할 태스크가 있는 경우
   - 먼저 들어온 태스크부터 순차적으로 처리
2. 처리해야 할 태스크가 없는 경우
   - 잠들어 있다가 새로운 태스크가 추가되면 다시 1로 돌아감

> `스크립트`나 `핸들러`, `이벤트`가 활성화 될때만 돌아감

## 대표적인 태스크

- 외부 스크립트 `<script src="…">가` 로드될 때, 해당 스크립트를 실행하는 것
- 사용자가 마우스를 움직일 때 `mousemove` 이벤트와 이벤트 핸들러를 실행하는 것
- `setTimeout`에서 설정한 시간이 다 된 경우, 콜백 함수를 실행
- 기타 등등

### 태스크 처리

- 자바스크립트 엔진은 태스크들을 차례대로 처리하고, `새로운 태스크가 추가될 때까지 기다림`
  - `태스크를 기다리는 동안`에는 CPU 자원 소비는 0에 가까워지고 `엔진은 잠듦`

## 매크로태스크 큐(macrotask queue)

- 자바스크립트 엔진이 바쁠 때 `새로운 태스크`가 추가되면 `큐에 추가`하는데, 이를 `매크로태스크 큐`라 지칭함

  <img alt="javascript_event_loop_1" src="/static/images/javascript_event_loop_1.png" width="500"/>

  - 엔진이 `script`를 처리하느라 바쁜데 사용자가 마우스를 움직여 `mousemove 이벤트`를 활성화하고, 이어서 `setTimeout`에서 설정한 시간이 지난 경우
    - 3개의 테스크는 큐에 하나씩 추가됨
    - 큐에 있는 태스크는 `들어간 순서대로` 처리됨
      1. script를 먼저 처리
      2. `mousemove` 이벤트와 핸들러 처리
      3. `setTimeout` 핸들러 처리

### 두 가지 세부 사항

1. 엔진이 특정 태스크를 처리하는 동안 `렌더링은 절대 일어나지 않음`
   - 태스크를 처리하는 시간이 길지 않으면 전혀 문제가 되지 않음

2. 태스크 처리에 긴 시간이 걸리면, 태스크를 처리하는 동안 발생한 사용자 이벤트 등의 새로운 태스크들을 처리하지 못함
   - 사용자는 `응답 없는 페이지`와 같은 얼럿을 마주할 수 있음
   - 브라우자는 얼럿 창을 통해 `프로그래밍 에러`에 관해 `태스크를 취할지 말지를 유도`함

## UseCase1 - CPU 소모가 많은 태스크 쪼개기

- `CPU 소모가 아주 많은 태스크`가 하나 있는 것을 가정
- 코드 일부를 강조하기 위해서는 어떤 부분을 강조해야 할지에 대한 사전 분석이 필요하고, 색을 변경한 요소와 새로운 요소들을 문서에 추가해야하는 일련의 작업
  - 강조해야 할 코드 양이 많다면 그 만큼 긴 시간이 소모됨
- 엔진이 바쁠 때엔 사용자 이벤트 처리나 DOM 관련 작업이 멈추므로 브라우저에 `지연`이 생기거나 `멈춤` 현상을 유발할 수 있음(Bad Case)
- 앞부분 `100줄만 먼저 강조`하고, `지연시간이 0인 setTimeout`을 사용해 새로 스케줄링한 후 반복 작업

### `리팩토링 전`

```jsx
let i = 0;

let start = Date.now();

function count() {

  for (let j = 0; j < 1e9; j++) {
    i++;
  }

  alert("처리에 걸린 시간: " + (Date.now() - start) + "ms");
}

count();
```

<img alt="javascript_event_loop_2.png" src="/static/images/javascript_event_loop_2.png" width="400"/>

- 해당 코드를 실행하면 자바스크립트 엔진이 해당 함수를 `처리할 때까지 몇 초간 멈춤`
- 숫자 카운팅이 끝나고 `얼럿 창이 뜨기 전까지`는 `사용자 이벤트는 처리되지 않음`

### `리팩토링 - setTimeout 호출`

```jsx
let i = 0;

let start = Date.now();

function count() {

  // (1) 무거운 작업 쪼개고 실행
  do {
    i++;
  } while (i % 1e6 != 0);

  if (i == 1e9) {
    alert("처리에 걸린 시간: " + (Date.now() - start) + "ms");
  } else {
    setTimeout(count); // (2) 새로운 호출 스케줄링
  }
}

count();
```

- `(1)`
  - do-while 반복에서 `count 태스크 일부가 처리`됨
- `(2)`
  - 카운팅이 다 끝나지 않았다면 `카운팅 태스크가 다시 스케줄링`됨

1. 첫 번째 부분 카운팅: `i=1…1000000`
2. 두 번째 부분 카운팅: `i=1000001..2000000`
3. 원하는 숫자를 다 셀때까지 부분 카운팅 진행

> 카운팅 실행 중간 중간에 `환기`를 통해 이벤트 루프가 돌아가게 하면 사용자 이벤트에 반응하면서 무거운 태스크 처리가 가능해짐

### `리팩토링 - 스케줄링 코드 함수 변경`

```jsx
let i = 0;

let start = Date.now();

function count() {
  if (i < 1e9 - 1e6) {
    setTimeout(count);
  }
  do {
    i++;
  } while (i % 1e6 != 0);

  if (i == 1e9) {
    alert("처리에 걸린 시간: " + (Date.now() - start) + "ms");
  }
}

count();
```

- 스케줄링 코드를 `함수 앞부분으로 옮김`으로서  `count()`가 호출되고 아직 원하는 숫자를 다 세지 못한 경우, 부분 카운팅이 시작되기 전에 `부분 카운팅 재스케줄링`이 이뤄짐
- 중첩 setTimeout 호출이 많은 경우에는 `브라우저 최소 대기 시간이 4밀초`가 됨
  - 코드상으론 대기 시간이 0이더라도 실제 대기시간은 4ms(`또는 그보다 조금 긴 시간`)가 됨
  - 숫자를 세기 전에 `스케줄링`하면 숫자를 세면서 `대기 시간을 소모`할 수 있어 `실행이 더 빨라짐`

## UseCase2 - 프로그레스 바

- 진행 상태를 나타내주는 `프로그레스 바`를 만들 떄에 큰 장점이 될 수 있음
- 브라우저는 현재 작업 중인 `태스크가 끝나야` DOM 변경분을 `화면에 렌더링` 해줌

```html
<div id="progress"></div>

<script>
  let i = 0;

  function count() {

    // 무거운 작업을 쪼갠 후 이를 수행
    do {
      i++;
      progress.innerHTML = i;
    } while (i % 1e3 != 0);

    if (i < 1e7) {
      setTimeout(count);
    }

  }

  count();
</script>
```

  - `setTimeout`을 사용해 태스크를 여러 개로 쪼개서 서브 태스크 중간마다 상태 변화를 볼 수 있음

## UseCase3 - 이벤트 처리가 끝난 이후 작업

- 이벤트 버블링이 끝나 모든 DOM 트리 레벨에서 이벤트 핸들링 될 때까지 `특정 액션을 연기`시켜야 하는 경우
  - 지연 시간이 0인 `setTimeout`으로 감싸면 원하는 동작을 구현할 수 있음

    ```jsx
    menu.onclick = function() {
      // ...
    
      // 클릭한 메뉴 내 항목 정보가 담긴 커스텀 이벤트 생성
      let customEvent = new CustomEvent("menu-open", {
        bubbles: true
      });
    
      // 비동기로 커스텀 이벤트를 디스패칭
      setTimeout(() => menu.dispatchEvent(customEvent));
    };
    ```

  - menu-open을 `setTimeout` 안에서 디스패치해서 사용하면 click 이벤트가 완전히 핸들링 된 후 `menu-open` 이벤트를 디스패칭할 수 있음

## 마이크로태스크

- 주로 프로미스를 사용해 만듦
  - `.then/catch/finally` 핸들러
  - `await` 문법
  - `queueMicrotask(func)`
    - 함수를 마이크로태스크 큐에 넣어 처리할 수 있음

> 매크로태스크 하나를 처리할 때마다 또 다른 매크로태스크나 렌더링 작업을 하기 전에 `마이크로태스크 큐에 쌓인 마이크로태스크 전부를 처리함`

```jsx
setTimeout(() => alert("timeout"));

Promise.resolve()
  .then(() => alert("promise"));

alert("code");
```

### 실행 순서

1. `code` - 일반적인 동기 호출이므로 가장 먼저 매크로태스크 큐에 들어간 후 실행
2. `promise` - `.then`은 마이크로태스크 큐에 들어가 처리되므로, 현재 코드(`alert("code")`)가 실행되고 난 후에 실행됨
3. `timeout` - `setTimeout`에서 설정한 시간이 끝난 후 콜백 함수를 실행하는 것은 `매크로태스크`이므로 `가장 마지막에 출력됨`

### 모식도

<img alt="javascript_event_loop_3" src="/static/images/javascript_event_loop_3.png" width="400"/>

- 마이크로태스크는 다른 이벤트 핸들러나 렌더링 작업, 혹은 `다른 매크로태스크가 실행되기 전에 처리`
- 마우스 좌표 변경이나 네트워크 통신에 의한 데이터 변경 같이 `애플리케이션 환경에 변화를 주는 작업에 영향을 받지 않고 모든 마이크로태스크를 동일한 환경에서 처리`할 수 있음

### queueMicrotask

- 현재 코드 실행이 끝난 후, 새로운 이벤트 핸들러가 처리되기 전이면서 렌더링이 실행되기 전에 `비동기적으로 실행`하는 경우 사용
  - 주로 `커스텀 함수를 스케줄링`하는데 사용

```jsx
let i = 0;

let start = Date.now();

function count() {

  do {
    i++;
    progress.innerHTML = i;
  } while (i % 1e3 != 0);

  if (i < 1e6) {
    queueMicrotask(count);
  }

}

count();
```

> **Referenced**

- [모던 JavaScript 튜토리얼](https://ko.javascript.info/event-loop)
