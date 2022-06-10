---
title: load event와 DOMContentLoaded event
date: '2022-06-10'
tags: ['javascript']
draft: false
summary: HTML 문서의 생성 주기, DOMContentLoaded, onload Event
layout: PostSimple
authors: ['default']
---

# load event와 DOMContentLoaded event

## HTML 문서의 생성 주기

| 구분  | DOMContentLoaded                                             | load                                                                          | beforeunload                                    | unload                                         |
|-----|--------------------------------------------------------------|-------------------------------------------------------------------------------|-------------------------------------------------|------------------------------------------------|
| 발생  | 브라우저가 HTML을 전부 읽고 `DOM 트리를 완성하는 즉시 발생`                       | HTML로 DOM 트리를 만드는 게 완성되고 이미지, 스타일시트 같은 `외부 자원도 모두 불러오는 것이 끝났을 경우` 발생          | 사용자가 `페이지를 떠날 때 발생`                             | 사용자가 `페이지를 떠날 때 발생`                            |
| 활용  | DOM이 준비된 것을 확인한 후 원하는 DOM 노드를 찾아 핸들러를 등록해 `인터페이스를 초기화 할때 사용` | `이미지 사이즈 확인`할 때, 외부 자원이 로드된 후이므로 스타일이 적용된 상태에서 `화면에 뿌려지는 요소의 실제 크기를 확인`할 수 있음 | 사용자가 사이트를 떠나려 할 때, `변경되지 않은 사항들을 저장했는지 확인`시켜줄 때 | 사용자가 진짜 떠나기 전에 `사용자 분석 정보를 담은 통계자료를 전송`하고자 할 때 |

## DOMContentLoaded

- `document` 객체에서 발생
- `addEventListerner` 사용

```jsx
document.addEventListener("DOMContentLoaded", ready);
// "document.onDOMContentLoaded = ..." 는 동작하지 않음
```

### 예시

```html

<script>
  function ready() {
    alert('DOM이 준비되었습니다!');

    // 이미지가 로드되지 않은 상태이기 때문에 사이즈는 0x0
    alert(`이미지 사이즈: ${img.offsetWidth}x${img.offsetHeight}`);
  }

  document.addEventListener("DOMContentLoaded", ready);
</script>

<img id="img" src="https://media.giphy.com/media/HHljKdrD2eoaKDYhMw/giphy.gif?cache=0">
```

- 예제 실행 시, 캐시 방지를 위해서 브라우저내 시크릿모드로 실행
- `DOMContentLoaded` 핸들러는 문서가 로드되었을 때 실행됨
  - 이미지와 같은 외부 리소스가 로드될 때까지 기다리지 않으므로, 이미지 사이즈가 0으로 출력

### DOMContentLoaded와 scripts

- 브라우저는 HTML 문서를 처리하는 도중에  `<script>` 태그를 만나면, `DOM 트리 구성을 멈추고 <script>를 실행함`
- 스크립트실행이 완전히 끝난 후 HTML 문서 처리

    ```html
    <script>
      document.addEventListener("DOMContentLoaded", () => {
        alert("DOM이 준비되었습니다!");
      });
    </script>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.3.0/lodash.js"></script>
    
    <script>
      alert("라이브러리 로딩이 끝나고 인라인 스크립트가 실행되었습니다.");
    </script>
    ```

  - 최하단 “라이브러리 로딩이 끝나고…”가 먼저 보인후 “DOM이 준비되었습니다" 알럿이 출력되는 것을 볼 수 있는데 이는 스크립트가 모두 실행되고 난 후에 `DOMContentLoaded` 이벤트가 발생되는
    것을 확인할 수 있음
  - `예외 사항`
    - async 속성이 있는 스크립트는 `DOMContentLoaded`를 막지 않음
    - `document.createElement('script')`로 동적으로 생성되고 웹페이지에 추가된 스크립트는 DOMContentLoaded를 막지 않음

### DOMContentLoaded와 styles

- 외부 스타일시트는 DOM에 영향을 주지 않으므로 `DOMContentLoaded`는 외부 스타일시트가 로드되기를 기다리지 않음
- `예외 사항`

    ```html
    <link type="text/css" rel="stylesheet" href="style.css">
    <script>
    	// 해당 스크립트는 위 스타일시트가 로드될 때까지 실행되지 않음
    	alert(getComputedStyle(document.body).marginTop);
    </script>
    ```

  - `스타일시트를 불러오는 태그 바로 다음에 스크립트가 위치`하면 해당 스크립트는 스타일시트가 로드되기 전까지 실행되지 않음
  - 스크립트에서 요소의 좌표 정보 정보를 사용하므로, 적용되고 난 다음에야 좌표 정보가 확정되므로 제약이 생김

### 브라우저 내장 자동완성

- 특정 브라우저의 폼 자동완성은 `DOMContentLoaded`에서 일어남
- 스크립트가 길어서 `DOMContentLoaded` 이벤트가 지연된다면 자동완성 역시 뒤늦게 처리됨

## Window.onload

- `window` 객체의 `load` 이벤트는 스타일, 이미지 등의 리소스들이 모두 로드되었을 때 실행됨

    ```html
    <script>
      window.addEventListener('load', (event) => {
        alert('페이지 전체가 로드되었습니다.');
    
        // 이번엔 이미지가 제대로 불러와 진 후에 얼럿창이 실행됩니다.
        alert(`이미지 사이즈: ${img.offsetWidth}x${img.offsetHeight}`);
      });
    </script>
      
    <img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
    ```

  - 이미지가 모두 로드되고 난 후 실행되므로 이미지 사이즈가 얼럿에서 제대로 출력됨

> **Referenced**

- [모던 JavaScript 튜토리얼](https://ko.javascript.info/onload-ondomcontentloaded)
