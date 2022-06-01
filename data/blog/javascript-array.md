---
title: 배열
date: '2022-06-01'
tags: ['javascript']
draft: false
summary: 여러 개의 값을 순차적으로 나열한 사용 빈도가 매우 높은 가장 기본적인 자료 구조
layout: PostSimple
authors: ['default']
---

# 배열

## 배열이란?

- 여러 개의 값을 순차적으로 나열한 사용 빈도가 매우 높은 `가장 기본적인 자료 구조`
- 자바스크립트의 모든 값은 배열의 `요소`가 될 수 있음
  - 원시값, 객체, 함수, 배열 등의 값으로 인정하는 모든 것
- `배열 리터럴`, `Array 생성자 함수`, `Array.of`, `Array.from` 메서드로 생성 가능
- 배열의 생성자 함수는 `Array`이며, 프로토타입 객체는 `Array.prototype`임

    ```javascript
    const arr = [1, 2, 3];
    
    arr.constructor === Array // -> true
    Object.getPrototypeOf(arr) === Array.prototype // -> true
    ```

### 일반 객체와 배열의 비교

| 구분          | 객체             | 배열      |
|-------------|----------------|---------|
| 구조          | 프로퍼티 키와 프로퍼티 값 | 인덱스와 요소 |
| 값의 참조       | 프로퍼티 키         | 인덱스     |
| 값의 순서       | X              | O       |
| length 프로퍼티 | X              | O       |

- `값의 순서`와 `length 프로퍼티`가 가장 명확한 차이

## 자바스크립트 배열은 배열이 아니다

### 일반적인 배열

<img alt="javascript_array1" src="/static/images/javascript_array1.png" width="500"/>

- 배열의 요소는 하나의 데이터 타입으로 통일되어 있으며 서로 연속적으로 인접해 있는 이러한 배열을 `밀집 배열`이라 함
  - 각 요소가 `동일한 데이터 크기`를 가지고, 빈틈없이 연속적으로 이어져 있으므로 `단 한번의 연산으로 임의의 요소에 접근`할 수 있음(임의 접근, `시간 복잡도 O(1)`)

- 정렬되지 않은 배열에서 특정 요소를 검색하는 경우 배열의 `모든 요소를 처음부터 발견할때까지 차례대로 검색`해야 함(선형 검색, `시간 복잡도 O(n)`)

    ```javascript
    // 선형 검색을 통해 배열에 특정 요소가 있는지 확인
    // 배열에 특정 요소가 존재하면 특정 요소의 인덱스를 반환하고, 존재하지 않으면 -1 반환
    function linearSearch(array, target){
    	const length = array.length;
    	
    	for(let i = 0; i < length; i++){
    		if(array[i] === target) return i;
    	}
    	
    	return -1;
    }
    
    console.log(linearSearch([1, 2, 3, 4, 5, 6], 3)); // 2
    console.log(linearSearch([1, 2, 3, 4, 5, 6], 0)); // -1
    ```

<img alt="javascript_array2" src="/static/images/javascript_array2.png" width="500"/>

- 배열에 요소에 `삽입 및 삭제`가 이뤄지는 경우에 `배열의 요소는 연속적으로 유지`하기 위해서 `요소를 이동`시켜야 함

### 자바스크립트의 배열

- 자바스크립트에서는 앞서 살펴본 배열들의 의미와 다름
- 배열의 요소를 위한 각각의 메모리 공간은 동일한 크기를 가지지 않아도 되며, 연속적으로 이어져 있지 않을 수도 있음
  - 배열의 요소가 `연속적으로 이어져 있지 않은 배열`을 `희소 배열`이라함
- 자바스크립의 배열은 일반적인 배열의 동작을 흉내 낸 `특수한 객체`

### 일반 배열 vs 자바스크립트 배열

- 일반적인 배열
  - 인덱스로 요소에 빠르게 접근이 가능함
  - `요소 삽입, 삭제 시 비효율적임`
- 자바스크립트 배열
  - `해시 테이블로 구현된 객체`이므로 인덱스 요소에 접근하는 경우 일반적인 배열보다 `성능적인 면에서 느릴 수 있음`
  - `요소 삽입, 삭제 시` 일반적인 배열보다 `빠른 성능을 기대`할 수 있음

```javascript
const arr = [];

console.time('Array Performance Test');

for (let i = 0; i < 10000000; i++) {
  arr[i] = i;
}
console.timeEnd('Array Performance Test');
// 약 340ms

const obj = {};

console.time('Object Performance Test');

for (let i = 0; i < 10000000; i++) {
  obj[i] = i;
}

console.timeEnd('Object Performance Test');
// 약 600ms
```

## length 프로퍼티와 희소 배열

- 요소의 개수, `배열의 길이`를 나타내는 `0 이상의 정수`를 값으로 가짐
- 배열에 요소를 `추가`하거나 `삭제`하면 자동 갱신
- 임의의 숫자 값을 명시적으로 할당 가능
- 현재 length 프로퍼티 값보다 작은 숫자 값을 할당하면 배열의 길이가 줄어듦

    ```javascript
    const arr = [1, 2, 3, 4, 5];
    
    arr.length = 3;
    console.log(arr); // [1, 2, 3]
    ```

- 현재 length 프로퍼티 값보다 `큰 숫자 값`을 할당하면 프로퍼티 값은 변경되지만 배열의 길이는 늘어나지 않음

    ```javascript
    const arr = [1];
    
    arr.length = 3;
    console.log(arr.length); // 3
    console.log(arr); // [1, empty x 2]
    ```

  - 값 없이 비어 있는 요소를 위해 메모리 공간을 확보하지도 않으며, 빈 요소를 생성하지도 않음

- 자바스크립트는 희소 배열을 문법적으로 허용함
  - 중간이나 앞부분이 비어 있을 수도 있음

    ```javascript
    // 희소 배열
    const sparse = [, 2, , 4];
    
    // 희소 배열의 length 프로퍼티 값은 요소의 개수와 일치하지 않음
    console.log(sparse.length); // 4
    console.log(sparse); // [empty, 2, empty, 4]
    
    // 배열 sparse에는 인덱스가 0, 2인 요소가 존재하지 않음
    console.log(Object.getOwnPropertyDescriptions(sparse));
    /*
    {
    	'1': { value: 2, writable: true, enumerable: true, configurable: true },
    	'3': { value: 4, writable: true, enumerable: true, configurable: true },
    	length: { value: 4, writable: true, enumerable: false, configurable: false },
    }
    */
    
    ```

- 희소 배열은 length와 배열 요소의 개수가 일치하지 않고, length는 희소 배열의 실제 요소 개수보다 언제나 큼

> 배열을 생성할 경우에는 희소 배열을 생성하지 않도록 주의하고, 배열에는 같은 타입의 요소를 연속적으로 위치 시켜야 함

## 배열 생성

### 배열 리터럴

- 가장 일반적이고 간편한 배열 생성 방식

```javascript
const arr = [1, 2, 3];
consoe.log(arr.length); // 3
```

### Array 생성자 함수

- Array 생성자 함수는 전달된 인수의 개수에 따라 다르게 동작함

```javascript
// 전달된 인수가 2개 이상이면 인수를 요소로 갖는 배열을 생성
new Array(1, 2, 3); // -> [1, 2, 3]

// 전달된 인수가 1개지만 숫자가 아니면 인수를 요소로 갖는 배열을 생성
new Array({}); // - [{}]
```

> new 연산자와 함께 호출하지 않더라도 배열을 생성하는 생성자 함수로서 동작함
> → Array 생성자 함수 내부에서 new.target을 확인함
>

```javascript
Array(1, 2, 3); // -> [1, 2, 3]
```

### Array.of

- 전달된 인수를 요소로 갖는 배열을 생성함
- 생성자 함수와 다르게 전달된 인수가 1개이고 숫자이더라도 `인수`를 `요소로 갖는 배열 생성`

```javascript
Array.of(1); // -> [1]

Array.of(1, 2, 3); // -> [1, 2, 3]

Array.of('string'); // -> ['string']
```

### Array.from

- 유사 배열 객체 또는 이터러블 객체를 `인수로 전달`받아 `배열로 변환`하여 반환함

```javascript
// 유사 배열 객체를 변환하여 배열을 생성
Array.from({ length: 2, 0: 'a', 1: 'b'}); // -> ['a', 'b']

// 이터러블을 변환하여 배열을 생성, 문자열은 이터러블
Array.from('Hello'); // -> ['H', 'e', 'l', 'l', 'o']

// Array.from은 두 번째 인수로 전달한 콜백 함수의 반환값으로 구성원 배열을 반환했음
Array.from({ length: 3}, (_, i) => i); // -> [0, 1, 2]
```

## 배열 요소의 참조

- 대괄호 표기법을 사용해배열의 요소를 참조
- 참조의 의미에서 봤을 때, 객체의 프로퍼티 키와 같은 역할을 함
- 인덱스를 나타내는 문자열을 프로퍼티 키로 갖는 `객체`이므로 `존재하지 않은 요소에 접근`했을 때 `undefined를 반환`함
- 희소 배열 역시, 존재하지 않는 요소를 참조해도 undefined가 반환됨

```javascript
// 희소 배열
const arr = [1, , 3];

// 배열 arr에는 인덱스가 1인 요소가 존재하지 않음
console.log(Object.getOwnPropertyDescriptors(arr));
/*
{
	'0': {value: 1, writable: true, enumerable: true, configurable: true},
	'2': {value: 3, writable: true, enumerable: true, configurable: true},
	length: {value: 3, writable: true, enumerable: false, configurable: false},
}
*/
```

## 배열 요소의 추가와 갱신

- 객체에 프로퍼티를 동적으로 추가할 수 있는 것처럼 `배열에도 요소를 동적으로 추가`할 수 있음
  - 존재하지 않는 인덱스를 사용해 값을 할당하면 `새로운 요소가 추가`됨

    ```javascript
    const arr = [0];
    
    // 배열 요소의 추가
    arr[1] = 1;
    
    console.log(arr); // [0, 1]
    console.log(arr.length) // 2
    ```


- `인덱스`는 요소의 위치를 나타내므로 `반드시 0 이상의 정수를 사용`해야 함
  - `정수 이외의 값`을 인덱스처럼 사용하면 `프로퍼티가 생성`됨

    ```javascript
    const arr = [];
    
    // 배열 요소의 추가
    arr[0] = 1;
    arr['1'] = 2;
    
    // 프로퍼티 추가
    arr['foo'] = 3;
    arr.bar = 4;
    arr[1.1] = 5;
    arr[-1] = 6;
    
    console.log(arr); // [1, 2, foo: 3, bar: 4, '1.1': 5, '-1': 6]
    
    // 프로퍼티는 length에 영향을 주지 않음
    console.log(arr.length); // 2
    ```

## 배열 요소의 삭제

- 배열은 객체이므로, 특정 요소를 삭제하기 위해 `delete 연산자`를 사용할 수 있으나, 희소 배열을 만드는 경우가 생겨서 사용을 하지 않는 것이 좋음
- `Array.prototype.splice` 메서드를 사용할 것을 권장

    ```javascript
    const arr = [1, 2, 3];
    
    // Array.prototype.splice(삭제를 시작할 인덱스, 삭제할 요소 수)
    // arr[1]부터 1개의 요소를 제거
    arr.splice(1, 1);
    console.log(arr); // [1, 3]
    
    // length 프로퍼티가 자동 갱신됨
    console.log(arr.length); // 2
    ```

## 배열 메서드

- 배열을 다룰 때 유용한 빌트인 메서드를 제공함
- 배열에는 원본 배열을 `직접 변경하는 메서드`와 원본 배열을 직접 변경하지 않고 `새로운 배열을 생성하여 반환하는 메서드`로 구분

```javascript
const arr = [1];

// push 메서드는 원본 배열(arr)을 직접 변경
arr.push(2);
console.log(arr); // [1, 2]

// concat 메서드는 원본 배열(arr)을 직접 변경하지 않고 새로운 배열을 생성하여 반환함
const result = arr.concat(3);
console.log(arr);    // [1, 2]
console.log(result); // [1, 2, 3]
```

- 원본 배열을 직접 변경하는 메서드는 `외부 상태를 직접 변경하는 부수 효과`가 있으므로 사용할 때 주의해야 함
  - 가급적이면 원본 배열을 `직접 변경하지 않는 메서드를 사용`하자!(⭐⭐⭐)

### Array.isArray

- Array 생성자 함수의 정적 메서드
- 전달된 인수가 `배열이면 true`, `배열이 아니면 false`를 반환

```javascript
// true
Array.isArray([]);
Array.isArray([1, 2]);
Array.isArray(new Array());

// false
Array.isArray();
Array.isArray({});
Array.isArray(null);
Array.isArray(undefined);
Array.isArray(1);
Array.isArray('Array');
Array.isArray(true);
Array.isArray(false);
Array.isArray({ 0: 1, length: 1});
```

### `Array.prototype.indexOf`

- 원본 배열에서 `인수로 전달된 요소를 검색`하여 인덱스 반환
  - `중복되는 요소`가 여러 개 있으면 `첫 번째로 검색된 요소`의 인덱스 반환
  - `존재하지 않으면 -1`을 반환

    ```javascript
    const arr = [1, 2, 2, 3];
    
    // 배열 arr에서 요소 2를 검색하여 첫 번째로 검색된 요소의 인덱스를 반환
    arr.indexOf(2); // -> 1
    
    // 배열 arr에서 요소 4가 없으므로 -1 반환
    arr.indexOf(4); // -> -1
    
    // 두 번째 인수는 검색을 시작할 인덱스를 지정할 수 있음
    arr.indexOf(2, 2); // -> 2
    ```


- 특정 요소 존재하는지 확인하기

    ```javascript
    // indexOf 메서드 사용
    const food = ['apple', 'banana', 'orange'];
    
    // foods 배열에 'orange' 요소가 존재하는지 확인
    if (foods.indexOf('orange') === -1) {
    	// foods 배열에 'orange' 요소가 존재하지 않으면 'orange' 요소를 추가함
    	foods.push('orange');
    }
    
    // includes 메서드 사용 - 가독성이 더 좋음
    /*
    if (!foods.includes('orange'){
    	foods.push('orange');
    }
    */
    
    console.log(foods); // ["apple", "banana", "orange"]
    ```

### `Array.prototype.push`

- 인수로 전달받은 모든 값을 `원본 배열의 마지막 요소로 추가`하고 변경된 length 프로퍼티 값을 반환
  - 원본 배열을 직접 변경함
  - 성능 면에서 좋지 않고, 마지막 요소로 추가할 요소가 하나뿐이라면 push 메서드를 사용하지 않고 length 프로퍼티를 사용하여 마지막 요소를 직접 추가하는 것이 더 빠름

      ```javascript
      const arr = [1, 2];
      
      // arr.push(3)과 동일한 처리를 함
      arr[arr.length] = 3;
      console.log(arr); // [1, 2, 3]
      ```

- 부수효과를 피하기위해 ES6의 `스프레드 문법을 이용해서 처리`할 수 있음

    ```javascript
    const arr = [1, 2];
    
    // ES6 스프레드 문법
    const newArr = [...arr, 3];
    console.log(newArr); // [1, 2, 3]
    ```

### `Array.prototype.pop`

- 원본 배열에서 마지막 요소를 제거하고 제거한 요소를 반환
- 빈 배열이면 undefined
- push와 동일하게 `원본 배열을 직접 변경`함

    ```javascript
    const arr = [1, 2];
    
    let result = arr.pop();
    console.log(result); // 2
    console.log(arr); // [1]
    ```

### `Array.prototype.unshift`

- 인수로 전달받은 모든 값을 `원본 배열의 선두에 요소로 추가`하고 변경된 length 프로퍼티 값을 반환
  - 원본 배열 직접 변경

    ```javascript
    const arr = [1, 2];
    
    let result = arr.unshift(3, 4);
    console.log(result); // 4
    console.log(arr); // [3, 4, 1, 2]
    ```

- push 메서드와 마찬가지로 부수효과가 우려되어 `ES6 스프레드 문법을 사용`하는 것이 좋음

    ```javascript
    const arr = [1, 2];
    
    // ES6 스프레드 문법
    const newArr = [3, ...arr];
    console.log(arr); // [3, 1, 2]
    ```

### `Array.prototype.shift`

- 원본 배열에서 첫 번째 요소를 제거하고 제거한 요소 반환
  - 빈 배열은 undefined로 반환
  - 원본 배열 직접 변경

```javascript
const arr = [1, 2];

let result = arr.shift();
console.log(result); // 1

// shift 메서드는 원본 배열을 직접 변경
console.log(arr); // [2]
```

### `Array.prototype.concat`

- 인수로 전달된 값들을 원본 배열의 마지막 요소로 추가한 새로운 배열 반환
  - 전달한 값이 배열인 경우 해체하여 새로운 배열의 요소로 추가함
  - 원본 배열은 변경되지 않음

    ```javascript
    const arr1 = [1, 2];
    const arr2 = [3, 4];
    
    let result = arr1.concat(arr2);
    console.log(result); // [1, 2, 3, 4]
    
    result = arr1.concat(3);
    console.log(result); // [1, 2, 3]
    
    result = arr1.concat(arr2, 5);
    console.log(result); // [1, 2, 3, 4, 5]
    ```

> push와 unshift 메서드는 원본 배열을 직접 변경하지만 `concat 메서드`는 원본 배열을 변경하지 않고 `새로운 배열을 반환`

```javascript
let result = [1, 2].concat([3, 4]);
console.log(result); // [1, 2, 3, 4]

// concat 메서드는 ES6 스프레드 문법으로 대체 가능
result = [...[1, 2], ...[3, 4]];
console.log(result); // [1, 2, 3, 4]
```

### `Array.prototype.splice`

<img alt="javascript_array3" src="/static/images/javascript_array3.png" width="400"/>

- push, pop, unshift, shift 메서드는 모두 원본 배열을 직접 변경하고 처음이나 마지막에 요소를 추가하거나 제거함
- 원본 배열의 `중간에 요소를 추가하거나 제거`하는 경우 splice 메서드를 사용하고 `원본 배열을 직접 변경함`
- 3개의 매개변수
  - `start`
    - 원본 배열의 요소를 `제거하기 시작할 인덱스`
    - start 지점부터 모든 요소를 제거함
    - `음수인 경우 배열의 끝`에서의 인덱스를 나타냄
  - `deleteCount`(Option)
    - 원본의 배열의 요소를 제거하기 시작할 인덱스인 `start부터 제거할 요소의 개수`
  - `items`(Option)
    - 제거한 위치에 삽입할 요소들의 목록

1. 3개의 인수를 빠짐없이 전달하는 경우

    ```javascript
    const arr = [1, 2, 3, 4];
    
    // 원본 배열의 인덱스 1부터 2개의 요소를 제거하고 그 자리에 새로운 요소 20, 30 삽입
    const result = arr.splice(1, 2, 20, 30);
    
    // 제거한 요소는 배열로 반환됨
    console.log(result); // [2, 3]
    // splice 메서드는 원본 배열을 직접 변경 시킴
    console.log(arr); // [1, 20, 30, 4]
    ```

   ![javascript_array4](/static/images/javascript_array4.png)

- 시작 인덱스부터 제거할 요소의 개수만큼 원본 배열에서 요소를 제거함
- 세번째로 전달받은 인수로 제거한 위치에 삽입할 요소들을 원본 배열에 삽입

2. 두 번째 인수, `제거할 요소의 개수를 0`으로 지정하는 경우

    ```javascript
    const arr = [1, 2, 3, 4];
    
    // 원본 배열의 인덱스 1부터 0개의 요소를 제거하고 그 자리에 새로운 요소 100 삽입
    const result = arr.splice(1, 0, 100);
    
    // 원본 배열 변경
    console.log(arr); // [1, 100, 2, 3, 4]
    // 제거한 요소가 배열로 반환됨
    console.log(result); // []
    ```

- 원본 배열에 `새로운 요소들을 삽입`함

3. 세 번째 인수, 제거한 위치에 `추가할 요소들의 목록을 전달하지 않은 경우`

    ```javascript
    const arr = [1, 2, 3, 4];
    
    // 원본 배열의 인덱스 1부터 2개의 요소를 제거함
    const result = arr.splice(1, 2);
    
    // 원본 배열이 변경됨
    console.log(arr); // [1, 4]
    // 제거한 요소가 배열로 반환됨
    console.log(result); // [2, 3]
    ```

   ![javascript_array5](/static/images/javascript_array5.png)

- 원본 배열에서 `지정된 요소를 제거`하기만 함

4. 두 번째 인수, `제거할 요소의 개수를 생략`하는 경우

    ```javascript
    const arr = [1, 2, 3, 4];
    
    // 원본 배열의 인덱스 1부터 모든 요소를 제거함
    const result = arr.splice(1);
    
    // 원본 배열이 변경됨
    console.log(arr); // [1]
    // 제거한 요소가 배열로 반환됨
    console.log(result); // [2, 3, 4]
    ```

- 첫 번째 인수로 전달된 `시작 인덱스부터 모든 요소 제거`

### `Array.prototype.slice`

- 인수로 전달된 범위의 요소들을 복사하여 배열로 반환
- 원본 배열은 변하지 않음
- 2개의 매개변수
  - `start`
    - 복사를 시작할 인덱스
    - 음수인 경우 배열 끝에서의 인덱스를 나타냄
  - `end`
    - 복사를 종료할 인덱스
    - end는 생략 가능하며, 생략 시 기본값은 length 프로퍼티 값

```javascript
const arr = [1, 2, 3];

arr.slice(0, 1); // -> [1]
arr.slice(1, 2); // -> [2]

// 원본은 변하지 않음
console.log(arr); // [1, 2, 3]
```

- 첫 번째 인수(start)로 전달받은 인덱스부터 두 번째 인수(end)로 전달 받은 인덱스까지 요소들을 복사하여 배열로 반환

<img alt="javascript_array6" src="/static/images/javascript_array6.png" width="400"/>

1. 두 번째 인수(`end`)를 `생략`하는 경우

    ```javascript
    const arr = [1, 2, 3];
    
    // arr[1]부터 이후의 모든 요소를 복사하여 반환
    arr.slice(1); // -> [2, 3]
    ```

- 첫 번째 인수(start)로 전달받은 인덱스부터 모든 요소를 복사하여 배열로 반환함

2. 첫 번째 인수가 `음수`인 경우

    ```javascript
    const arr = [1, 2, 3];
    
    // 배열의 끝에서부터 요소를 한 개 복사하여 반환
    arr.slice(-1); // -> [3]
    
    // 배열의 끝에서부터 요소를 두 개 복사하여 반환
    arr.slice(-2); // -> [2, 3]
    ```

- `배열의 끝에서부터 요소를 복사`하여 `배열로 반환`

3. 인수를 `모두 생략`

    ```javascript
    const arr = [1, 2, 3];
    
    // 인수를 모두 생략하면 원본 배열의 복사본을 생성하여 반환
    const copy = arr.slice();
    console.log(copy); // [1, 2, 3]
    console.log(copy === arr); // false
    ```

- `원본 배열의 복사본을 생성하여 반환`
- 얕은 복사를 통해 생성됨

### 얕은 복사와 깊은 복사

- 얕은 복사
  - 객체를 프로퍼티 값으로 갖는 객체의 경우 얕은 복사는 `한 단계까지만 복사하는 것을 말함`
  - slide 메서드, ES6 스프레드 문법, Object.assign 메서드

    ```javascript
    function sum() {
    	// 이터러블을 배열로 변환(ES6 스프레드 문법)
    	const arr = [...arguments];
    	console.log(arr); // [1, 2, 3]
    
    	return arr.reduce((pre, cur) => pre + cur, 0);
    }
    
    console.log(sum(1, 2, 3)); // 6
    ```

- 깊은 복사
  - 객체에 `중첩되어 있는 객체까지 모두 복사`하는 것을 말함
  - Lodash 라이브러리 cloneDeep 권장

### `Array.prototype.join`

- join 메서드는 원본 배열의 모든 요소를 문자열로 변환 후, `구분자(생략 가능)로 연결한 문자열을 반환`

```javascript
const arr = [1, 2, 3, 4];

// 기본 구분자는 콤마
// 원본 배열 arr의 모든 요소를 문자열로 변환한 후 기본 구분자로 연결한 문자열을 반환
arr.join(); // -> '1,2,3,4';

// 원본 배열 arr의 모든 요소를 문자열로 변환한 후, 빈 문자열로 연결한 문자열을 반환
arr.join(''); // -> '1234'

// 원본 배열 arr의 모든 요소를 문자열로 변환한 후 구분자 ':'로 연결한 문자열 반환
arr.join(':')l // -> '1:2:3:4'
```

### `Array.prototype.reverse`

- reverse 메서드는 원본 `배열의 순서를 반대로 뒤집음`
- `원본 배열은 변경`됨

```javascript
const arr = [1, 2, 3];
const result = arr.reverse();

// reverse 메서드는 원본 배열을 직접 변경함
console.log(arr); // [3, 2, 1]
// 반환값은 변경된 배열임
console.log(result); // [3, 2, 1]
```

### `Array.prototype.fill`

- ES6에 도입된 fill 메서드는 인수로 전달받은 값을 `배열의 처음부터 끝까지 요소로 채움`
- `원본 배열 변경`됨

```javascript
const arr = [1, 2, 3];

// 인수로 전달받은 값 0을 배열의 처음부터 끝까지 요소로 채움
arr.fill(0);

// fill 메서드는 원본 배열을 직접 변경함
console.log(arr); // [0, 0, 0]
```

1. 두 번째 인수로 요소 채우기를 시작할 인덱스를 전달

    ```javascript
    const arr = [1, 2, 3];
    
    // 인수로 전달받은 값 0을 배열의 인덱스 1부터 끝까지 요소로 채움
    arr.fill(0, 1);
    
    // fill 메서드는 원본 배열을 직접 변경함
    console.log(arr); // [1, 0, 0]
    ```

2. 세 번째 인수로 요소 채우기를 멈출 인덱스를 전달

    ```javascript
    const arr = [1, 2, 3];
    
    // 인수로 전달받은 값 0을 배열의 인덱스 1부터 3 이전(인덱스 3 미포함)까지 요소로 채움
    arr.fill(0, 1, 3);
    
    // fill 메서드는 원본 배열을 직접 변경함
    console.log(arr); // [1, 0, 0, 4, 5]
    ```

3. 배열을 생성하면서 특정 값으로 요소를 채울 수 있음

    ```javascript
    // 인수로 전달받은 정수만큼 요소를 생성하고 0부터 1씩 증가하면서 요소를 채움
    const sequences = (length = 0) => Array.from({ length }, (_, i) => i);
    // const sequences = (length = 0) => Array.from(new Array(length), (_, i) => i);
    
    console.log(sequences(3)); // [0, 1, 2]
    ```

### `Array.prototype.includes`

- ES7에서 도입된 includes 메서드는 배열 내에 특정 요소가 포함되어 있는지 확인하여, true 또는 false를 반환

```javascript
const arr = [1, 2, 3];

// 배열에 요소 2가 포함되어 있는지 확인
arr.includes(2); // -> true

// 배열에 요소 100이 포함되어 있는지 확인
arr.includes(100); // -> false
```

### `Array.prototype.flat`

- ES10에 도입된 flat 메서드는 인수로 전달한 깊이만큼 재귀적으로 배열을 평탄화 함

```javascript
[1, [2, 3, 4, 5]].flat(); // -> [1,2,3,4,5]

// 중첩 배열을 평탄화할 깊이를 인수로 전달할 수 있음
[1, [2, [3, [4]]]].flat(); // -> [1, 2, [3, [4]]]
[1, [2, [3, [4]]]].flat(1); // -> [1, 2, [3, [4]]]

// 2단계 깊이까지 평탄화
[1, [2, [3, [4]]]].flat(2); // -> [1, 2, 3, [4]]
[1, [2, [3, [4]]]].flat().flat(); // -> [1, 2, 3, [4]]

// Infinity로 지정하여 중첩 배열 모두 평탄화
[1, [2, [3, [4]]]].flat(Infinity); // -> [1, 2, 3, 4]
```

- 중첩 배열을 평탄화할 깊이를 인수(기본값은 1)로 전달할 수 있음
- 인수로 Infinity를 전달하면 중첩 배열 모두를 평탄화함

## 스택(Stack)

<img alt="javascript_array7" src="/static/images/javascript_array7.png" width="650"/>

- 데이터를 마지막에 밀어 넣고, 마지막에 밀어 넣은 데이터를 먼저 꺼내는 `후입 선출(LIFO - Last In First Out)` 방식의 자료구조
- 가장 마지막에 밀어 넣은 `최신 데이터를 먼저 취득`

### 스택 구현 - 생성자 함수로 구현

```javascript
const Stack = (function () {
	function Stack(array = []){
		if(!Array.isArray(array)) {
			throw new TypeError(`${array} is not an array`);
		}
		this.array = array;
	}
	Stack.prototype = {
		constructor: Stack,
		push(value){
			return this.array.push(value);
		},
		pop(){
			return this.array.pop();
		},
		entries(){
			return [...this.array];
		}
	};
	
	return Stack;
}());

const stack = new Stack([1,2]);
console.log(stack.entries()); // [1, 2]

stack.push(3);
console.log(stack.entries()); // [1, 2, 3]

stack.pop();
console.log(stack.entries()); // [1, 2]
```

### 스택 구현 - 클래스로 구현

```javascript
class Stack{
	#array; // private class member

	constructor(array = []) {
		if(!Array.isArray(array)){
			throw new TypeError(`${array} is not an array.`);
		}
		this.#array = array;
	}

	// 스택의 가장 마지막에 데이터를 밀어 넣음
	push(value){
		return this.#array.push(value);
	}

	// 스택의 가장 마지막 데이터, 즉 가장 나중에 밀어 넣은 최신 데이터를 꺼냄
	pop(){
		return this.#array.pop();
	}

	// 스택의 복사본 배열을 반환
	entries(){
		return [...this.#array];
	}
}

const stack = new Stack([1, 2]);
console.log(stack.entries()); // [1, 2]

stack.push(3);
console.log(stack.entries()); // [1, 2, 3]

stack.pop();
console.log(stack.entries()); // [1, 2]
```

## 큐(Queue)

<img alt="javascript_array8" src="/static/images/javascript_array8.png" width="650"/>

- 데이터를 마지막에 밀어 넣고 처음 데이터, 즉 가장 먼저 밀어 넣은 데이터를 먼저 꺼내는 `선입 선출(FIFO - FIrst In First Out)` 방식의 자료구조
  - 언제나 데이터를 밀어 넣은 순서대로 취득함

### 큐 - 생성자 함수로 구현

```javascript
const Queue = (function() {
	function Queue(array = []){
		if (!Array.isArray(array)){
			throw new TypeError(`${array} is not an array.`);
		}
		this.array = array;
	}
	
	Queue.prototype = {
		constructor: Queue,
		enqueue(value){
			return this.array.push(value);
		},
		dequeue(){
			return this.array.shift();
		},
		entries(){
			return [...this.array];
		}
	};
		
	return Queue;
}());

const queue = new Queue([1, 2]);
console.log(queue.entries()); // [1, 2]

queue.enqueue(3);
console.log(queue.entries()); // [1, 2, 3]

queue.dequeue();
console.log(queue.entries()); // [2, 3]
```

### 큐 - 클래스로 구현

```javascript
class Queue{
  #array; // private class member
  constructor(array = []) {
    if (!Array.isArray(array)) {
      throw new TypeError(`${array] is not an array.`
    );			
        }
        this.#array = array;
      }
    
      // 큐의 가장 마지막에 데이터를 밀어 넣음
      enqueue(value){
        return this.#array.push(value);
      }
    
      // 큐의 가장 처음 데이터, 즉 가장 먼저 밀어 넣은 데이터를 꺼냄
      dequeue(){
        return this.#array.shift();
      }
    
      // 큐의 복사본 배열을 반환
      entries(){
        return [...this.#array];
      }
    }
    
    const queue = new Queue([1, 2]);
    console.log(queue.entries()); // [1, 2]
    
    queue.enqueue(3);
    console.log(queue.entries()); // [1, 2, 3]
    
    queue.dequeue();
    console.log(queue.entries()); // [2, 3]
```

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 492 ~ 528p