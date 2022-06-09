---
title: 버블링과 캡처링
date: '2022-06-09'
tags: ['javascript']
draft: false
summary:
layout: PostSimple
authors: ['default']
---

# 버블링과 캡처링

## 버블링

- 한 요소에 이벤트가 발생하면 부모요소의 핸들러가 동작하고 `최상단의 조상 요소를 만날 때까지 반복`하면서 요소 `각각에 할당된 핸들러가 동작`

```html
<style>
  body * {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<form onclick="alert('form')">FORM
  <div onclick="alert('div')">DIV
    <p onclick="alert('p')">P</p>
  </div>
</form>
```

- 버블링 동작 방식으로 인해 `<p>` 요소를 클릭하면 `p` → `div` → `form` 순서로 3개의 alert 창이 뜸

<img alt="javascript_capturing_bubbling_1" src="/static/images/javascript_capturing_bubbling_1.png" width="400"/>

1. `<p>`에 할당된 `onclick 핸들러`가 동작
2. 부모 요소 `<div>`에 할당된 핸들러 동작
3. 부모 요소 `<form>`에 할당된 핸들러 동작
4. `document` 객체에 도달할 때까지, 각 요소에 할당된 `onclick` 핸들러 동작

> 해당 흐름을 `이벤트 버블링`이라 부르고, 이벤트가 발생하는 최하위 요소에서 최상위 요소로 올라가며 발생

### event.target

- 부모 요소의 핸들러는 이벤트가 정확히 어디서 발생했는지에 대한 자세한 정보를 얻을 수 있음
- 가장 안쪽의 요소는 타겟(target) 요소라 부르고, `event.target`을 통해 접근 가능함
- `event.currentTarget`과의 차이점(⭐⭐⭐⭐⭐)
  - event.target은 `실제 이벤트가 시작된 요소`로서, 버블링이 진행되도 변하지 않음
  - 반면에 this(`event.currentTarget`)는 `현재`요소로, 실행 중인 `핸들러가 할당된 요소를 참조`함

```html
<form id="form">FORM
  <div>DIV
    <p>P</p>
  </div>
</form>
<script>
  form.onclick = function (event) {
    event.target.style.backgroundColor = 'yellow';
    setTimeout(() => {
      alert("target = " + event.target.tagName + ", this=" + this.tagName);
      event.target.style.backgroundColor = ''
    }, 0);
  };
</script>
```

- 핸들러는 `form.onclick`만 존재하지만 폼 안의 모든 요소에서 발생하는 클릭 이벤트를 잡아내고 있는데 이는 클릭 이벤트가 어디서 발생했는지의 여부와 상관없이 `<form>` 요소까지 이벤트가 버블링 되어
  핸들러를 실행
  - `this`(`event.currentTarget`)
    - `<form>` 요소에 있는 핸들러가 동작했으므로 `<form>` 요소
  - `event.target`
    - 폼 안쪽에 실제 클릭한 요소

## 버블링 중단하기

- `<html>` 요소를 거쳐 `document` 객체를 만날 때까지 각 노드에서 모두 발생

### event.stopPropagation

- 한 요소의 특정 이벤트를 처리하는 `핸들러가 여러개인 상황`에서 핸들러 중 `하나가 버블링을 멈춰도 나머지 핸들러는 동작`함
- 위쪽으로 일어나는 버블링은 막아주나, `다른 핸들러들이 동작하는 것은 막지 못함`
- 요소에 할당된 특정 이벤트를 처리하는 핸들러 모두를 동작하지 않게 하려면 `event.stopImmediatePropagation()`을 사용

### 버블링 중단의 문제점

- 버블링을 중단하는 메서드인 `event.stopPropagation()`은 추후에 문제가 될 수 있는 상황을 만들어 냄
- `개발 시나리오`
  1. 중첩 메뉴를 만들었을때, 각 서브메뉴에 해당하는 요소에서 클릭 이벤트를 처리하고 상위 메뉴의 클릭 이벤트 핸들러는 동작하지 않도록 `stopPropagation`을 적용
  2. 사람들이 어디서 뭘 클릭했는지 행동 패턴 분석을 위해 window내에서 발생하는 이벤트 감지 코드(`document.addEventListener(’click’…)`) 추가
  3. `stopPropagation`로 버블링을 막아 놓은 영역은 일명 `죽은 영역(dead zone)`이므로 분석 시스템의 코드가 정상적으로 동작하지 않음

> 버블링을 막는 것이 근본적인 문제 해결이 아니므로, 핸들러의 event 객체에 데이터를 저장해 다른 핸들러에서 읽을 수 있게하는 방법이나 `커스텀 이벤트`를 사용

## 캡처링

- 표준 DOM 이벤트에서 정의한 이벤트 흐름 3가지 단계
  1. 캡처링 단계 - 이벤트가 하위 요소로 전파되는 단계
  2. 타깃 단계 - 이벤트가 실제 타깃 요소에 전달되는 단계
  3. 버블링 단계 - 이벤트가 상위 요소로 전파되는 단계

<img alt="javascript_capturing_bubbling_2" src="/static/images/javascript_capturing_bubbling_2.png" width="600"/>

1. `<td>`를 클릭하면 이벤트가 최상위 조상에서 시작해 아래로 전파(`캡처링`)
2. `타깃 요소`에 도착해 실행(`타겟 단계`)
3. 다시 상위 조상 요소로 전파(`버블링`)

> 캡처링 단계를 이용해야 하는 경우는 흔치 않음

- 캡처링 단계에서 이벤트를 잡아내기 위해서는 `addEventListener`의 `capture` 옵션을 `true`로 설정해서 확인

    ```text
    elem.addEventListener(..., {capture: true})
    // elem.addEventListener(..., true)
    ```

  - `capture` 옵션
    - `false`(=default)면 핸들러는 버블링 단계에서 동작
    - `true`면 핸들러는 캡처링 단계에서 동작

### 캡처링 → 버블링 예제

```html
<!DOCTYPE html>
<html>

<head>
  <style>
    body * {
      margin: 10px;
      border: 1px solid blue;
    }
  </style>
</head>

<body>
  <form>FORM
    <div>DIV
      <p>P</p>
    </div>
  </form>
  <script>
    for (let elem of document.querySelectorAll('*')) {
      elem.addEventListener("click", e => alert(`캡쳐링: ${elem.tagName}`), true);
      elem.addEventListener("click", e => alert(`버블링: ${elem.tagName}`));
    }
  </script>
</body>

</html>
```

- `<p>`를 클릭했을 때의 실행 순서
  1. `HTML` → `BODY` → `FORM` → `DIV` (캡처링 단계, 첫 번째 리스너)
  2. `P`(타겟 단계, 캡처링과 버블링 둘 다 리스너를 설정했으므로 두 번 호출)
  3. `DIV` → `FORM` → `BODY` → `HTML` (버블링 단계, 두 번째 리스너)
- `event.eventPhase` 프로퍼티를 이용하면 현재 발생 중인 이벤트의 흐름 단계를 알 수 있음
- 핸들러를 제거할 때는 `removeEventListener`가 같은 단계에 있어야 함
- 같은 요소와 같은 단계에 설정한 리스너는 `설정한 순서대로 동작`
  - 특정 요소에 `addEventListener`를 사용해 한 단계에 이벤트 핸들러를 여러개 설정했다면 이 핸들러들은 설정한 순서대로 동작

> **Referenced**

- [모던 JavaScript 튜토리얼](https://ko.javascript.info/bubbling-and-capturing)
