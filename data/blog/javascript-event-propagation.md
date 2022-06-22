---
title: Event - 이벤트 전파
date: '2022-06-21'
tags: ['javascript']
draft: false
summary: DOM 트리 상에 존재하는 DOM 요소 노드에서 발생한 이벤트는 DOM 트리를 통해 전파됨
layout: PostSimple
authors: ['default']
---

# Event - 이벤트 전파

## 이벤트 전파

- DOM 트리 상에 존재하는 DOM 요소 노드에서 발생한 이벤트는 DOM 트리를 통해 전파됨

  - ul 요소의 두 번째 자식 요소인 li 요소를 클릭하면 클릭 이벤트가 발생
  - 이벤트를 발생시킨 DOM 요소인 `이벤트 타깃`을 중심으로 `DOM 트리`를 통해 전파됨
  
  <img alt="javascript_event_propagation" src="/static/images/javascript_event_propagation.png" width="500"/>

  - 캡처링 단계: 이벤트가 상위 요소에서 하위 요소 방향으로 전파
  - 타깃 단계: 이벤트가 이벤트 타깃에 도달
  - 버블링 단계: 이벤트가 하위 요소에서 상위 요소 방향으로 전파

```jsx
// HTML
/*
<ul id="fruits">
  <li id="apple">Apple</li>
  <li id="banana">Banana</li>
  <li id="apple">Apple</li>
</ul>
*/

// Javascript
const $fruits = document.getElementById('fruits');
const $banana = document.getElementById('banana');

// #fruits 요소의 하위 요소인 li 요소를 클릭한 경우 캡처링 단계의 이벤트를 캐치
$fruits.addEventListener('click', e => {
  console.log(`이벤트 단계: ${e.eventPhase}`); // 1: 캡처링 단계
  console.log(`이벤트 타깃: ${e.target}`); // [object HTMLLIElement]
  console.log(`이벤트 타깃: ${e.currentTarget}`); // [object HTMLUListElement]
});

// 타깃 단계의 이벤트를 캐치
$banana.addEventListener('click', e => {
  console.log(`이벤트 단계: ${e.eventPhase}`); // 2: 타깃 단계
  console.log(`이벤트 타깃: ${e.target}`); // [object HTMLLIElement]
  console.log(`이벤트 타깃: ${e.currentTarget}`); // [object HTMLUListElement]
});

// 버블링 단계의 이벤트를 캐치
$fruits.addEventListener('click', e => {
  console.log(`이벤트 단계: ${e.eventPhase}`); // 3: 버블링 단계
  console.log(`이벤트 타깃: ${e.target}`); // [object HTMLLIElement]
  console.log(`이벤트 타깃: ${e.currentTarget}`); // [object HTMLUListElement]
});
```

`캡처링 단계`

- li 요소를 클릭하면 클릭 이벤트가 발생하여 클릭 이벤트 객체가 생성되고 클릭된 li 요소가 이벤트 타깃이 됨
  - 클릭 이벤트 객체는 window에서 시작해서 `이벤트 타깃 방향(li)으로 전파`

`버블링 단계`

- 이벤트 객체는 이벤트 타깃으로 시작해서 `window 방향으로 전파됨`

> 이벤트는 이벤트를 발생시킨 이벤트 타깃은 물론 `상위 DOM 요소`에서도 `캐치`할 수 있음

- 다음 이벤트는 버블링을 통해 전파되지 않고 이벤트 객체의 공통 프로퍼티 `event.bubbles`의 값은 모두 `false`
  - 포커스 이벤트: `focus`/`blur`
  - 리소스 이벤트: `load`/`unload`/`abort`/`error`
  - 마우스 이벤트: `mouseenter`/`mouseleave`

## 이벤트 위임

- 여러 개의 하위 DOM 요소에 각각 이벤트 핸들러를 등록하는 대신 하나의 `상위 DOM 요소`에 이벤트 핸들러를 등록하는 방법
- `상위 DOM 요소`에 `이벤트 핸들러를 등록`하면 여러 개의 하위 DOM 요소에 이벤트 핸들러를 등록할 필요가 없음

```jsx
// HTML
/*
<nav>
  <ul id="fruits">
    <li id="apple">Apple</li>
    <li id="banana">Banana</li>
    <li id="apple">Apple</li>
  </ul>
</nav>
<div>선택된 네비게이션 아이템: <em class="msg">apple</em></div>
*/

// Javascript
const $fruits = document.getElementById('fruits');
const $msg = document.querySelector('.msg');

// 사용자 클릭에 의해 선택된 내비게이션 아이템(li 요소)에 active 클래스를 추가하고
// 그 외의 모든 내비게이션 아이템의 active 클래스를 제거
function activate({target}) {
  // 이벤트를 발생시킨 요소(target)가 ul#fruits의 자식 요소가 아니라면 무시
  if (!target.matches('#fruits > li')) return;

  [...$fruits.children].forEach($fruit => {
    $fruit.classList.toggle('active', $fruit === target);
    $msg.textContent = target.id;
  });
}

// 이벤트 위임: 상위 요소(ul#fruits)는 하위 요소의 이벤트를 캐치할 수 있음
$fruits.onclick = activate;

```

- 이벤트를 실제로 발생시킨 DOM 요소가 개발자가 기대한 DOM 요소가 아닐 수도 있으므로, 이벤트에 반응이 필요한 DOM 요소에 한정하여 이벤트핸들러가 실행되도록 이벤트 타깃을 검사할 필요가 있음
- `Element.prototype.matches`
  - 인수로 전달된 선택자에 의해 특정 노드를 탐색 가능한지 확인
  - 이벤트 객체의 `target` 프로퍼티와 `currentTarget` 프로퍼티는 동일한 DOM 요소를 가리키지만 이벤트 위임을 통해 상위 DOM 요소에 이벤트를 바인딩한 경우 이벤트 객체의 `target`
    프로퍼티와 `currentTarget` 프로퍼티가 다른 DOM 요소를 가리킬 수 있음

## DOM 요소의 기본 동작 조작

### DOM 요소의 기본 동작 중단

- `preventDefault`
  - DOM 요소의 기본 동작을 중단시킴

    ```jsx
    // HTML
    // <a href="https://www.google.com">go</a>
    // <input type="checkbox">
    
    // Javascript
    document.querySelector('a').onclick = e => {
    	// a 요소의 기본 동작을 중단
    	e.preventDefault();
    };
    
    document.querySelector('input[type=checkbox]').onclick = e => {
    	// checkbox 요소의 기본 동작을 중단
    	e.preventDefault();
    }
    ```

### 이벤트 전파 방지

- `stopPropagation`
  - 이벤트 전파를 중단 시킴

    ```jsx
    // HTML
    /*
    <div class="container">
    	<button class="btn1">Button 1</button>
    	<button class="btn2">Button 2</button>
    	<button class="btn3">Button 3</button>
    </div>
    */
    
    // Javascript
    // 이벤트 위임. 클릭된 하위 버튼 요소의 color를 변경
    document.querySelector('.container').onclick = ({ target }) => {
    	if (!target.matches('.container > button')) return;
    	target.style.color = 'red';
    };
    
    // .btn2 요소는 이벤트를 전파하지 않으므로 상위 요소에서 이벤트를 캐치할 수 없음
    document.querySelector('.btn2').onclick = e => {
    	e.stopPropagation(); // 이벤트 전파 중단
    	e.target.style.color = 'blue';
    };
    ```

  - `상위 DOM 요소`인 container 요소에 이벤트를 위임하여 `하위 DOM 요소`에서 발생한 클릭 이벤트를 상`위 DOM 요소`인 container 요소가 캐치하여 이벤트를 처리
  - 주로 하위 DOM 요소의 이벤트를 개별적으로 처리하기 위해 `이벤트의 전파를 중단`시키는 메서드로 사용함

## 이벤트 핸들러 내부의 this

### 이벤트 핸들러 어트리뷰트 방식

- handleClick 함수 내부의 this는 전역 객체의 `window`를 가리킴

```jsx
// HTML
// <button onClick="handleClick()">Click me</button>

// Javascript
function handleClick() {
  console.log(this); // window
}
```

- handleClick 함수는 이벤트 핸들러에 의해 일반 함수로 호출되므로 일반 함수로서 호출되는 함수 내부의 this는 전역 객체를 가리킴(⭐⭐⭐)

```jsx
// HTML
// <button onClick="handleClick(this)">Click me</button>

// Javascript
function handleClick(button) {
  console.log(button); // 이벤트를 바인딩한 button 요소
  console.log(this); // window
}
```

- 단, 이벤트 핸들러를 호출할 때 인수로 전달한 this는 이벤트를 바인딩한 DOM 요소를 가리킴

### 이벤트 핸들러 프로퍼티 방식과 addEventListener 메서드 방식

- 이벤트 핸들러 `프로퍼티` 방식과 `addEventListener` 메서드 방식 모두 이벤트 핸들러 내부의 this는 이벤트를 `바인딩한 DOM 요소`를 가리킴
  - 이벤트 핸들러 내부의 this는 이벤트 객체의 `currentTarget` 프로퍼티와 같음

    ```jsx
    // HTML
    // <button class="btn1">0</button>
    // <button class="btn2">0</button>
    
    // Javascript
    const $button1 = document.querySelector('.btn1');
    const $button2 = document.querySelector('.btn2');
    
    // 이벤트 핸들러 프로퍼티 방식
    $button1.onclick = function (e){
    	// this는 이벤트를 바인딩한 DOM 요소를 가리킴
    	console.log(this); // $button1
    	console.log(e.currentTarget); // $button1
    	console.log(this === e.currentTarget); // true
    
    	// $button1의 textContent를 1 증가 시킴
    	++this.textContent;
    };
    
    // addEventListener 메서드 방식
    $button2.addEventListener('click', function (e) {
    	// this는 이벤트를 바인딩한 DOM 요소를 가리킴
    	console.log(this); // $button2
    	console.log(e.currentTarget); // $button2
    	console.log(this === e.currentTarget); // true
    
    	// $button2의 textContent를 1 증가 시킴
    	++this.textContent;
    });
    ```

- `화살표 함수`로 정의한 이벤트 핸들러 내부의 this는 `상위 스코프의 this`를 가리킴
  - 함수 자체의 this 바인딩을 가지지 않음
- 클래스 이벤트 핸들러를 바인딩하는 경우 `bind 메서드`를 사용해 this를 전달하여 increase 메서드 내부의 `this`가 클래스가 `생성할 인스턴스`를 가리키도록 해야 함

## 이벤트 핸들러에 인수 전달

- 함수에 인수를 전달하려면 함수를 호출할 때 전달해야 함
- 이벤트 핸들러 프로퍼티 방식과 `addEventListener` 메서드 방식의 경우 이벤트 핸들러를 브라우저가 호출하기 때문에 함수 호출문이 아닌 함수 자체를 등록해야 하므로 인수를 전달할 수 없음

### 해결

```jsx
// HTML
// <label>User name <input type='text'></label>
// <em class="message"></em>

// Javascript
const MIN_USER_NAME_LENGTH = 5; // 이름 최소 길이
const $input = document.querySelector('input[type=text]');
const $msg = document.querySelector('.message');

const checkUserNameLength = min => {
  $msg.textContent
    = $input.value.length < min ? `이름은 ${min}자 이상 입력해 주세요` : '';
};

// 이벤트 핸들러 내부에서 함수를 호출하면서 인수를 전달함
$input.onblur = () => {
  checkUserNameLength(MIN_UESR_NAME_LENGTH);
};
```

- 이벤트 핸들러 `내부에서 함수를 호출`하면서 인수를 전달

```jsx
// HTML
// <label>User name <input type='text'></label>
// <em class="message"></em>

// Javascript
const MIN_USER_NAME_LENGTH = 5; // 이름 최소 길이
const $input = document.querySelector('input[type=text]');
const $msg = document.querySelector('.message');

const checkUserNameLength = min => e => {
  $msg.textContent
    = $input.value.length < min ? `이름은 ${min}자 이상 입력해 주세요` : '';
};

// 이벤트 핸들러를 반환하는 함수를 호출하면서 인수를 전달
$input.onblur = checkUserNameLength(MIN_UESR_NAME_LENGTH);

```

- 이벤트 핸들러를 `반환하는 함수를 호출`하면서 인수를 전달

## 커스텀 이벤트

### 커스텀 이벤트 생성

- 이벤트 객체는 `Event`, `UIEvent`, `MouseEvent`와 같은 이벤트 생성자 함수로 생성할 수 있는데, 명시적으로 생성한 이벤트 객체는 `임의의 이벤트 타입`을 지정할 수 있음
  - 첫번째 인수로는 이벤트 타입을 나타내는 문자열을 지정할 수 있음

    ```jsx
    // CustomEvent 생성자 함수로 foo 이벤트 타입의 커스텀 이벤트 객체를 생성
    const customEvent = new CustomEvent('foo');
    console.log(customEvent.type); // foo
    ```

- 커스텀 이벤트 객체는 버블링되지 않으며 `preventDefault` 메서드로 취소할 수 없음
  - `bubbles`, `cancelable` 프로퍼티가 `false`가 default임

      ```jsx
      // MouseEvent 생성자 함수로 click 이벤트 타입의 커스텀 이벤트 객체를 생성
      const customEvent = new MouseEvent('click');
      console.log(customEvent.type); // click
      console.log(customEvent.bubbles); // false
      console.log(customEvent.canclable); // false
      ```

- 커스텀 이벤트 객체의 bubble 또는 cancelable 프로퍼티를 true로 설정하려면 이벤트 생성자 함수의 두 번째 인수 `bubbles` 또는 `cancelable` 프로퍼티를 갖는 객체를 전달할 수 있음

    ```jsx
    // MouseEvent 생성자 함수로 click 이벤트 타입의 커스텀 이벤트 객체를 생성
    const customEvent = new MouseEvent('click', {
    	bubbles: true,
    	cancleable: true
    });
    
    console.log(customEvent.bubbles); // true
    console.log(customEvent.canclable); // true
    ```

- 이벤트 생성한 커스텀 이벤트는 `isTrusted` 프로퍼티의 값이 언제나 `false`

    ```jsx
    // InputEvent 생성자 함수로 foo 이벤트 타입의 커스텀 이벤트 객체를 생성
    const customEvent = new InputEvent('foo');
    console.log(customEvent.isTrusted); // false
    ```

  - 반면, 커스텀 이벤트가 아닌 사용자의 행위에 의해 발생한 이벤트에 의해 생성된 이벤트 객체의 `isTrusted` 프로퍼티 값은 언제나 `true`임

### 커스텀 이벤트 디스패치

- 생성된 `커스텀 이벤트`는 dispatchEvent 메서드로 디스패치할 수 있음
  - `dispatchEvent` 메서드에 이벤트 객체를 `인수로 전달`하면서 `호출`하면 인수로 전달한 이벤트 타입의 `이벤트가 발생`함

```jsx
// HTML
// <button class="btn">Click me</button>

// Javascript
const $button = document.querySelector('.btn');

// 버튼 요소에 foo 커스텀 이벤트 핸들러 등록
// 커스텀 이벤트를 디스패치하기 이전에 이벤트 핸들러를 등록
$button.addEventListener('click', e => {
  console.log(e); // MouseEvent {isTrusted: false, screenX: 0, ... }
  alert(`${e} Clicked!`);
});

// 커스텀 이벤트 생성
const customEvent = new MouseEvent('click');

// 커스텀 이벤트 디스패치(동기 처리). click 이벤트 발생
$button.dispatchEvent(customEvent);
```

- `이벤트 핸들러`는 `비동기` 처리 방식으로 동작
- `dispatchEvent` 메서드는 이벤트 핸들러를 `동기` 처리 방식으로 호출
  - 이벤트를 디스패치하기 전에 커스텀 이벤트를 처리할 이벤트 핸들러를 등록해야 함

### 커스텀 이벤트 생성자 함수

```jsx
// HTML
<button class="btn">Click me</button>

// Javascript
const $button = document.querySelector('.btn');

// 버튼 요소에 foo 커스텀 이벤트 핸들러를 등록
// 커스텀 이벤트를 디스패치하기 이전에 이벤트 핸들러를 등록해야 함
$button.addEventListener('foo', e => {
  // e.detail에는 CustomEvent 함수의 두 번째 인수로 전달한 정보가 담겨 있음
  alert(e.detail.message);
});

// CustomEvent 생성자 함수로 foo 이벤트 타입의 커스텀 이벤트 객체 생성
const customEvent = new CustomEvent('foo', {
  detail: {message: 'Hello'} // 이벤트와 함께 전달하고 싶은 정보
});

// 커스텀 이벤트 디스패치
$button.dispatchEvent(customEvent);
```

- 기존 이벤트 타입이 아닌 임의의 이벤트 타입을 지정하여 커스텀 이벤트 객체를 생성하는 경우 반드시 `addEventListener` 메서드 방식으로 이벤트 핸들러를 등록해야 함(⭐⭐⭐)

> `on + 이벤트 타입`으로 이루어진 이벤트 핸들러 어트리뷰트/프로퍼티가 요소 노드에 존재하지 않기 때문에 `이벤트 핸들러 방식`으로 등록해야 함

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 779 ~ 799p