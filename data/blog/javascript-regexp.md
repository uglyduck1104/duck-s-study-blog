---
title: String
date: '2022-06-03'
tags: ['javascript']
draft: false
summary: String 래퍼 객체는 읽기 전용 객체로 제공되며 String 객체의 메서드는 언제나 새로운 문자열을 반환함
layout: PostSimple
authors: ['default']
---

# String

## String 생성자 함수

- String 객체는 생성자 함수이므로 new 연산자와 함께 호출하여 String 인스턴스를 생성할 수 있음
- `new 연산자와 함께 호출`하면 `[[StringData]]` 내부 슬롯에 빈 문자열을 할당한 `String 래퍼 객체를 생성`

    ```javascript
    const strObj = new String('Lee');
    console.log(strObj); // String {0: "L", 1: "e", 2: "e", legnth: 3, [[PrimitiveValue]]: "Lee"}
    ```

  - 생성자 함수의 인수로 문자열을 전달하면서 new 연산자를 함께 호출하면 `[[StringData]] 내부 슬롯`에 인수로 전달받은 문자열을 할당한 `String 래퍼 객체를 생성`
- `문자열`은 원시 값으로 `변경할 수 없음`
- 문자열이 아닌 값을 전달하면 문자열로 강제 변환 후 [[StringData]] 내부 슬롯에 변환된 문자열을 할당

    ```javascript
    let strObj = new String(123);
    console.log(strObj);
    // String {0: "1", 1: "2", 2: "3", length: 3, [[PrimitiveValue]]: "123"}
    ```

## String 메서드

- `원본 배열을 직접 변경`하는 메서드와 `새로운 배열을 생성하여 반환`하는 메서드가 있음
- String 객체에는 원본 String 래퍼 객체를 직접 변경하는 메서드는 존재하지 않음(⭐⭐⭐)
  - String 래퍼 객체는 `읽기 전용 객체로 제공`되며 String 객체의 메서드는 `언제나 새로운 문자열을 반환함`

  ```javascript
  const strObj = new String('Lee');
  
  console.log(Object.getOwnPropertyDescriptors(strObj));
  /* String 래퍼 객체는 읽기 전용 객체이므로 writable 프로퍼티 어트리뷰트 값이 false
  {
    '0': { value: 'L', writable: false, enumerable: true, configurable: false },
    '1': { value: 'e', writable: false, enumerable: true, configurable: false },
    '2': { value: 'e', writable: false, enumerable: true, configurable: false },
    length: { value: 3, writable: false, enumerable: false, configurable: false }
  }
  */
  ```

### `String.prototype.indexOf`

- 대상 문자열에서 인수로 전달받은 문자열을 검색하여 첫 번째 인덱스 반환
- 실패 시 -1 반환

```javascript
const str = 'Hello World';

// 문자열 str에서 'l'을 검색하여 첫 번째 인덱스 반환
str.indexOf('l'); // -> 2

// 문자열 str의 인덱스 3부터 'l'을 검색하여 첫 번째 인덱스를 반환
str.indexOf('l', 3); // -> 3
```

- 문자열 존재 유무 확인

    ```javascript
    if(str.indexOf('Hello') !== -1){
    	// 문자열 str에 'Hello'가 포함되어 있는 경우에 처리할 내용
    }
    
    // ES6 include 메서드 사용하기
    /*
    if(str.includes('Hello')){
    	// 문자열 str에 'Hello'가 포함되어 있는 경우에 처리할 내용
    }
    */
    ```

### `String.prototype.search`

- `정규 표현식과 매치`하는 문자열을 검색하여 일치하는 문자열의 인덱스 반환
- 검색 실패시 -1 반환

  ```javascript
  const str = 'Hello world';
  
  // 문자열 str에서 정규 표현식과 매치하는 문자열을 검색하여 일치하는 문자열의 인덱스 반환
  str.search(/o/); // -> 4
  str.search(/x/); // -> -1
  ```

### `String.prototype.includes`

- `ES6`에서 도입된 includes 메서드는 대상 문자열에 인수로 전달받은 `문자열이 포함`되어 있는지 확인하여 그 결과를 `true` 또는 `false`로 반환

  ```javascript
  const str = 'Hello world';
  
  str.includes('Hello'); // -> true
  str.includes(''); // -> true
  str.includes('x'); // -> false
  str.includes(); // -> false
  
  // 2번째 인수로 검색
  str.includes('l', 3); // -> true
  str.includes('H', 3); // -> false
  ```

### `String.prototype.startsWith`

- `ES6`에서 도입된 startWith 메서드는 대상 문자열이 인수로 전달받은 `문자열로 시작`하는지 확인하여 그 결과를 `true` 또는 `false`로 반환

  ```javascript
  const str = 'Hello world';
  
  // 문자열이 str이 'He'로 시작하는지 확인
  str.startsWith('He'); // -> true
  ```

### `String.prototype.endsWith`

- `ES6`에서 도입된 endsWith 메서드는 대상 문자열이 인수로 전달받은 `문자열로 끝나는지` 확인하여 그 결과를 `true` 또는 `false`로 반환

  ```javascript
  const str = 'Hello world';
  
  // 문자열이 str이 'ld'로 시작하는지 확인
  str.startsWith('ld'); // -> true
  ```

### `String.prototype.charAt`

- 대상 문자열에서 인수로 전달받은 인덱스에 위치한 문자를 검색하여 반환

  ```javascript
  const str = 'Hello';
  
  for(let i = 0; i < str.length; i++){
    console.log(str.charAt(i)); // H e l l o
  }
  ```

- 문자열을 벗어나는 경우 빈 문자열 반환

### `String.prototype.substring`

- `첫 번째 인수`로 전달받은 `인덱스에 위치하는 문자`부터 `두 번째 인수`로 전달받은 `인덱스에 위치하는 문자의 바로 이전 문자까지`의 부분 문자열 반환
- 두 번째 인수 생략 시 `마지막 문자까지` 부분 문자열을 반환함

  ```javascript
  const str = 'Hello World';
  
  // 인덱스 1부터 인덱스 4 이전까지의 부분 문자열을 반환
  str.substring(1, 4); // -> ell
  ```

![javascript_string](/static/images/javascript_string.png)

- 첫 번째 인수와 두 번째 인수의 `상관 관계`에 따라 동작 여부가 달라짐
  - 첫 번째 인수 > 두 번째 인수인 경우 `두 인수 교환`
  - 인수 < 0 또는 NaN인 경우 `0으로 취급`
  - 인수 > 문자열의 길이(str.length)인 경우 `인수는 문자열의 길이(str.length)로 취급`됨

    ```javascript
    const str = 'Hello World'; // str.length == 11
    
    // 첫 번째 인수 > 두 번째 인수인 경우 교환
    str.substring(4, 1); // -> 'ell'
    
    // 인수 < 0 또는 NaN인 경우 0으로 취급
    str.substring(-2); // -> 'Hello World'
    
    // 인수 > 문자열의 길이(str.length)인 경우 문자열의 길이로 취급
    str.substring(1, 100); // -> 'ello World'
    str.substring(20); // -> ''
    ```

### `String.prototype.slice`

- substring 메서드와 동일하게 동작하나, slice 메서드에는 `음수인 인수를 전달`할 수 있음

  ```javascript
  const str = 'hello world';
  
  // substring과 slice 메서드는 동일하게 동작함
  // 0번째부터 5번째 이전 문자까지 잘라내어 반환
  str.substring(0, 5); // -> 'hello'
  str.slice(0, 5); // -> 'hello'
  
  // 인덱스가 2인 문자부터 마지막 문자까지 잘라내어 반환
  str.substring(2); // - 'llo world'
  str.slice(2); // -> 'llo world'
  
  // 인수 < 0 또는 NaN인 경우 0으로 취급
  str.substring(-5); // -> 'hello world'
  // slice 메서드는 음수인 인수를 전달할 수 있음. 뒤에서 5자리를 잘라내어 반환
  str.slice(-5); // -> 'world'
  ```

### `String.prototype.toUpperCase`

- 대상 문자열을 모두 대문자로 변경

  ```javascript
  const str = 'Hello World!';
  
  str.toUpperCase(); // -> 'HELLO WORLD!'
  ```

### `String.prototype.toLowerCase`

- 대상 문자열을 모두 소문자로 변경한 문자열을 반환

  ```javascript
  const str = 'Hello World!';
  
  str.toLowerCase(); // -> 'hello world!'
  ```

### `String.prototype.trim`

- 대상 문자열 앞뒤에 공백 문자가 있을 경우 이를 제거한 문자열 반환

  ```javascript
  const str = '   foo   ';
  
  str.trim(); // -> 'foo';
  ```

### `String.prototype.repeat`

- 대상 문자열을 인수로 전달받은 정수만큼 반복해 연결한 `새로운 문자열을 반환`
  - `0`이면 `빈 문자열` 반환
  - `음수`면 `RangeError` 발생

  ```javascript
  const str = 'abc';
  
  str.repeat(); // -> ''
  str.repeat(0); // -> ''
  str.repeat(1); // -> 'abc'
  str.repeat(-1); // -> RangeError: Invalid count value
  ```

### `String.prototype.replace`

- 첫 번째 인수로 전달받은 문자열 또는 정규표현식을 검색한 두 번째 인수로 전달된 문자열로 치환한 문자열 반환

  ```javascript
  const str = 'Hello world';
  
  // str에서 첫 번째 인수 'world'를 검색하여 두 번째 인수 'Lee'로 치환
  str.replace('world', 'Lee'); // -> 'Hello Lee'
  ```

- 검색된 문자열이 여럿 존재할 경우 첫 번째로 검색된 문자열만 치환

    ```javascript
    const str = 'Hello world world';
    
    str.replace('world', 'Lee'); // -> 'Hello Lee world'
    ```

- 첫 번째 인수로 정규 표현식을 전달할 수 있음

    ```javascript
    const str = 'Hello Hello';
    
    // 'hello'를 대소문자를 구별하지 않고 전역 검색
    str.replace(/hello/gi, 'Lee'); // -> 'Lee Lee'
    ```

- 카멜 케이스 ↔ 스네이크 케이스 치환

    ```javascript
    function camelToSnake(camelCalse){
    	// /.[A-Z]/g는 임의의 한 문자와 대문자로 이루어진 문자열에 매치
    	// 치환 함수의 인수로 매치 결과가 전달되고, 치환 함수가 반환된 결과와 매치 결과를 치환
    	return camelCase.replace(/.[A-Z]/g, match => {
    		console.log(match); // 'oW'
    		return match[0] + '_' + match[1].toLowerCase();
    	});
    }
    
    const camelCase = 'helloWorld';
    camelToSnake(camelCase); // -> 'hello_world'
    
    function snakeCaseToCamel(snakeCase){
    	return snakeCase.replace(/_[a-z]]/g, match => {
    		console.log(match); // '_w'
    		return match[1].toUpperCase();
    	});
    }
    
    const snakeCase = 'hello_world';
    snakeToCamel(snakeCase); // -> 'helloWorld'
    ```

### `String.prototype.split`

- 첫 번째 인수로 전달한 문자열 또는 정규 표현식을 검색하여 `문자열을 구분한 후 분리된 문자열로 이루어진 배열을 반환`

  ```javascript
  const str = 'How are you doing?';
  
  // 공백으로 구분(단어로 구분)하여 배열로 반환
  str.split(' '); // -> ["How", "are", "you", "doing?"]
  
  // \s는 여러 가지 공백 문자를 의미함
  // [\t\r\n\v\f]와 같은 의미
  str.split(/\s/); // -> ["How", "are", "you", "doing?"]
  
  // 인수로 빈 문자열을 전달하면 각 문자를 모두 분리
  str.split('');
  // -> ['H', 'o', 'w', ' ', 'a', 'r', 'e', ' ', 'y', 'o', 'u', ' ', 'd', 'o', 'i', 'n', 'g', '?']
  ```

- `역순으로 뒤집기`

    ```javascript
    function reverseString(str) {
    	return str.split('').reverse().join('');
    }
    
    reverseString('Hello world!'); // -> '!dlrow olleH'
    ```

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 592 ~ 604p
