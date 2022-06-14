---
title: 브라우저의 렌더링 과정
date: '2022-06-14'
tags: ['javascript']
draft: false
summary:
layout: PostSimple
authors: ['default']
---

# 브라우저의 렌더링 과정

## 등장 배경 및 개요

- V8 자바스크립트 엔진으로 빌드된 자바스크립트 런타임 환경인 Node.js의 등장으로 웹 브라우저를 벗어나 서버 사이드 애플리케이션 개발에서도 사용할 수 있는 범용 언어 개발이 되었음
  - 대부분의 언어는 운영체제나 가상 머신 위에서 실행되면 웹 애플리케이션의 클라이언트 사이드 자바스크립트는 브라우저에서 HTML, CSS와 함께 실행됨

## 파싱(parsing)

### 과정

- 프로그래밍 언어의 문법에 맞게 작성된 텍스트 문서를 읽어 들여 실행하기 위해 텍스트 문서의 문자열을 `토큰으로 분해하고`, 토큰에 문법적 의미와 구조를 반영하여 트리 구조의 자료구조인 `파스 트리를 생성`하는
  일련의 과정

### 실행

- 파스트리를 기반으로 중간 언어인 바이트코드를 생성하고 실행함

## 렌더링(rendering)

- HTML, CSS, 자바스크립트로 작성된 문서를 파싱하여 브라우저에 시각적으로 출력하는 것

## 렌더링 과정

1. `브라우저`는 HTML, CSS, 자바스크립트, 이미지, 폰트 파일 등 `렌더링에 필요한 리소스를 요청`하고 `서버로부터 응답` 받음
2. `렌더링 엔진`은 서버로부터 응답된 HTML과 CSS를 `파싱`하여 `DOM과 CSSOM을 생성`하고 이들을 결합하여 `렌더 트리를 생성`
3. 브라우저의 `자바스크릡트 엔`진은 서버로부터 응답된 자바스크립트를 파싱하여 `AST를 생성`하고 `바이트코드로 변환`하여 `실행`

- DOM API를 통해 DOM이나 CSSOM을 변경할 수 있고 변경된 DOM과 CSSOM은 다시 렌더 트리로 결합됨

4. 렌더 트리를 기반으로 HTML 요소의 `레이아웃을 계산`하고 브라우저 화면에 `HTML 요소를 페인팅`함

## 요청과 응답

- 브라우저의 핵심 기능
  - 필요한 `리소스`를 `서버에 요청`
  - 서버로부터 `응답`
  - 시각적으로 `렌더링`

- Q. 개발자 도구의 Network 패널을 활성화 했을 때, 요청도 하지 않은 리소스가 응답된 이유?
  - A. HTML을 파싱하는 도중에 `외부 리소스를 로드하는 태그`(link, img, script)를 만나면 파싱을 중단하고 `해당 리소스 파일을 서버로 요청`하기 때문임(⭐)

## HTTP 1.1과 HTTP 2.0

- HTTP는 웹에서 브라우저와 서버가 통신하기 위한 프로토콜 규약

### HTTP 1.1

- 커넥션당 `하나의 요청과 응답`만 처리
  - 여러 개의 요청을 한 번에 전송 및 응답할 수 없음
  - 리소스 요청은 개별적으로 전송되고 응답 또한 `개별적으로 전송됨`
  - 동시 전송이 불가능한 구조

### HTTP 2

- 커넥션당 `여러 개의 요청과 응답`이 가능
  - 동시 전송 가능
- HTTP/1.1에 비해 페이지 로드 속도가 50% 정도 빠름

## HTML 파싱과 DOM 생성

- 순수한 텍스트인 HTML 문서를 브라우저에 시각적인 픽셀로 렌더링하려면 `HTML 문서를 브라우저가 이해할 수 있는 자료구조로 변환하여 메모리에 저장`해야 함

    ```html
    <!DOCTYPE html>
    <html>
    	<head>
    		<meta charset="UTF-8">
    		<link rel="stylesheet" href="style.css">
    	</head>
    	<body>
    		<ul>
    			<li id="apple">Apple</li>
    			<li id="banana">Banana</li>
    			<li id="orange">Orange</li>
    		</ul>
    		<script src="app.js"></script>
    	</body>
    </html>
    ```

  - 응답받은 HTML 문서를 파싱하여 브라우저가 이해할 수 있는 자료구조인 DOM을 생성

  ![javascript_browser_rendering_1](/static/images/javascript_browser_rendering_1.png)

  1. 서버에 존재하던 HTML 파일이 브라우저 요청에 의해 응답

  - `서버`는 브라우저가 요청한 `HTML 파일`을 읽어 들여 `메모리에 저장`한 다음 메모리에 저장된 `바이트`를 인터넷 경유하여 `응답`

  2. `브라우저`는 서버가 응답한 HTML 문서를 바이트 형태로 응답

  - 바이트 형태의 문서는 `meta` 태그의 `charset` 어트리뷰트에 의해 지정된 `인코딩` 방식(UTF-8) `문자열로 변환`

  3. 문자열로 변환된 HTML 문서를 읽어들여 문법적 의미를 갖는 코드의 최소 단위인 `토큰들로 분해`
  4. 토큰 → 객체로 변환하여 `노드` 생성
  5. 중첩 관계를 갖는 HTML 요소의 부자 관계 형성(트리 자료구조화) - DOM 생성

## CSS 파싱과 CSSOM 생성

- 렌더링 엔진은 HTML 을 처음부터 한 줄씩 순차적으로 파싱하여 DOM을 생성하다가 `외부 리소스를 로드(link, style)하는 태그를 만나면 DOM 생성을 일시 중단`
  - HTML와 동일한 파싱 과정을 거치며 해석하여 `CSSOM을 생성`
- CSS 파싱 완료 후 HTML 파싱이 중단된 시점부터 `DOM 생성 재개`

```css
body {
    font-size: 18px;
}

ul {
    list-style-type: none;
}
```

- 외부 리소스를 로드하는 태그(`link`)를 만나서 위와 같은 style.css 파일이 서버로부터 응답이 되었다고 가정
- 서버로부터 CSS 파일이 응답되면 렌더링 엔진은 HTML과 동일한 해석 과정을 거쳐 CSSOM을 생성함

  <img alt="javascript_browser_rendering_2" src="/static/images/javascript_browser_rendering_2.png" width="500"/>

## 렌더 트리 생성

- `DOM`과 `CSSOM`은 렌더링을 위해 `렌더 트리로 결합`됨
- `렌더 트리`
  - 렌더링을 위한 `트리 구조`의 자료구조
  - 브라우저 화면에 `렌더링되는 노드`만으로 구성
  - HTML 요소의 `레이아웃을 계산하는 데 사용`되며 브라우저 화면에 픽셀을 렌더링하는 `페인팅 처리`에 입력됨

    <img alt="javascript_browser_rendering_3" src="/static/images/javascript_browser_rendering_3.png" width="700"/>

  - `리렌더링` 발생되는 경우
    - 자바스크립트에 의한 `노드 추가` 또는 `삭제`
    - 브라우저 창의 리사이징에 의한 `뷰포트 크기 변경`
    - `HTML 요소의 레이아웃`(위치, 크기)에 변경을 발생시키는 `width/height`, `margin`, `padding`, `border`, `display`, `position`
      , `top/right/bottom/left` 등의 `스타일 변경`

    > `레이아웃 계산`과 `페인팅`이 재차 발생하면 `성능에 악영향`을 주는 작업이므로 피해야 함

>

## 자바스크립트 파싱과 실행

- HTML 문서를 파싱한 결과물로서 생성된 `DOM`은 HTML 문서의 구조와 정보뿐만 아니라 HTML 요소와 스타일 등을 변경할 수 있는 `프로그래밍 인터페이스`로서 `DOM API`를 제공함
  - DOM API를 통해 DOM을 동적으로 조작할 수 있음
- 자바스크립트 `파싱`과 `실행`은 브라우저의 렌더링 엔진이 아닌 `자바스크립트 엔진이 처리함`
  - 자바스크립트 엔진은 ECMAScript 사양을 준수
- `렌더링 엔진`이 HTML과 CSS를 `파싱`하여 DOM과 CSSOM을 생성하듯이 `자바스크립트 엔진`은 자바스크립트를 해석하여 `AST(추상적 구문 트리)를 생성`
  - `AST`를 기반으로 인터프리터가 실행할 수 있는 중간 코드인 `바이트코드를 생성하여 실행`

![javascript_browser_rendering_4](/static/images/javascript_browser_rendering_4.png)

### 토크나이징

- 자바스크립트 소스코드를 어휘 분석하여 문법적 의미를 갖는 `코드의 최소 단위`인 `토큰`들로 분해

### 파싱

- 토큰들의 집합을 `구문 분석`하여 `AST`(추상적 구문 트리)를 생성
  - AST를 사용하면 `Typescript`, `Babel`, `Prettier` 같은 `트랜스파일러` 구현을 할 수 있음
  - [AST Explorer 웹사이트](https://astexplorer.net) - AST 생성

### 바이트코드 생성과 실행

- `AST`는 인터프리터가 실행할 수 있는 중간 코드인 `바이트코드로 변환`되고 `인터프리터`에 의해 실행
  - V8 엔진의 경우 자주 사용되는 코드는 `터보팬`이라 불리는 `컴파일러`에 의해 최적화된 `머신 코드로 컴파일되어 성능을 최적화`

## 리플로우와 리페인트

- `DOM API`가 사용되어, 변경된 `DOM`과 `CSSOM`은 다시 렌더 트리로 결합되고 변경된 렌더 트리를 기반으로 레이아웃과 페인트 과정을 거쳐 `브라우저의 화면에 다시 렌더링`되는 흐름
- `리플로우`
  - `레이아웃 계산`을 다시 하는 것
- `리페인트`
  - `재결합된 렌더 트리`를 기반으로 다시 `페인트` 하는 것

## 자바스크립트 파싱에 의한 HTML 파싱 중단

<img alt="javascript_browser_rendering_5" src="/static/images/javascript_browser_rendering_5.png" width="700"/>

- 브라우저는 동기적으로, 위에서 아래 방향으로 순차적으로 HTML, CSS, 자바스크립트를 파싱하고 실행함
- `script 태그의 위치`에 따라 HTML 파싱이 블로킹되어 `DOM 생성이 지연`될 수 있음(⭐⭐)
  - HTML 파싱이 중단되는 위치(app.js)에서 DOM이나 CSSOM을 변경하는 `DOM API을 사용할 경우 DOM이나 CSSOM이 이미 생성되어 있어야 함`
    - DOM이 완성되지 않은 상태에서 자바스크립트가 DOM을 조작하면 에러가 발생할 수 있음
    - 자바스크립트 `로딩`/`파싱`/`실행`으로 인해 HTML 요소들의 렌더링에 지장받는 일이 발생하지 않아 페이지 로딩 시간이 단축됨

## script 태그의 async/defer 어트리뷰트

- 자바스크립트 파싱에 의한 DOM 생성 중단 문제를 근본적으로 해결하기 위해 HTML5부터 script 태그에 `async`와 `defer` 어트리뷰트가 추가됨
  - src 어트리뷰트가 없는 인라인 자바스크립트에서는 사용할 수 없음

```html

<script async src="extern.js"></script>
<script defer src="extern.js"></script>
```

### async 어트리뷰트

<img alt="javascript_browser_rendering_6" src="/static/images/javascript_browser_rendering_6.png" width="700"/>

- HTML 파싱과 외부 자바스크립트 파일의 로드가 비동기 적으로 동시에 진행됨
- `IE10 이상`에서 지원

### defer 어트리뷰트

<img alt="javascript_browser_rendering_7" src="/static/images/javascript_browser_rendering_7.png" width="700"/>

- async 어트리뷰트와 마찬가지로 진행되지만 `DOM 생성이 완료된 직후에 진행`되므로 DOM 생성이 완료된 이후 실행되어야 할 자바스크립트에 유용함
- `IE10 이상`에서 지원

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 643 ~ 659p
