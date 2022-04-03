---
title: 자바스크립트
date: '2022-04-03'
tags: ['javascript']
draft: false
summary: 자사 브라우저의 시장 점유율을 점유하기 위해 자사 브라우저에서만 동작하는 기능을 경쟁적으로 추가하기 시작함
layout: PostSimple
authors: ['default']
---

# 표준화

- 자사 브라우저의 시장 점유율을 점유하기 위해 자사 브라우저에서만 동작하는 기능을 경쟁적으로 추가하기 시작함

  > 이로 인해, 브라우저에 따라 웹 페이지가 정상 동작하지 않는 `크로스 브라우징 이슈`가 발생했고 모든 부라우저에서 동작하는 웹 페이지를 개발하는 것은 무척 어려워짐

- 1996년 11월 넷스케이프 커뮤니케이션즈는 컴퓨터 시스템의 표준을 관리하는 비영리 표준화 기구인 ECMA 인터내셔널에 자바스크립트 표준화를 요청함

# ECMA Script 버전별 특징표

| 태그                 | 출시 | 특징                                                                                                                                                                   |
| -------------------- | :--: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ES1                  | 1997 | 초판                                                                                                                                                                   |
| ES2                  | 1998 | ISO/IEC 16262 국제 표준과 동일한 규격 적용                                                                                                                             |
| ES3                  | 1999 | 정규 표현식, try...catch 예외 처리                                                                                                                                     |
| ES5                  | 2009 | HTML5와 함께 출현한 표준안. JSON, strict mode, 접근자 프로퍼티(getter, setter), 향상된 배열 조작 기능(forEach, map, filter, reduce, some, every)                       |
| ES6(ECMAScript 2015) | 2015 | let, const, class, error function(⇒), 템플릿 리터럴, 디스트럭처링 할당, spread 문법, rest 파라미터, Symbol, Promise, Map/Set, iterator/generator, module import/export |
| ES7(ECMAScript 2016) | 2016 | 지수(\*\*) 연산자, Array.prototype.includes, String.prototype.includes                                                                                                 |
| ES8(ECMAScript 2017) | 2017 | async/await, Object 정적 메소드(Object.values, Object.entries, Object.getOwnPropertyDescriptors)                                                                       |
| ES9(ECMAScript 2018) | 2018 | Object Rest/Spread 프로퍼티                                                                                                                                            |

# Ajax의 등장

- 이전의 웹페이지
  - 서버로부터 완전한 HTML을 전송받아 웹 페이지 전체를 렌더링 하는 방식으로 동작
  - 웹 페이지를 처음부터 모든 요소들을 다시 렌더링하는 것은 불필요한 데이터 통신이 발생, 퍼포먼스 측면에서도 불리한 방식임
- 현재의 웹페이지
  - 서버로부터 필요한 데이터만 전송 받아서 변경하는 부분만 한정적으로 렌더링
    - 빠른 퍼포먼스와 부드러운 화면전환 가능

# Node.js의 등장

- 2009년 자바스크립트 실행환경인 Node.js의 등장으로 웹 브라우저에 한정적이었던 범위가 서버 사이드 애플리케이션 개발까지 확장되는 범용 프로그래밍 언어가 됨
- 현재 백엔드 영역까지 아우르는 웹 프로그래밍 언어의 표준

# Javascript와 ECMAScript

- ECMAScript
  - 자바스크립트의 표준 명세
  - 프로그래밍 언어의 타입, 값, 객체와 프로퍼티, 함수, 빌트인 객체 등 핵심 문법을 규정
- 자바스크립트
  - ECMAScript와 브라우저가 별도 지원하는 클라이언트 사이드 Web API, 즉 DOM, BOM, Canvas, XMLHttpRequest, Fetch, requestAnimationFrame, SVG, WebStorage, Web Component, Web worker 등을 아우르는 개념

## 자바스크립트의 특징

- 웹 브라우저에서 동작하는 유일한 프로그래밍 언어
- C, Java와 유사하고 Self에서는 프로토타입 기반 상속, Scheme에서는 일급 함수의 개념 차용
- `인터프리터 언어`
  - 개발자가 별도의 컴파일 작업을 수행하지 않음
  - 소스코드를 즉시 실행하고 컴파일러는 빠르게 동작하는 머신 코드를 생성하고 최적화 함
- 명령형(imperative), 함수형(functional), `프로토타입 기반(prototype-based)` 객체지향 프로그래밍 지원
  - `객체 지향 프로그래밍`
    - 상속, 정보 은닉을 위한 키워드가 없어서 오해
    - 클래스 기반 객체지향 언어 X → 프로토타입 기반 객체지향 언어

> **Referenced**

- [ES6 브라우저 지원 현황 확인하기](https://kangax.github.io/compat-table/es6/)
- https://poiemaweb.com/js-introduction
