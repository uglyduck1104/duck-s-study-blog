---
title: 이벤트 위임(event delegation)
date: '2022-06-08'
tags: ['javascript']
draft: false
summary: 캡처링과 버블링을 활용해 강력한 이벤트 핸들링 패턴인 이벤트 위임(event delegation)을 구현
layout: PostSimple
authors: ['default']
---

# 이벤트 위임

- `캡처링`과 `버블링`을 활용해 강력한 이벤트 핸들링 패턴인 `이벤트 위임(event delegation)` 을 구현
- `여러 요소를 비슷한 방식으로 다뤄야 할 때` 사용
  - 주로 캡쳐링과 버블링을 이용해 구현
- `이벤트 위임`을 사용하면 요소마다 핸들러를 할당하지 않고, 요소의 `공통 조상에 이벤트 핸들러를 단 하나만 할당`해도 여러 요소를 한꺼번에 다룰 수 있음
  - `event.target`을 이용해 이벤트 발생 확인하기

## 특정 영역을 클릭하면 영역 강조하는 이벤트 구현

<img alt="javascript_event_delegation2" src="/static/images/javascript_event_delegation2.png" width="300"/>

### HTML

```html

<table id="tableEl">
  <tr>
    <td>
      <strong>Fruit</strong>
      <p>...</p>
    </td>
    <td>
      <strong>Color</strong>
      <p>...</p>
    </td>
    <td>
      <strong>City</strong>
      <p>...</p>
    </td>
  </tr>
</table>
```

- 특정 영역 `<td>`을 클릭하면 노란색 박스로 해당 칸을 강조하기 위한 table element 마크업
- 각 `<td>` 마다 onclick 핸들러를 할당하지 않고, `<table>` 요소에 할당
  - `event.target`을 통해 현재 이벤트의 target이 무엇인지 감지

### Javascript - 문제 인식

```jsx
let selectedTd;
const table = document.getElementById('tableEl');
table.addEventListener('click', (e) => {
  let target = e.target;  // 클릭 발생 확인

  if (target.tagName != 'TD') return; // tagName이 TD에서 발생하지 않으면 이벤트 종료

  highlight(target);
})

function highlight(td) {
  if (selectedTd) { // 이미 강조되어있는 칸이 있을 경우 원복
    selectedTd.classList.remove('highlight');
  }
  selectedTd = td;
  selectedTd.classList.add('highlight'); // 새로운 td 강조
}
```

- 예제와 같이 코드를 작성하면 클릭 이벤트가 `<td>`가 아닌 `<td> 안에서` 동작할 수 있음
- html 안을 살펴보면 `<td>`안에 하위 태그에 `<p>`, `<strong>` 태그를 볼 수 있는데, 이는 `event.target`을 이용해서 클릭 이벤트가 일어나는 곳이 어디인지 확인해야 함

  <img alt="javascript_event_delegation1" src="/static/images/javascript_event_delegation1.png" width="400"/>

### Javascript - 문제점 보완

```jsx
table.addEventListener('click', (e) => {
  let target = e.target.closest('td'); // (1)

  if (!target) return; // (2)

  if (!table.contains(target)) return; // (3)

  highlight(target); // (4)
})
```

1. `elem.closet(selector)`

- `elem`의 상위 요소 중에 `selector`와 일치하는 가장 근접한 조상 요소 반환
- 이벤트가 발생한 요소부터 시작해 위로 올라가면서 가장 가까운 `<td>` 요소를 찾음

2. `event.target`이 `<td>`안에 있지 않으면 `null`을 반환
3. 중첩 테이블이 있는 경우 `event.target`은 테이블 바깥에 있는 `<td>`가 될 수 있으므로 해당 경우를 처리
4. `highlight` 함수 실행

## 이벤트 위임 활용

- 여러 버튼이 있는 메뉴를 구현할 때, 각 버튼의 기능에 대한 메서드를 연결하고자 할때 보통 버튼에 독립된 핸들러를 할당하는 방법이 떠오르지만 이벤트 위임을 활용해 더 좋은 방식으로 구현할 수 있음

- 메뉴 전체에 핸들러를 추가한 후, 각 버튼의 `data-action` 속성에 호출할 메서드를 할당

    ```html
    <div id="menu">
        <button data-action="save">저장</button>
        <button data-action="load">로드</button>
        <button data-action="search">검색</button>
    </div>
    <script>
      class Menu {
        constructor(elem){
          this._elem = elem;
          elem.onclick = this.onClick.bind(this);
        }
    
        save() {
          alert('저장하기');
        }
    
        load() {
          alert('불러오기');
        }
    
        search() {
          alert('검색하기');
        }
    
        onClick(event){
          let action = event.target.dataset.action;
          if(action) {
            this[action]();
          }
        }
      }
    
      new Menu(menu);
    </script>
    ```

  - this는 `호출될 때 결정`되므로, this가 `Menu 객체`를 가리키도록 `bind` 메서드를 통해 전달
    - 버튼마다 핸들러를 할당해주는 코드를 작성할 필요가 없으므로 단지 메서드를 만든 후 HTML에 그 메서드를 써주면 됨
    - 언제든지 버튼을 추가하고 제거할 수 있어 `HTML 구조가 유연해짐`

  ## `행동` 패턴 사용하기

  - 선언적 방식으로 행동을 추가할 때 사용할 수 있음
  - 행동 패턴 구성
    1. 요소의 행동을 설명하는 `커스텀 속성을 요소에 추가`
    2. 문서 전체를 감지하는 `핸들러가 이벤트를 추적`

  ### 카운터 구현

  - 버튼 클릭하면 숫자가 증가

    ```html
    첫 번째 카운터: <input type="button" value="1" data-counter>
    두 번째 카운터: <input type="button" value="2" data-counter>
    
    <script>
      document.addEventListener('click', (event) => {
        if(event.target.dataset.counter != undefined) {
          event.target.value++;
        }
      })
    </script>
    ```

  - 접근 방식에 집중하기
  - `data-counter` 속성이 있는 요소를 이벤트 위임을 사용해 새로운 행동을 선언해주는 속성을 추가해 HTML `확장`

  > `document 객체`에 핸들러를 할당할 때에는 `addEventListner`를 사용해야 함\
  > `document.onclick`은 새로운 핸들러가 `이전의 핸들러를 덮어 쓸 수도 있음`(주의!!!)

  ### 토글러 구현

  - `data-toggle-id` 속성이 있는 요소를 클릭하면 속성값이 id인 요소가 사라지게 하기

    ```html
    <button data-toggle-id="subscribe-mail">구독 폼 보여주기</button>
    <form id="subscribe-mail" hidden>
      메일 주소: <input type="email" />
    </form>
    <script>
      document.addEventListener('click', (event) => {
        let id = event.target.dataset.toggleId;
        if(!id) return;
    
        let elem = document.getElementById(id);
    
        elem.hidden = !elem.hidden;
      });
    </script>
    ```

  - 자바스크립트에 의존하지 않고 태그에 `data-toggle-id` 속성을 추가하여 해당 요소를 토글할 수 있음
  - `행동 패턴을 응용`하여 토글 기능이 필요한 요소 전체에 자바스크립트로 해당 기능을 구현하지 않아도 되므로 매우 편리함

  > 자바스크립트 미니 프래그먼트의 대안이 될 수 있음

  ## 정리

  ### 이벤트 위임의 장점

  - 많은 핸들러를 할당하지 않아도 되므로 `초기화가 단순해지고 메모리 절약`
  - 해당 요소에 할당된 핸들러를 추가하거나 제거할 필요가 없으므로 `코드가 짧아짐`
  - `innerHTML`이나 유사한 기능을 하는 스크립트로 DOM 수정이 쉬워짐

  ### 이벤트 위임의 단점

  - 이벤트 위임은 `반드시 버블링 되어야 함`
  - 컨테이너 수준에 할당된 핸들러가 응답할 필요가 있는 이벤트든 아니든 `모든 하위 컨테이너에서 발생하는 이벤트에 응답`해야 하므로 CPU 작업 부하가 늘어 날 수 있음(`실제로 미비함`)

> **Referenced**

- [모던 JavaScript 튜토리얼](https://ko.javascript.info/event-delegation#tasks)
