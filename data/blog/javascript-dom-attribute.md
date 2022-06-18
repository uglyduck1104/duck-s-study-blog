---
title: DOM - 어트리뷰트
date: '2022-06-18'
tags: ['javascript']
draft: false
summary: HTML 어트리뷰트와 DOM 프로퍼티, 스타일 조작
layout: PostSimple
authors: ['default']
---

# DOM - 어트리뷰트

## 어트리뷰트

### 어트리뷰트 노드와 attributes 프로퍼티

- HTML 요소는 여러 개의 어트리뷰트를 가지고, HTML 요소의 동작을 제어하기 위한 추가적인 정보를 제공함
  - `어트리뷰트 이름 = "어트리뷰트 값"` 형식으로 정의
- 글로벌 어트리뷰트 및 이벤트 핸들러 어트리뷰트는 모든 HTML 요소에 사용 가능하나, `type`, `value`, `checked`와 같은 어트리뷰트는 `input 요소`에만 사용할 수 있음
- HTML 문서가 파싱될 때 `HTML 요소`의 어트리뷰트는 `어트리뷰트 노드`로 변환되어 `요소 노드`와 연결됨
  - 모든 어트리뷰트 노드의 참조는 유사 배열 객체이자 이터러블인 `NamedNodeMap` 객체에 담겨 `attributes` 프로퍼티에 저장됨
  - 모든 어트리뷰트 노드는 요소 노드의 `Element.prototype.attributes` 프로퍼티로 취득할 수 있음
- attributes 프로퍼티
  - `getter`만 존재하는 `읽기 전용` 접근자 프로퍼티로서 요소 노드의 모든 `어트리뷰트 노드의 참조`가 담긴 `NamedNodeMAp` 객체를 반환함

### HTML 어트리뷰트 조작

- `Element.prototype`/`getAttribte`/`setAttribute`
  - attribute 프로퍼티를 통하지 않고 요소 노드에서 메서드를 통해 `직접 HTML 어트리뷰트 값을 취득`하거나 `변경`할 수 있음

      ```jsx
      // HTML
      // <input id="user" type="text" value="ungmo2">
      
      // Javascript
      const $input = document.getElementById('user');
      
      // value 어트리뷰트 값 취득
      const inputValue = $input.getAttribute('value');
      console.log(inputValue); // ungmo2
      
      // value 어트리뷰트 값 변경
      $input.setAttribute('value', 'foo');
      console.log($input.getAttribute('value')); // foo
      ```

- `Element.prototype.hasAttribute(attributeName)`
  - 특정 HTML 어트리뷰트가 `존재하는지` 확인하는 메서드
- `Element.prototype.removeAttribute(attirbuteName)`
  - 특정 HTML 어트리뷰트 `삭제`

    ```jsx
    // HTML
    // <input id="user" type="text" value="ungmo2">
    
    // Javascript
    const $input = document.getElementById('user');
    
    // value 어트리뷰트의 존재 확인
    if($input.hasAttribute('value')){
    	// value 어트리뷰트 삭제
    	$input.removeAttribute('value');
    }
    
    // value 어트리뷰트 삭제
    console.log($input.hasAttribute('value')); // false
    
    ```

### HTML 어트리뷰트 vs. DOM 프로퍼티

- `HTML 어트리뷰트`의 역할은 HTML 요소의 `초기 상태`를 지정하는 것
  - 어트리뷰트 값은 HTML 요소의 `초기 상태`를 의미하며 변하지 않음
- `요소 노드`는 `상태`를 가지고 있음
  - `초기 상태`
    - `어트리뷰트 노드`가 관리
  - `최신 상태`
    - `DOM 프로퍼티`가 관리
- `어트리뷰트 노드`
  - HTML 어트리뷰트로 지정한 HTML 요소의 `초기 상태`는 `어트리뷰트 노드`에 관리
  - 사용자 입력에 의해 상태가 변경되어도 `변하지 않음`

    ```jsx
    // attribute 프로퍼티에 저장된 value 어트리뷰트 값을 취득함. 결과는 언제나 동일
    document.getElementById('user').getAttribute('value'); // ungmo2
    ```

    ```jsx
    // HTML
    // <input id="user" type="text" value="ungmo2">
    
    // Javascript
    // HTML 요소에 지정한 어트리뷰트 값, 초기 상태 값을 변경
    document.getElementById('user').setAttribute('value', 'foo');
    ```

- `DOM 프로퍼티`
  - 사용자가 입력한 `최신 상태`는 HTML 어트리뷰트에 대응하는 요소 노드의 `DOM 프로퍼티가 관리`
  - DOM 프로퍼티는 사용자의 입력에 의한 상태 변화에 반응하여 `언제나 최신 상태를 유지`

      ```jsx
      // HTML
      // <input id="user" type="text" value="ungmo2">
      
      // Javascript
      const $input = document.getElementById('user');
      
      // DOM 프로퍼티에 값을 할당하여 HTML 요소의 최신 상태를 변경
      $input.value = 'foo';
      console.log($input.value); // foo
      
      // getAttribute 메서드로 취득한 HTML 어트리뷰트 값, 초기 상태 값은 변하지 않고 유지됨
      console.log($input.getAttribute('value')); // ungmo2
      ```


- `HTML 어트리뷰트와 DOM 프로퍼티의 대응 관계`
  - id 어트리뷰트와 id 프로퍼티는 1:1 대응하며, 동일한 값으로 연동
  - `input` 요소의 `value` 어트리뷰트 `value` 프로퍼티와 1:1 대응함
    - value 어트리뷰트는 `초기 상태`를, value 프로퍼티는 `최신 상태`를 갖음
  - `class` 어트리뷰트는 `className`, `classList` 프로퍼티와 대응
  - `for` 어트리뷰트는 `htmlFor` 프로퍼티와 1:1 대응
  - td 요소의 colspan 어트리뷰트는 대응하는 프로퍼티가 없음
  - textContent 프로퍼티는 대응하는 어트리뷰트가 존재하지 않음
  - 어트리뷰트 이름은 대소문자를 구별하지는 않지만, `프로퍼티 키`는 `카멜 케이스`를 따름(maxlength → `maxLength`)

### data 어트리뷰트와 dataset 프로퍼티

- HTML 요소에 정의한 `사용자 정의` 어트리뷰트와 자바스크립트 간에 `데이터를 교환`할 수 있음
  - data-user-id, data-role과 같이 `data-*` 접두사를 사용
- `data` 어트리뷰트 값은 `HTMLElement.dataset` 프로퍼티로 취득 가능
- `dataset` 프로퍼티는 `DOMStringMap` 객체를 반환
  - `data-` 접두사 다음에 붙인 임의의 이름을 `카멜 케이스`로 변환한 `프로퍼티`를 가지고 있음

      ```jsx
      // HTML
      /*
      <ul class="users">
        <li id="1" data-user-id="7621" data-role="admin">Lee</li>
        <li id="2" data-user-id="9524" data-role="subscribe">Lee</li>
      </ul>
      */
      
      // Javascript
      const users = [...document.querySelector('.users').children];
      
      // user-id가 '7621'인 요소 노드를 취득
      const user = users.find(user => user.dataset.userId === '7621');
      // user-id가 '7621'인 요소 노드에서 data-role의 값을 취득
      console.log(user.dataset.role); // "admin"
      
      // user-id가 '7621'인 요소 노드의 data-role 값을 변경
      user.dataset.role = 'subscriber';
      // dataset 프로퍼티는 DOMStringMap 객체를 반환
      console.log(user.dataset); // DOMStringMap {userId: "7621", role: "subscriber"}
      ```

## 스타일

### 인라인 스타일 조작

- `HTMLElement.prototype.style`
  - `setter`와 `getter` 모두 존재하는 접근자 프로퍼티
  - 요소 노드의 `인라인 스타일`을 취득하거나 추가 또는 변경

    ```jsx
    // HTML
    // <div style="color: red">Hello World</div>
    
    // Javascript
    const $div = document.querySelector('div');
    
    // 인라인 스타일 취득
    console.log($div.style); // CSSStyleDeclaration { 0: "color", ... }
    
    // 인라인 스타일 변경
    $div.style.color = 'blue';
    
    // 인라인 스타일 추가
    $div.style.width = '100px';
    $div.style.height = '100px';
    $div.style.backgroundColor = 'yellow';
    ```

  - style 프로퍼티를 참조하면 `CSSStyleDeclaration` 타입의 객체를 반환
    - `CSSStyleDeclaration` 객체의 프로퍼티는 카멜 케이스를 따름
  - CSS 프로퍼티는 `케밥 케이스`를 따름

### 클래스 조작

- class 어트리뷰트에 대응하는 DOM 프로퍼티는 `className`과 `classList`
- `Element.prototype.className`
  - `setter`와 `getter` 모두 존재하는 접근자 프로퍼티
  - className 프로퍼티를 `참조`하면 class 어트리뷰트 값을 `문자열로 반환`
  - className 프로퍼티에 문자열을 `할당`하면 class 어트리뷰트 값을 할당한 `문자열로 변경`

    ```jsx
    // HTML
    // <div class="box red">Hello World</div>
    
    // Javascript
    const $box = document.querySelector('.box');
    
    // .box 요소의 class 어트리뷰트 값 취득
    console.log($box.className); // 'box red'
    
    // .box 요소의 class 어트리뷰트 값 중에서 'red'만 'blue'로 변경
    $box.className = $box.className.replace('red', 'blue');
    ```

  - 공백으로 구분된 여러 개의 클래스를 반환하는 경우 다루기 불편함
- `Element.prototype.classList`
  - class 어트리뷰트의 정보를 담은 `DOMTokenList` 객체 반환

    ```jsx
    // HTML
    // <div class="box red">Hello World</div>
    
    // Javascript
    const $box = document.querySelector('.box');
    
    // .box 요소의 class 어트리뷰트 정보를 담은 DOMTokenList 객체를 취득
    // classList가 반환하는 DOMTokenList 객체는 HTMLCollection과 NodeList와 같이
    // 노드 객체의 상태 변화를 실시간을 반영하는 살아 있는(live) 객체
    console.log($box.classList);
    // DOMTokenList(2) [length: 2, value: "box blue", 0: "box", 1: "blue"]
    
    // .box 요소의 class 어트리뷰트 값 중에서 'red'만 'blue'로 변경
    $box.classList.replace('red', 'blue');
    ```

  - `DOMTokenList`
    - `add`(…className)
      - 인수로 전달한 1개 이상의 문자열을 class 어트리뷰트 값으로 `추가`

        ```jsx
        $box.classList.add('foo'); // -> class="box red foo"
        $box.classList.add('bar', 'baz'); // -> class="box red foo bar baz"
        ```

    - `remove`(…className)
      - 인수로 전달한 1개 이상의 문자열과 일치하는 클래스를 class 어트리뷰트를 `삭제`

        ```jsx
        $box.classList.remove('foo'); // -> class="box red bar baz"
        $box.classList.remove('bar', 'baz'); // -> class="box red"
        $box.classList.remove('x'); // -> class="box red"
        ```

    - `item`(index)
      - 인수로 전달한 `index`에 해당하는 클래스를 class 어트리뷰트에서 `반환`함

        ```jsx
        $box.classList.item(0); // -> "box"
        $box.classList.item(1); // -> "red"
        ```

    - `contains`(className)
      - 인수로 전달한 문자열과 일치하는 클래스가 class 어트리뷰트에 포함되어 있는지 `확인`

        ```jsx
        $box.classList.contains('box'); // -> true
        $box.classList.contains('blue'); // -> false
        ```

    - `replace`(oldClassName, newClassName)
      - class 어트리뷰트에서 첫 번째 인수로 전달할 문자열을 두 번째 인수로 전달한 문자열로 `변경`

        ```jsx
        $box.classList.replace('red', 'blue'); // -> class="box blue"
        ```

    - `toggle`(className[, force])
      - class 어트리뷰트에 인수로 전달한 문자열과 일치하는 클래스가 존재하면 제거, 존재하지 않으면 추가

        ```jsx
        $box.classList.toggle('foo'); // -> class="box blue foo"
        $box.classList.toggle('foo'); // -> class="box blue"
        ```

### 요소에 적용되어 있는 CSS 스타일 참조

- style 프로퍼티는 인라인 스타일만 반환하므로, 클래스를 적용한 스타일이나 상속을 통해 `암묵적으로 적용된 스타일`은 style 프로퍼티로 `참조할 수 없음`
- HTML 요소에 적용되어 있는 모든 CSS 스타일을 참조할 경우 `getComputedStyle` 메서드를 사용
- `Window.getComputedStyle(element[, pseudo])`
  - 첫 번째 인수(`element`)로 전달한 요소 노드에 적영되어 있는 평가된 스타일(computed style)을 `CSSStyleDeclaration` 객체에 담아 반환

    ```jsx
    // HTML
    /*
    <head>
    	<style>
    		body {
    			color: red;
    		}
    		.box {
    			width: 100px;
    			height: 50px;
    			background-color: cornsilk;
    			border: 1px solid black;
    		}
    	</style>
    </head>
    <body>
    	<div class="box">Box</di>
    </body>
    */
    
    // Javascript
    const $box = document.querySelector('.box');
    
    // .box 요소에 적용된 모든 CSS 스타일을 담고 있는 CSSStyleDeclaration 객체를 취득
    const computedStyle = window.getComputedStyle($box);
    console.log(computedStyle); // CSSStyleDeclaration
    
    // 임베딩 스타일
    console.log(computedStyle.width); // 100px
    console.log(computedStyle.height); // 50px
    console.log(computedStyle.backgroundColor); // rgb(255, 248, 220)
    console.log(computedStyle.border); // 1px solid rgb(0, 0, 0)
    
    // 상속 스타일(body -> .box)
    console.log(computedStyle.color); // rgb(255, 0, 0)
    
    // 기본 스타일
    console.log(computedStyle.display); // block
    ```

    ```jsx
    // HTML
    /*
    <head>
    	<style>
    		.box:before{
    			content: 'Hello';
    		}
    	</style>
    </head>
    <body>
    	<div class="box">Box</di>
    </body>
    */
    
    // Javascript
    const $box = document.querySelector('.box');
    
    // 의사 요소 :before의 스타일을 취득
    const computedStyle = window.getComputedStyle($box, ':before');
    console.log(computedStyle.content); // "Hello"
    ```

  - 두 번째 인수로(pseudo) :after, :before와 같은 의사 요소를 지정하는 문자열을 전달할 수 있음

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 734 ~ 753p
