---
title: DOM - 노드 탐색과 노드 정보 취득
date: '2022-06-15'
tags: ['javascript']
draft: false
summary: HTML 문서의 계층적 구조와 정보를 표현하며 이를 제어할 수 있는 API, 프로퍼티와 메서드를 제공하는 트리 자료구조
layout: PostSimple
authors: ['default']
---

# DOM - 노드 탐색과 노드 정보 취득

- HTML 문서의 `계층적 구조`와 `정보`를 표현하며 이를 제어할 수 있는 `API`, `프로퍼티`와 `메서드`를 제공하는 `트리 자료구조`

## 노드

### HTML 요소와 노드 객체

- HTML 요소
  - `HTML`을 구성하는 `개별적인 요소`
  - `렌더링 엔진`에 의해 `파싱`되어 DOM을 구성하는 요소 `노드 객체로 변환`됨
- `중첩 관계`에 의해 `계층적인 부자 관계`가 형성
- HTML 문서의 구성 요소인 HTML 요소를 객체화한 `모든 노드 객체들을 트리 자료 구조로 구성`

### 트리 자료구조

- 부모 노드와 자식 노드로 구성되어 노드 간의 `계층적 구조`(부자, 형제 관계)를 표현하는 `비선형 자료 구조`
- 노드 객체들로 구성된 트리 자료를 `DOM`이라고 함

  <img alt="javascript_dom_1" src="/static/images/javascript_dom_1.png" width="400"/>

  - 노드 객체의 트리로 구조화되어 있으므로, `DOM 트리`라고 부름

### 노드 객체의 타입

`문서 노드`

- `DOM 트리의 최상위`에 존재하는 `루트 노드`로서 document 객체를 가리킴
- 브라우저가 렌더링한 `HTML 문서 전체를 가리키는 객체`로서 전역 객체 window의 document 프로퍼티에 바인딩되어 있음

> HTML 문서당 document 객체는 유일하며, `요소`, `어트리뷰트`, `텍스트 노드`에 접근하려면 문서 노드를 통해야 함

`요소 노드`

- `HTML 요소`를 가리키는 객체로서, HTML 요소 간의 중첩에 의해 `부자 관계`를 가지며, 부자 관계를 기반해 정보를 구조화함

`어트리뷰트 노드`

- HTML 요소의 `어트리뷰트`를 가리키는 객체
- `부모 노드가 없으므로` 요소 노드의 형제 노드는 아님

`텍스트 노드`

- HTML 요소의 `텍스트`를 가리키는 객체
- `문서의 정보를 표현`하는 것이 목표
- `DOM 트리의 최종단`이므로 텍스트 노드에 접근하려면 부모 노드인 요소 노드에 접근해야 함

### 노드 객체의 상속 구조

- `노드 객체`는 자신의 `구조`와 `정보`를 제어할 수 있는 `DOM API`를 사용할 수 있음
  - 브라우저 환경에서 추가적으로 제공하는 `호스트 객체`임

  ![javascript_dom_2](/static/images/javascript_dom_2.png)

  - 모든 노드 객체는 `Object`, `EventTarget`, `Node` 인터페이스를 상속받음
  - `문서` 노드
    - `Document`, `HTMLDocument` 인터페이스를 상속 받음
  - `어트리뷰트` 노드
    - `Attr` 인터페이스를 상속 받음
  - `텍스트` 노드
    - `CharacterData` 인터페이스를 상속 받음
  - `요소` 노드
    - `Element` 인터페이스를 상속받음

## 요소 노드 취득

- HTML의 구조나 내용 또는 스타일 등을 `동적으로 조작`하기 위한 요소 노드 취득을 제공

### id를 이용한 요소 노드 취득

- `Document.prototype.getElementID`
  - id 값은 HTML 문서 내에서 `유일한 값`
  - class 어트리뷰트와는 달리 `공백 문자로 구분`하여 언제나 `단 하나`의 요소 노드를 반환함
- 요소가 존재하지 않는 경우
  - `null` 반환
- HTML 요소에 id 어트리뷰트를 부여하면 id 값과 동일한 이름의 `전역 변수가 암묵적으로 선언 및 할당됨`(`사이드이펙트`)
  - 동일한 이름의 전역 변수가 `이미 선언`되어 있으면 노드 객체가 `재할당되지 않음`

### 태그 이름을 이용한 요소 노드 취득

`Document.prototype`/`Element.prototype.getElementsByTagName`

- 인수로 전달한 `태그 이름`을 갖는 모든 요소 노드들을 탐색하여 반환
- 여러 개의 요소 노드 객체를 갖는 `HTMLCollection` 객체 반환
  - `HTMLCollection` - 유사 배열 객체이면서 이터러블 함
- 요소가 존재하지 않은 경우
  - `빈 HTMLCollection` 반환

### class를 이용한 요소 노드 취득

`Document.prototype`/`Element.prototype.getElementsByClassName`

- 인수로 전달한 `class 어트리뷰트 값`을 갖는 모든 요소 노드들을 탐색하여 반환
  - 인수로 전달할 값은 `공백으로 구분`하여 `여러 개의 class를 지정`할 수 있음
- 여러 개의 요소 노드 객체를 갖는 `HTMLCollection` 객체 반환
- 요소가 존재 하지 않은 경우
  - `빈 HTMLCollection` 반환

### CSS 선택자를 이용한 요소 노드 취득

- CSS 선택자는 스타일을 적용하고자 하는 `HTML 요소를 특정`할 때 사용

`Document.prototype`/`Element.prototype.querySelector`

- 요소 노드가 `여러 개`인 경우 → `첫번째 요소 노드`만 반환
- 요소 노드 `존재하지 않는` 경우  → `null`을 반환
- 문법에 맞지 않는 경우 → `DOMException` 에러 발생

`Document.prototype`/`Element.prototype.querySelectorAll`

- 여러 개의 노드 객체를 갖는 DOM 컬렉션 객체인 `NodeList 객체` 반환
  - `NodeList` - 유사 배열 객체이면서 이터러블
- 요소가 존재하지 않는 경우 → `빈 NodeList 객체` 반환
- 문법에 맞지 않는 경우 → `DOMException` 에러 발생

> `querySelector`, `querySelectorAll` 메서드는 `getElementById`, `getElementsBy***` 메서드보다 다소 느린 경우가 있지만 구체적인 조건으로 요소 노드를 취득할 수 있고 일관된 방식으로 요소 노드를 취득할 수 있음
>

### 특정 요소 노드를 취득할 수 있는지 확인

`Element.prototype.matches`

- 인수로 전달한 CSS 선택자를 통해 `특정 요소 노드를 취득할 수 있는지` 확인

### HTMLCollection과 NodeList

- DOM API가 여러 개의 결과값을 반환하기 위한 DOM 컬렉션 객체
- 유사 배열 객체이면서 이터러블이므로 `for…of` 문으로 순회 가능
- 노드 객체의 상태 변화를 `실시간`으로 반영하는 `살아 있는 객체`

`HTMLCollection`

- 노드 객체의 `상태 변화를 실시간으로 반영`하는 살아 있는 DOM 컬렉션 객체
- HTMLCollection 객체는 실시간으로 노드 객체의 상태 변경을 반영하여 요소를 제거하므로, HTMLCollection 객체를 직접 사용해서 수정하지 않고 배열의 `고차 함수 사용`(`forEach`, `map`, `filter`, `reduce` 등)을 권장

`NodeList`

- HTMLCollection 객체의 부작용을 해결하기 위해 `querySelectorAll` 메서드를 사용하는 방법이 있음
- 대부분의 경우 non-live 객체로 동작함
- `childNodes` 프로퍼티가 반환하는 NodeList 객체는 HTMLCollection 객체와 같이 `실시간으로 노드 객체의 상태 변경을 반영`하는 live 객체로 동작함

> 노드 객체의 상태 변경과 상관없이 안전하게 DOM 컬렉션을 사용하려면 `HTMLCollection`이나 `NodeList` 객체를 `배열로 변환하여 사용`하는 것을 권장함(`스프레드` 문법, `Array.from` 메서드)

## 노드 탐색

- DOM 트리 상의 노드를 탐색할 수 있도록 Node, Element 인터페이스는 `트리 탐색 프로퍼티`를 제공함
  - 접근자 프로퍼티로서, `getter`만 존재하여 `참조만 가능`함

<img alt="javascript_dom_3" src="/static/images/javascript_dom_3.png" width="600"/>

- Node.prototype
  - `parentNode`, `previousSibling`, `firstChild`, `childNodes`
- Element.prototype
  - `previousElementSibling`, `nextElementSibling`, `children`

### 자식 노드 탐색

- 자식 노드를 탐색하기 위해서는 다음과 같은 노드 탐색 프로퍼티를 사용함

| 프로퍼티                                  | 설명                                                                                                                     |
|---------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| Node.prototype.`childNodes`           | 자식 노드를 모두 탐색하여 DOM 컬렉션 객체인 `NodeList`에 담아 반환함.<br/>childNodes 프로퍼티가 반환된 NodeList에는 노드뿐만 아니라 텍스트 노드도 포함되어 있을 수 있음       |
| Element.prototype.`children`          | 자식 노드 중에서 요소 노드만 모두 탐색하여 DOM 컬렉션 객체인 `HTMLCollection`에 담아 반환함.<br/>children 프로퍼티가 반환한 HTMLCollection에는 텍스트 노드가 포함되지 않음 |
| Node.prototype.`firstChild`           | `첫 번째 자식 노드`를 반환함.<br/>firstChild 프로퍼티가 반환한 노드는 텍스트 노드이거나 요소 노드임                                                       |
| Element.prototype.`lastChild`         | `마지막 자식 노드`를 반환함.<br/>lastChild 프로퍼티가 반환한 노드는 텍스트 노드이거나 요소 노드임                                                         |
| Element.prototype.`firstElementChild` | `첫 번째 자식 요소 노드`를 반환함.<br/>firstElementChild 프로퍼티는 요소 노드만 반환함                                                           |
| Element.prototpe.`lastElementChild`   | `마지막 자식 요소 노드`를 반환함.<br/>lastElementChild 프로퍼티는 요소 노드만 반환함                                                             |

### 자식 노드 존재 확인

- 자식 노드가 존재하는지 확인하려면 `Node.prototype.hasChildNodes` 메서드를 사용
  - 자식 노드가 존재할 경우
    - true
  - 자식 노드가 존재하지 않는 경우
    - false
- `텍스트 노드를 포함`하여 자식 노드의 존재를 확인함
- 요소 노드가 존재하는지 확인하려면 `children.length` 또는 Element 인터페이스의 `childElementCount` 프로퍼티 사용

### 부모 노드 탐색

- `Node.prototype.parentNode` 프로퍼티 사용

### 형제 노드 탐색

| 프로퍼티                                       | 설명                                                                     |
|--------------------------------------------|------------------------------------------------------------------------|
| Node.prototype.`previousSibling`           | 부모 노드가 같은 형제 노드 중에서 자신의 `이전 형제 노드`를 탐색하여 반환.<br/>요소 노드뿐만 아니라 텍스트 노드 반환 |
| Node.prototype.`nextSibling`               | 부모 노드가 같은 형제 노드 중에서 자신의 `다음 형제 노드`를 탐색하여 반환.<br/>요소 노드뿐만 아니라 텍스트 노드 반환 |
| Element.prototype.`previousElementSibling` | 부모 노드가 같은 형제 노드 중에서 자신의 `이전 형제 요소 노드`를 탐색하여 반환.<br/>프로퍼티는 요소 노드만 반환    |
| Element.prototype.`nextElementSibling`     | 부모 노드가 같은 형제 노드 중에서 자신의 `다음 형제 요소 노드`를 탐색하여 반환.<br/>프로퍼티는 요소 노드만 반환    |

## 노드 정보 취득

- Node.prototype.nodeType
  - 노드 객체의 종류, 즉 노드 타입을 나타내는 `상수`를 반환
  - 노드 타입 상수는 Node에 정의됨
    - Node.`ELEMENT_NODE`: 요소 노드 타입을 나타내는 `상수 1`을 반환
    - Node.`TEXT_NODE`: 텍스트 노드 타입을 나타내는 `상수 3`을 반환
    - Node.`DOCUMENT_NODE`: 문서 노드 타입을 나타내는 `상수 9`을 반환
- Node.prototype.nodeName
  - 노드의 이름을 `문자열`로 반환                                                                                                                                                                           
    - `요소` 노드: 대문자 문자열로 태그 이름("UL", "LI" 등)을 반환
    - `텍스트` 노드: 문자열 `#text`를 반환
    - `문서` 노드: 문자열 `#document`를 반환

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 677 ~ 709p
