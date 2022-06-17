---
title: DOM - DOM 조작
date: '2022-06-17'
tags: ['javascript']
draft: false
summary: DOM의 요소 노드의 텍스트 조작과 DOM 조작
layout: PostSimple
authors: ['default']
---

# DOM - DOM 조작

## 요소 노드의 텍스트 조작

### nodeValue

- `setter`와 `getter` 모두 존재하는 접근자 프로퍼티로서, 참조와 할당 모두 가능
- 노드 객체의 `nodeValue` 프로퍼티를 참조하면 노드 객체의 값을 반환함

    ```jsx
    // HTML
    // <div id="foo">Hello <span>world!</span></div>
    
    // Javascript
    // #foo 요소 노드의 자식 노드인 텍스트 노드를 취득
    console.log(document.getElementById('foo').nodeValue); // null
    // #foo 요소 노드의 자식 노드인 텍스트 노드의 값을 취득함
    console.log(document.getElementById('foo').firstChild.nodeValue); // Hello
    // span 요소 노드의 자식 노드인 텍스트 노드의 값을 취득함
    console.log(document.getElementById('foo').lastChild.firstChild.nodeValue); // world!
    ```

  <img alt="javascript_dom_manipulation1" src="/static/images/javascript_dom_manipulation1.png" width="650"/>

  1. 텍스트를 변경할 `요소 노드를 취득`한 다음, 취득한 요소 노드의 `텍스트 노드를 탐색`

  - 텍스트 노드는 요소 노드의 자식 노드이므로 firstChild 프로퍼티를 사용해 탐색

  2. 탐색한 텍스트 노드의 nodeValue 프로퍼티를 사용하여 `텍스트 노드의 값을 변경`

### textContent

- `setter`와 `getter` 모두 존재하는 접근자 프로퍼티로서, 참조와 할당 모두 가능
- 요소 노드의 `textContent` 프로퍼티를 참조하면 요소 노드의 콘텐츠 영역 내의 `텍스트`를 모두 반환
- `childNodes` 프로퍼티가 반환한 모든 노드들의 텍스트 노드의 값(`텍스트`) 반환

  ```jsx
  // HTML
  // <div id="foo">Hello <span>world!</span></div>
    
  // Javascript
  // #foo 요소 노드의 텍스트를 모두 취득함. HTML 마크업 무시
  console.log(document.getElementById('foo').textContent); // Hello world!
  ```

  <img alt="javascript_dom_manipulation2" src="/static/images/javascript_dom_manipulation2.png" width="600"/>

- textContent 프로퍼티에 문자열을 할당하면 모든 자식 노드가 제거되고 할당한 문자열이 텍스트로 추가됨
  - `HTML` 마크업은 `파싱되지 않음`

### nodeValue vs textContent

- `nodeValue` 프로퍼티를 사용하면 `textContent` 프로퍼티를 사용할 때와 비교해서 코드가 더 복잡함
- 요소 노드의 콘텐츠 영역에 자식 요소 노드가 없고 텍스트만 존재한다면 `firstChild.nodevalue`와 `textContent` 프로퍼티는 같은 결과를 반환

    ```jsx
    // HTML
    // 요소 노드의 콘텐츠 영역에 다른 요소 노드가 없고 텍스트만 존재
    // <div id="foo">Hello</div>
    
    const $foo = document.getElementById('foo');
    
    // 요소 노드의 콘텐츠 영역에 자식 요소 노드가 없고 텍스트만 존재한다면
    // firstChild.nodeValue와 textContent는 같은 결과를 반환함
    console.log($foo.textContent === $foo.firstChild.nodeValue); // true
    ```

### innerText 프로퍼티 사용 자제

1. innderText 프로퍼티는 CSS에 순종적임

- CSS에 의해 비표시(`visibility: hidden;`)로 지정된 요소 노드의 텍스트를 반환하지 않음

2. innerText 프로퍼티는 CSS를 고려해야하므로 `textContent 프로퍼티보다 느림`

## DOM 조작

- 새로운 노드를 생성하여 DOM에 추가하거나 기존 노드를 `삭제` 또는 `교체`하는 것
  - 리플로우와 리페인트가 발생하는 원인이되므로 성능에 영향을 줌

> 복잡한 콘텐츠를 다루는 DOM 조작은 성능 최적화를 위해 주의해서 다뤄야 함
>

### innerHTML

- `Element.prototype.innerHTML`
  - `setter`와 `getter` 모두 존재하는 접근자 프로퍼티로서, 참조와 할당 모두 가능
  - textContent 프로퍼티와 달리 `HTML 마크업이 포함`된 문자열 그대로 반환

  <img alt="javascript_dom_manipulation3" src="/static/images/javascript_dom_manipulation3.png" width="700"/>

  - 요소 노드의 모든 자식 노드가 제거되고 할당한 문자열에 포함되어 있는 `HTML 마크업이 파싱`되어 요소 노드의 자식 노드로 `DOM에 반영`됨

`단점`

- 사용자로부터 입력받은 데이터를 그대로 할당하는 것은 XSS(`크로스 사이트 스크립팅 공격 - Cross-Site Scripting Attacks`) 공격에 위험함
  - `HTML 새니티제이션`(HTML sanitization) 함수 및 라이브러리(`DOMPurify`) 사용 권장
- 요소 노드의 `모든 자식 노드를 제거`하고 할당한 HTML 마크업 문자열을 파싱하여 DOM을 변경함
  - 리소스 낭비
- 새로운 요소를 삽입할 때 `삽입될 위치`를 지정할 수 없음

### insertAdjacentHTML

- `Element.prototype.insertAdjacentHTML(position, DOMString)`
  - 기존 요소를 제거하지 않고, 위치를 지정해 새로운 요소를 삽입
  - 첫번째 인수 `position` 값
    - `beforebegin`
    - `afterbegin`
    - `beforeend`
    - `afterend`

    <img alt="javascript_dom_manipulation4" src="/static/images/javascript_dom_manipulation4.png" width="700"/>

- 기존 요소에는 영향을 주지 않고 `새롭게 삽입될 요소만 파싱`하여 자식 요소로 추가하므로 `innerHTML` 프로퍼티보다 `효율적이고 빠름`
  - 단, XSS Attack에 취약한 것은 동일함

### 노드 생성과 추가

`요소 노드 생성`

- `Document.prototype.createElement(tagName)`
  - 요소 노드를 생성하여 반환하고 매개변수 `tagName`에는 태그 이름을 나타내는 문자열을 인수로 전달

      ```jsx
      // 1. 요소 노드 생성
      const $li = document.createElement('li');
      ```

    <img alt="javascript_dom_manipulation5" src="/static/images/javascript_dom_manipulation5.png" width="300"/>

    - 기존 DOM에 추가되지 않고 `홀로 존재`하는 상태
    - 요소 노드를 생성할 뿐 DOM에 추가되지 않으므로, 이후에 생성된 요소 노드를 `DOM에 추가하는 처리`가 별도로 필요함

`텍스트 노드 생성`

- `Document.prototype.createTextNode(text)`
  - 텍스트 노드를 생성하여 반환하고 매개변수 `text`에는 텍스트 노드의 값으로 사용할 문자열을 인수로 전달

      ```jsx
      // 2. 텍스트 노드 생성
      const textNode = document.createTextNode('Banana');
      ```

    <img alt="javascript_dom_manipulation6" src="/static/images/javascript_dom_manipulation6.png" width="300"/>

    - 기존 DOM에 추가되지 않고 `홀로 존재`하는 상태
    - 요소 노드를 생성할 뿐 DOM에 추가되지 않으므로, 이후에 생성된 요소 노드를 `DOM에 추가하는 처리`가 별도로 필요함

`텍스트 노드를 요소 노드의 자식 노드로 추가`

- `Node.prototype.appendChild(childNode)`
  - 매개변수 `childNode`에게 인수로 전달한 노드를 `appendChild` 메서드를 호출한 노드의 `마지막 자식 노드로 추가`

      ```jsx
      // 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
      $li.appendChild(textNode);
      ```

    <img alt="javascript_dom_manipulation7" src="/static/images/javascript_dom_manipulation7.png" width="300"/>

    - 부자 관계로 연결되었지만 기존 DOM에 추가되지는 않은 상태
    - 자식 노드가 `하나도 없는 경우`
      - 텍스트 노드를 생성하여 요소 노드의 자식 노드로 텍스트로 추가하는 것보다 `textContent` 프로퍼티를 사용하는 편이 더욱 간편함

`요소 노드를 DOM에 추가`

- `Node.prototype.appendChild`
  - 텍스트 노드와 부자 관계로 연결한 요소 노드를 `#fruits` 요소 노드의 마지막 자식 요소로 추가

      ```jsx
      // 4. $li 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
      $fruits.appendChild($li);
      ```

    <img alt="javascript_dom_manipulation8" src="/static/images/javascript_dom_manipulation8.png" width="300"/>

    - 새롭게 생성한 요소 노드가 DOM에 추가됨
    - 단 하나의 요소 노드를 생성하여 DOM에 한 번 추가하므로 DOM은 한 번 변경됨

### 복수의 노드 생성과 추가

- 요소 노드를 DOM에 추가하는 작업은 리플로우와 리페인트가 n번 실행되므로 가급적 횟수를 줄이는 편이 성능에 좋음

`컨테이너 요소(div)로 추가`

```jsx
// HTML
// <ul id="fruits"></ul>

// Javascript
const $fruits = document.getElementById('fruits');

// 컨테이너 요소 노드 생성
const $container = document.createElement('div');

['Apple', 'Banana', 'Orange'].forEach(text => {
  // 1. 요소 노드 생성
  const $li = document.createElement('li');

  // 2. 텍스트 노드 생성
  const textNode = document.createTextNode(text);

  // 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
  $li.appendChild(textNode);

  // 4. $li 요소 노드를 컨테이너 요소의 마지막 자식 노드로 추가
  $container.appendChild($li);
});

// 5. 컨테이너 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
$fruits.appendChild($container);
```

- 불필요한 컨테이너 요소(`div`)가 추가됨

    ```html
    <ul id="fruits">
    	<div>
    		<li>apple</li>
    		<li>banana</li>
    		<li>orange</li>
    	</div>
    </ul>
    ```

`DocumentFragment`  사용

- 문서, 요소, 어트리뷰트, 텍스트 노드와 같은 노드 객체의 일종
  - 부모 노드가 없어서 기존 DOM과는 `별도로 존재`함
- 자식 노드들의 부모 노드로서 별도의 `서브 DOM`을 구성하여 `기존 DOM에 추가`하기 위한 용도

<img alt="javascript_dom_manipulation9" src="/static/images/javascript_dom_manipulation9.png" width="700"/>

```jsx
// HTML
// <ul id="fruits"></ul>

// Javascript
const $fruits = document.getElementById('fruits');

// DocumentFragment 노드 생성
const $fragment = document.createDocumentFragment();

['Apple', 'Banana', 'Orange'].forEach(text => {
  // 1. 요소 노드 생성
  const $li = document.createElement('li');

  // 2. 텍스트 노드 생성
  const textNode = document.createTextNode(text);

  // 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
  $li.appendChild(textNode);

  // 4. $li 요소 노드를 DocumentFragment 노드의 마지막 자식 노드로 추가
  $fragment.appendChild($li);
});

// 5. 컨테이너 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
$fruits.appendChild($container);
```

1. `DocumentFragment` 노드 생성
2. DOM에 추가할 `요소 노드 생성`
3. DocumentFragment 노드에 `자식 노드로 추가`
4. DocumentFragment 노드를 `기존 DOM에 추가`

### 노드 삽입

`마지막 노드로 추가`

- `Node.prototype.appendChild`
  - 언제나 마지막 자식 노드로 추가

`지정한 위치에 노드 삽입`

- `Node.prototype.insertBefore(newNode, childNode)`
  - `첫 번째 인수`로 전달받은 노드를 `두 번째 인수`로 전달받은 노드 앞에 삽입
  - `두 번째 인수`로 전달받은 노드는 반드시 `insertBefore` 메서드를 호출한 노드의 자식 노드여야 함
    - 그렇지 않으면, DOMException 에러가 발생함
    - 두 번째 인수로 전달받은 노드가 `null`이면 첫 번째 인수로 전달받은 노드를 `insertBefore` 메서드를 호출한 노드의 마지막 자식 노드로 추가됨

### 노드 이동

- DOM에 이미 존재하는 노드를 `appendChild` 또는 `insertBefore` 메서드를 사용하여 DOM에 다시 추가하면 현재 위치에서 노드를 제거하고 새로운 위치에 노드를 추가함

    ```jsx
    // HTML
    /*
    <ul id="fruits">
    	<li>Apple</li>
    	<li>Banana</li>
    	<li>Orange</li>
    </ul>
    */
    
    // Javascript
    const $fruits = document.getElemenetById('fruits');
    
    // 이미 존재하는 요소 노드를 취득
    const [$apple, $banana, ] = $fruits.children;
    
    // 이미 존재하는 $apple 요소 노드를 #fruits 요소 노드의 마지막 노드로 이동
    $fruits.appendChild($apple); 
    // Banana - Orange - Apple
    
    // 이미 존재하는 $banana 요소 노드를 #fruits 요소의 마지막 자식 노드 앞으로 이동
    $fruits.insertBefore($banana, $fruits.lastElementChild);
    // Orange - Banana - Apple
    ```

### 노드 복사

- `Node.prototype.cloneNode([deep: true | false])`
  - 노드의 사본을 생성하여 반환
  - 매개변수 `deep` → `true`
    - 노드를 `깊은 복사`하여 `모든 자손 노드가 포함`된 사본을 생성
  - 매개변수 `deep` → `false`
    - 노드를 `얕은 복사`하여 `노드 자신`만의 사본을 생성

      ```jsx
      // HTML
      /*
      <ul id="fruits">
        <li>Apple</li>
      </ul>
      */
    
      // Javascript
      const $fruits = document.getElementById('fruits');
      const $apple = $fruits.firstElementChild;
    
      // $apple 요소를 얕은 복사하여 사본을 생성. 텍스트 노드가 없는 사본이 생성
      const $shallowClone = $apple.cloneNode();
      // 사본 요소 노드에 텍스트 추가
      $shallowClone.textContent = 'Banana';
      // 사본 요소 노드를 #fruits 요소 노드의 마지막 노드로 추가
      $fruits.appendChild($shallowClone);
    
      // #fruits 요소를 깊은 복사하여 모든 자손 노드가 포함된 사본을 생성
      const $deepClone = $fruits.cloneNode(true);
      // 사본 요소 노드를 #fruits 요소 노드의 마지막 노드로 추가
      $fruits.appendChild($deepClone);
      ```

### 노드 교체

- `Node.prototype.replaceChild(newChild, oldChild)`
  - 자신을 호출한 노드의 자식 노드를 다른 노드로 교체
  - 첫 번째 매개변수(`newChild`)
    - 교체할 `새로운 노드`를 인수로 전달
  - 두 번째 매개변수(`oldChild`)
    - 이미 존재하는 `교체될 노드`를 인수로 전달
  - `replaceChild` 메서드는 자신을 호출한 노드의 자식 노드인 `oldChild` 노드를 `newChild` 노드로 교체
    - oldChild 노드는 `DOM에서 제거`됨

    ```jsx
    //HTML
    /*
    <ul id="fruits">
      <li>Apple</li>
    </ul>
    */
        
    //Javascript
    const $fruits = document.getElementById('fruits');
        
    // 기존 노드와 교체할 요소 노드를 생성
    const $newChild = document.createElement('li');
    $newChild.textContent = 'Banana';
        
    // #fruits 요소 노드의 첫 번째 자식 요소 노드를 $newChild 요소 노드로 교체
    $fruits.replaceChild($newChild, $fruits.firstElementChild);
    ```

### 노드 삭제

- `Node.prototype.removeChild(child)`
  - child 매개변수에 인수로 전달한 노드를 DOM에서 `삭제`
    - 인수로 전달한 노드는 removeChild 메서드를 호출한 노드의 `자식 노드`여야 함

    ```jsx
    //HTML
    /*
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
    </ul>
    */
      
    //Javascript
    const $fruits = document.getElementById('fruits');
    
    // #fruits 요소 노드의 마지막 요소를 DOM에서 삭제
    $fruits.removeChild($fruits.lastElementChild);
    ```

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 710 ~ 733p
