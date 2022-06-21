---
title: Event - 이벤트 타입과 핸들러 조작
date: '2022-06-20'
tags: ['javascript']
draft: false
summary: 이벤트 타입의 종류와 이벤트 객체
layout: PostSimple
authors: ['default']
---

# Event - 이벤트 타입과 핸들러 조작

## 이벤트 드리븐 프로그래밍

- 처리해야 할 `특정 사건`이 발생하면 이를 감지하여 이벤트를 발생시킴
  - 클릭, 키보드 입력, 마우스 이동 등
- `이벤트 핸들러`
  - 이벤트가 발생했을 때 `호출될 함수`
- `이벤트 핸들러 등록`
  - 이벤트가 발생했을 때 `브라우저에게 이벤트 핸들러의 호출을 위임`
- 개발자가 명시적으로 함수를 호출하는 것이 아니라, `브라우저`에게 함수 호출을 `위임`하는 것

> 사용자와 애플리케이션은 `상호작용`을 할 수 있고 이와 같이 프로그램의 흐름을 `이벤트 중심으로 제어`하는 프로그래밍 방식을 이벤트 드리븐 프로그래밍이라 함

## 이벤트 타입

### 마우스 이벤트

| 이벤트 타입     | 이벤트 발생 시점                             |
|------------|---------------------------------------|
| click      | 마우스 버튼을 클릭했을 때                        |
| dblclick   | 마우스 버튼을 더블 클릭했을 때                     |
| mousedown  | 마우스 버튼을 눌렀을 때                         |
| mouseup    | 누르고 있던 마우스 버튼을 놓았을 때                  |
| mousemove  | 마우스 커서를 움직였을 때                        |
| mouseenter | 마우스 커서를 HTML 요소 안으로 이동했을 때(버블링 되지 않음) |
| mouseover  | 마우스 커서를 HTML 요소 안으로 이동했을 때(버블링됨)      |
| mouseleave | 마우스 커서를 HTML 요소 밖으로 이동했을 때(버블링 되지 않음) |
| mouseout   | 마우스 커서를 HTML 요소 밖으로 이동했을 때(버블링됨)      |

### 키보드 이벤트

| 이벤트 타입   | 이벤트 발생 시점               |
|----------|-------------------------|
| keydown  | 모든 키를 눌렀을 때 발생함         |
| keypress | 문자 키를 눌렀을 때 연속적으로 발생    |
| keyup    | 누르고 있던 키를 놓았을 때 한 번만 발생 |

### 포커스 이벤트

| 이벤트 타입   | 이벤트 발생 시점                     |
|----------|-------------------------------|
| focus    | HTML 요소가 포커스를 받았을 때(버블링되지 않음) |
| blur     | HTML 요소가 포커스를 잃었을 때(버블링되지 않음) |
| focusin  | HTML 요소가 포커스를 받았을 때(버블링됨)     |
| focusout | HTML 요소가 포커스를 잃었을 때(버블링됨)     |
- `focusin`, `focusout` 이벤트 핸들러를 프로퍼티 방식으로 등록하면 크롬, 사파리에서 정상 동작하지 않음
  - `addEventListener` 메서드 방식으로 사용 해 등록해야 함

### 폼 이벤트

- `submit`
  1. form 요소 내의 `input`(text, checkbox, radio), `select` 입력 필드(textarea 제외)에서 `엔터키`를 눌렀을 때
  2. form 요소 내의 `submit` 버튼(`<button>`, `<input type="submit">`)을 클릭했을 때
    - submit 이벤트는 form 요소에서 발생
- `reset`
  - form 요소 내의 `reset` 버튼을 클릭했을 때(최근에는 사용 안 함)

### 값 변경 이벤트

| 이벤트 타입           | 이벤트 발생 시점                                                                                        |
|------------------|--------------------------------------------------------------------------------------------------|
| input            | input(text, checkbox, radio), select, textarea 요소의 값이 입력되었을 때                                    |
| change           | input(text, checkbox, radio), select, textarea 요소의 값이 변경되었을 때                                    |
| readystatechange | HTML 문서의 로드와 파싱 상태를 나타내는 document.readyState 프로퍼티 값(’loading’, ‘interactive’, ‘complete’)이 변경될 때 |

### DOM 뮤테이션 이벤트

| 이벤트 타입           | 이벤트 발생 시점                             |
|------------------|---------------------------------------|
| DOMContentLoaded | HTML 문서의 로드와 파싱이 완료되어 DOM 생성이 완료되었을 때 |

### 뷰 이벤트

| 이벤트 타입 | 이벤트 발생 시점                                  |
|--------|--------------------------------------------|
| resize | 브라우저 윈도우(window)의 크기를 리사이즈할 때 연속적으로 발생     |
| scroll | 웹페이지(document) 또는 HTML 요소를 스크롤할 때 연속적으로 발생 |

### 리소스 이벤트

| 이벤트 타입 | 이벤트 발생 시점                                                                       |
|--------|---------------------------------------------------------------------------------|
| load   | DOMContentLoaded 이벤트가 발생한 이후, 모든 리소수(이미지, 폰트 등)의 로딩이 완료되었을 때(주로 window 객체에서 발생) |
| unload | 리소스가 언로드될 때(주로 새로운 웹페이지 요청한 경우)                                                 |
| abort  | 리소스 로딩이 중단되었을 때                                                                 |
| error  | 리소스 로딩이 실패했을 때                                                                  |

## 이벤트 핸들러 등록

### 이벤트 핸들러 어트리뷰트

- 이름은 onclick과 같이 on 접두사와 이벤트의 종류를 나타내는 이벤트 타입의 조합으로 이루어져 있음
- 함수 호출문 등의 문을 할당하면 이벤트 핸들러가 등록됨

```jsx
// HTML
// <button onclick="sayHi('Lee')">Click me!</button>

// Javascript
function sayHi(name){
	console.log(`Hi! ${name}.`);
}
```

- 이벤트 핸들러 어트리뷰트 값은 암묵적으로 생성될 이벤트 핸들러의 함수 몸체를 의미함
  - 예를 들어 `onclick="sayHi('Lee')"` 어트리뷰트는 파싱되어 `암묵적으로 함수를 생성`하고, 이벤트 핸들러 어트리뷰트 이름과 동일한 onclick 이벤트 핸들러 프로퍼티에 할당함

  > HTML과 자바스크립트는 관심사가 다르므로 혼재하는 것보다 분리하는 것이 좋음
>
- CBD(`Component Based Development`) 방식의 Angular/React/Svelte/Vue.js 같은 프레임워크/라이브러리에서는 이벤트 핸들러 어트리뷰트 방식으로 이벤트를 처리함

  > 자바스크립트를 관심사가 다른 개별적인 요소가 아닌, 뷰를 구성하기 위한 구성 요소로 보므로 관심사가 다르다고 생각하지 않음
>

### 이벤트 핸들러 프로퍼티 방식

- 이벤트 핸들러 어트리뷰트와 마찬가지로 onclick과 같이 on 접두사와 이벤트의 종류를 나타내는 이벤트 타입으로 이루어져 있음

```jsx
// HTML
// <button>Click me!</button>

// Javascript
const $button = document.querySelector('button');

// 이벤트 핸들러 프로퍼티에 이벤트 핸들러를 바인딩
$button.onclick = function (){
	console.log('button click');
};
```

- `이벤트 타깃`과 이벤트의 종류를 나타내는 문자열인 `이벤트 타입`, `이벤트 핸들러`를 지정할 필요가 있음

  <img alt="javascript_event_handler1" src="/static/images/javascript_event_handler1.png" width="550"/>

- 이벤트 핸들러 프로퍼티 방식은 `하나의 이벤트`에 `하나의 이벤트 핸들러`만을 바인딩 할 수 있음

### addEventListener 메서드 방식

- DOM Level 2에 도입된 `EventTarget.prototype.addEventListener` 메서드를 사용해 이벤트 핸들러 등록
  - 이벤트 핸들러 어트리뷰트 방식과 이벤트 핸들러 프로퍼티 방식은 DOM Level 0부터 제공되던 방식

    ![javascript_event_handler2](/static/images/javascript_event_handler2.png)

    - 첫 번째 매개변수
      - 이벤트 종류를 나타내는 문자열인 `이벤트 타입`을 전달
      - 이벤트 핸들러 프로퍼티 방식과 달리 on 접두사를 붙이지 않음
    - 두 번째 매개변수
      - `이벤트 핸들러` 전달
    - 마지막 매개변수
      - `버블링` 단계에서 이벤트를 캐치할지에 대한 `boolean값` 전달
- 동일한 HTML 요소에서 동일한 이벤트에 대해 `이벤트 핸들러 프로퍼티 방식`은 하나 이상의 이벤트 핸들러를 등록할 수 없지만 `addEventListener` 메서드는 하나 이상의 이벤트 핸들러를 등록할 수 있음

    ```jsx
    // HTML
    <button>Click me!</button>
    
    // Javascript
    const $button = document.querySelector('button');
    
    // addEventListener 메서드는 동일한 요소에서 발생한 동일한 이벤트에 대해
    // 하나 이상의 이벤트 핸들러를 등록할 수 있음
    $button.addEventListener('click', function() {
    	console.log('[1]button click');
    });
    
    $button.addEventListener('click', function() {
    	console.log('[2]button click');
    });
    ```

  - 등록된 순서대로 호출

    ```jsx
    // HTML
    <button>Click me!</button>
    
    // Javascript
    const $button = document.querySelector('button');
    
    const handleClick = () => console.log('button click');
    
    $button.addEventListener('click', handleClick);
    $button.addEventListener('click', handleClick);
    ```

  - `참조가 동일한 이벤트 핸들러`를 중복 등록하면 `하나의 이벤트 핸들러`만 등록됨

## 이벤트 핸들러 제거

- `EventTaget.prototype.removeEventListener`
  - `removeEventListener` 메서드에 전달할 인수는 `addEventListener` 메서드와 동일함
  - `addEventListener` 메서드에 전달한 인수와 `removeEventListener` 메서드에 전달한 인수가 일치하지 않으면 이벤트 핸들러가 제거되지 않음

      ```jsx
      // 이벤트 핸들러 등록
      $button.addEventListener('click', () => console.log('button click'));
      // 등록한 이벤트 핸들러를 참조할 수 없으므로 제거할 수 없음
      ```

- 이벤트 핸들러 프로퍼티 방식으로 등록한 이벤트 핸들러는 `removeEventListener` 메서드로 제거할 수 없음

    ```jsx
    // HTML
    <button>Click me!</button>
    
    // Javascript
    const $button = document.querySelector('button');
    
    const handleClick = () => console.log('button click');
    
    // 이벤트 핸들러 프로퍼티 방식으로 이벤트 핸들러 등록
    $button.onclic = handleClick;
    
    // removeEventListener 메서드로 이벤트 핸들러를 제거할 수 없으므로 null을 할당하여 이벤트 핸들러를 제거
    $button.onclick = null;
    ```


## 이벤트 객체

- 생성된 이벤트 객체는 이벤트 핸들러의 `첫 번째 인수로 전달`됨
- 이벤트 핸들러 어트리뷰트 방식의 경우 이벤트 객체를 전달받으려면 이벤트 핸들러의 `첫 번째 매개변수` 이름이 반드시 `event`이어야 함
  - 다른 이름으로 매개변수를 선언하면 이벤트 객체를 전달받지 못함

### 이벤트 객체의 상속 구조

- 이벤트가 발생하면 이벤트 타입에 따라 다양한 타입의 이벤트 객체가 생성됨

  <img alt="javascript_event_handler3" src="/static/images/javascript_event_handler3.png" width="750"/>


### 이벤트 객체의 공통 프로퍼티

- `type`(string)
  - 이벤트 타입
- `target`(DOM 요소 노드)
  - 이벤트를 `발생시킨 DOM 요소`
- `currentTarget`(DOM 요소 노드)
  - 이벤트 핸들러가 `바인딩된 DOM 요소`
- `eventPhase`(number)
  - 이벤트 `전파 단계`
    - 0: 이벤트 없음
    - 1: 캡처링 단계
    - 2: 타겟 단계
    - 3: 버블링 단계
- `bubbles`(boolean)
  - 이벤트를 `버블링으로 전파`하는지 여부
    - 포커스 이벤트 `focus`/`blur`
    - 리소스 이벤트 `load`/`unload`/`abort`/`error`
    - 마우스 이벤트 `mouseenter`/`mouseleave`
- `cancelable`(boolean)
  - preventDefault 메서드를 호출하여 기본 동작을 `취소`할 수 있는지 여부
    - 포커스 이벤트 `focus`/`blur`
    - 리소스 이벤트 `load`/`unload`/`abort`/`error`
    - 마우스 이벤트 `dblclick`/`mouseenter`/`mouseleave`
- `defaultPrevented`(boolean)
  - preventDefault 메서드를 호출하여 이벤트를 취소했는지 여부
- `isTrusted`(boolean)
  - 사용자의 행위에 의해 발생한 이벤트인지 여부
- `timeStamp`(number)
  - 이벤트가 발생한 시각(1970/01/01/00:00:0부터 경과한 밀리초)

### 마우스 정보 취득

- `MouseEvent` 타입의 이벤트 객체는 고유의 프로퍼티를 가짐
  - 마우스 포인터의 좌표정보를 나타내는 프로퍼티
    - `screenX`/`screenY`
    - `clientX`/`clientY`
    - `pageX`/`pageY`
    - `offsetX`/`offsetY`
  - 버튼 정보를 나타내는 프로퍼티
    - `altKey`, `ctrlKey`, `shiftKey`, `button`

### 키보드 정보 취득

- `KeyboardEvent` 타입의 이벤트 객체는 고유의 프로퍼티를 가짐
  - `ctrlKey`, `shiftKey`, `metaKey`, `key`, `keyCode`

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 754 ~ 778p