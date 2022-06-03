---
title: RegExp(정규 표현식)
date: '2022-06-03'
tags: ['javascript']
draft: false
summary: 일정한 패턴을 가진 문자열의 집합을 표현하기 위해 사용하는 형식 언어
layout: PostSimple
authors: ['default']
---

# RegExp

## 정규 표현식

- 일정한 패턴을 가진 문자열의 집합을 표현하기 위해 사용하는 형식 언어
  - 대부분의 프로그래밍 언어와 코드 에디터에 내장됨
- 문자열을 대상으로 패턴 매칭 기능 제공
  - 특정 패턴과 일치하는 문자열 검색 및 추출, 치환 기능

## 정규 표현식의 생성

<img alt="javascript_regexp1" src="/static/images/javascript_regexp1.png" width="300"/>

- 정규 표현식 객체를 생성하기 위해서는 `정규 표현식 리터럴`과 `RegExp 생성자 함수`를 사용할 수 있음

    ```
    /*
     * pattern: 정규 표현식의 패턴
     * flags: 정규 표현식의 플래그(g, i, m, u, y)
    */
    
    new RegExp(pattern[, flags])
    ```

    ```javascript
    
    const target = 'Is this all there is?';
    
    const regexp = new RegExp(/is/i); // ES6
    // const regexp = new RegExp(/is/, 'i');
    // const regexp = new RegExp('is', 'i');
    
    regexp.test(target); // -> true
    ```

- RegExp 생성자 함수를 사용하면 동적으로 RegExp 객체를 생성할 수 있음

  ```javascript
  const count = (str, char) => (str.match(new RegExp(char, 'gi')) ?? []).length;
    
  count('Is this all there is?', 'is'); // -> 3
  count('Is this all there is?', 'xx'); // -> 0
  ```

## RegExp 메서드

### `RegExp.prototype.exec`

- 정규 표현식의 패턴을 검색해서 매칭 결과를 `배열로 반환`
- 매칭 결과 없을 때에는 `null 반환`

```javascript
const target = 'Is this all there is?';
const regExp = /is/;

regExp.exec(target);
// -> ["is", index: 5, input: "Is this all there is?", groups: undefined]
```

- exec 메서드는 문자열 내의 모든 패턴을 검색하는 g플래그를 지정해도 `첫 번째 매칭 결과만 반환함`

### `RegExp.prototype.test`

- 정규 표현식의 패턴을 검색하여 매칭 결과를 `불리언 값으로 반환`

```javascript
const target = 'Is this all there is?';
const regExp = /is/;

regExp.test(taraget); // -> true
```

### `String.prototype.match`

- String 표준 빌트인 객체가 제공하는 `match 메서드`는 대상 문자열과 인수로 전달받은 `정규 표현식과의 매칭 결과를 배열로 반환`

```javascript
const target = 'Is this all there is?';
const regExp = /is/;

target.match(regExp);
// -> ["is", index: 5, input: "Is this all there is?", groups: undefined]
```

- exec 메서드와 다르게 `g플래그가 지정`되면 `모든 매칭 결과를 배열로 반환`

  ```javascript
  const target = 'Is this all there is?';
  const regExp = /is/g;
  
  target.match(regExp); // -> ["is", "is"]
  ```

## 플래그

- 총 6개의 플래그를 지원하며 정규 표현식의 검색 방식을 설정하기 위해 사용함

| 플래그 | 의미          | 설명                                |
|-----|-------------|-----------------------------------|
| i   | Ignore case | 대소문자를 구별하지 않고 패턴을 검색함             |
| g   | Global      | 대상 문자열 내에서 패턴과 일치하는 모든 문자열을 전역 검색 |
| m   | Multi line  | 문자열의 행이 바뀌더라도 패턴 검색을 계속함          |

- 플래그는 `옵션`이므로 `선택 가능`하며, 순서와 상관없이 `하나 이상의 플래그`를 `동시에 설정`할 수 있음
- 플래그를 사용하지 않았을 경우 `대소문자를 구별`해서 `패턴을 검색`함

```javascript
const target = 'Is this all there is?';

// target 문자열에서 is 문자열을 대소문자 구별하여 한 번만 검색
target.math(/is/);
// -> ["is", index: 5, input: "Is this all there is?", groups: undefined]

// target 문자열에서 is 문자열을 대소문자 구별하지 않고 한 번만 검색
target.math(/is/i);
// -> ["is", index: 0, input: "Is this all there is?", groups: undefined]

// target 문자열에서 is 문자열을 대소문자 구별하여 전역 검색
target.math(/is/g);
// -> ["is", "is"]

// target 문자열에서 is 문자열을 대소문자 구별하지 않고 전역 검색
target.math(/is/ig);
// -> ["is", "is", "is"]
```

## 패턴

- `일정한 규칙을 가진 문자열의 집합`을 표현하기 위해 사용하는 `형식 언어`
- 문자열의 `일정한 규칙을 표현`하기 위해서 사용함
- `플래그`
  - 정규 표현식의 검색 방법
- 패턴은 `/`로 열고 닫으며 문자열의 `따옴표는 생략`

### 문자열 검색

- `RegExp 메서드`를 사용해 검색 `대상 문자열과 정규 표현식의 매칭 결과`를 구하면 검색이 수행

  ```javascript
  const target = 'Is this all there is?';
  
  // 플래그 생략으로 대소문자 구별함
  const regExp = /is/;
  
  // target과 정규 표현식이 매치하는지 테스트
  regExp.test(target); // -> true
  
  // target과 정규 표현식의 매칭 결과 구하기
  target.match(regExp);
  // -> ["is", index: 5, input: "Is this all there is?", groups: undefined]
  ```

### 임의의 문자열 검색

- `.`은 임의의 문자 한 개를 의미함

  ```javascript
  const target = 'Is this all there is?';
  
  const regExp = /.../g;
  
  target.match(regExp); // -> ["Is ", "thi", "s a", "ll ", "the", "re ", "is?"]
  ```

### 반복 검색

- \{m,n\}은 앞선 패턴이 최소 m번, 최대 n번 반복되는 문자열을 의미함

  ```javascript
  const target = 'A AA B BB Aa Bb AAA';
  
  // 'A'가 최소 1번, 최대 2번 반복되는 문자열을 전역 검색
  const regExp = /A{1,2}/g;
  
  target.match(regExp); // -> ["A", "AA", "A", "AA", "A"]
  ```

- \{n\}은 앞선 패턴이 n번 반복되는 문자열을 의미함
  - 즉, \{n\}은 \{n, n\}과 같음
- \{n,\}은 앞선 패턴이 최소 n번 이상 반복되는 문자열 의미

  ```javascript
  const target = 'A AA B BB Aa Bb AAA';
  
  // 'A'가 2번 반복되는 문자열을 전역 검색
  const regExp = /A{2,}/g;
  
  target.match(regExp); // -> ["AA", "AAA"];
  ```

- +는 앞선 패턴 ‘A’가 한번 이상 반복되는 문자열을 의미

  ```javascript
  const target = 'A AA B BB Aa Bb AAA';
  
  // 'A' 가 최소 한 번 이상 반복되는 문자열('A', 'AA', 'AAA, ...)을 전역 검색함
  const regExp = /A+/g;
  
  target.match(regExp); // ["A", "AA", "A", "AAA"]
  ```

- ?는 앞선 패턴이 최대 한 번 이상 반복되는 문자열

  ```javascript
  const target = 'color colour';
  
  // 'colo' 다음 'u'가 최대 한 번 이상 반복되고 'r'이 이어지는
  // 문자열 'color', 'colour'를 전역 검색함
  const regExp = /colou?r/g;
  
  target.match(regExp); // -> ["color", "colour"]
  ```

### OR 검색

- |은 or의 의미를 가짐
- `[]` 내의 문자는 or로 동작함
- 그 뒤에 +를 사용하면 앞선 패턴을 한 번 이상 반복함
- `문자열 검색`

    ```javascript
    const target = 'A AA B BB Aa Bb';
    
    // 'A' 또는 'B'를 전역 검색함
    const regExp1 = /A|B/g;
    
    target.match(regExp1); // -> ["A", "A", "A", "B", "B", "B", "A", "B"]
    
    // 'A' 또는 'B'가 한 번 이상 반복되는 문자열을 전역 검색
    // 'A', 'AA', 'AAA', ... 또는 'B', 'BB', 'BBB', ...
    const regExp2 = /A+|B+/g;
    
    target.match(regExp2); // -> ["A", "AA", "B", "BB", "A", "B"]
    
    // 'A' ~ 'Z'가 한 번 이상 반복되는 문자열을 전역 검색
    // 'A', 'AA', 'AAA', ... 또는 'B', 'BB', 'BBB', ... ~ 또는 'Z', 'ZZ', 'ZZZ', ...
    const regExp3 = /[A-Z]+/g;
    
    target.match(regExp3); // -> ["A", "AA", "BB", "ZZ", "A", "B"]
    
    // 'A' ~ 'Z' 또는 'a' ~ 'z'가 한 번 이상 반복되는 문자열을 전역 검색
    const regExp4 = /[A-Za-z]+/g;
    
    target.match(regExp4); // -> ["AA", "BB", "Aa", "Bb"]
    ```

- `숫자 검색`

    ```javascript
    const target = 'AA BB 12,345';
    
    // '0' ~ '9'가 한 번 이상 반복되는 문자열을 전역 검색
    const regExp1 = /[0-9]+/g;
    
    target.match(regExp1); // -> ["12", "345"]
    
    // '0' ~ '9' 또는 ','가 한 번 이상 반복되는 문자열 전역 검색
    const regExp2 = /[0-9,]+/g;
    target.match(regExp2); // -> ["12,345"]
    ```

### NOT 검색

- […] 내의 ^는 not의 의미를 가짐

  ```javascript
  const target = 'AA BB 12 Aa Bb';
  
  // 숫자를 제외한 문자열을 전역 검색
  const regExp = /[^0-9]+/g;
  
  target.match(regExp); // -> ["AA BB ", " Aa Bb"]
  ```

### 시작 위치로 검색

- […] 밖의 ^은 문자열의 시작을 의미함

  ```javascript
  const target = 'https://naver.com';
  
  // 'https'로 시작하는지 검사
  const regExp = /^https/;
  
  regExp.test(target); // -> true
  ```

### 마지막 위치로 검색

- $는 문자열의 마지막을 의미함

  ```javascript
  const target = 'https://naver.com';
  
  // 'com'으로 끝나는지 검색
  const regExp = /com$/;
  
  regExp.test(target); // -> true
  ```

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 578 ~ 588p
