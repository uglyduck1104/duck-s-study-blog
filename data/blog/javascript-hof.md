---
title: 배열 고차 함수
date: '2022-06-02'
tags: ['javascript']
draft: false
summary: 고차 함수(Higher-Order Function)는 함수를 인수로 전달받거나 함수를 반환하는 함수
layout: PostSimple
authors: ['default']
---

# 배열 고차 함수

- 고차 함수(Higher-Order Function)는 `함수를 인수로 전달`받거나 `함수를 반환`하는 함수
- 외부 상태의 변경이나 가변 데이터를 피하고 `불변성을 지향`하는 함수형 프로그래밍 기반
  - 순수 함수와 보조 함수의 조합을 통해 로직 내에 존재하는 `조건문과 반복문을 제거`하여 `복잡성 해결`
  - `변수의 사용을 억제`하여 `상태 변경을 피하는` 프로그래밍 패러다임

> `순수 함수`를 통해 부수 효과를 최대한 억제하여 오류를 피하고 `프로그래밍의 안정성`을 높이려는 일환

## `Array.prototype.sort`

- 배열의 요소를 `정렬`하고 반환함
- 원본 배열을 `직접 변경`
- `디폴트값은 오름차순`

### 문자열 정렬

```javascript
const fruits = ['Banana', 'Orange', 'Apple'];

// 오름차순(ascending) 정렬
fruits.sort();

// 원본 배열 직접 변경
console.log(fruits); // ['Apple', 'Banana', 'Orange']

// 내림차순 변경
fruits.reverse();

// reverse 메서드도 원본 배열을 직접 변경
console.log(fruits); // ['Orange', 'Banana', 'Apple']
```

### 숫자 정렬

```javascript
const points = [40, 100, 1, 5, 2, 25, 10];

points.sort();

// 숫자 요소들로 이루어진 배열은 의도한 대로 정렬되지 않음
console.log(points); // [1, 10, 100, 2, 25, 40, 5]
```

- sort 메서드의 기본 정렬 순서는 `유니코드 코드 포인트의 순서`를 따르므로 배열의 요소가 `숫자 타입`이라해도 배열의 요소를 `일시적으로 문자열로 변환 후` 유니코드 코드 포인트의 순서를 기준으로 정렬함
- 올바른 결과값을 얻기 위해서는 정렬 순서를 정의하는 `비교 함수를 인수로 전달`해야 함(⭐⭐)

    ```javascript
    const points = [40, 100, 1, 5, 2, 25, 10];
    
    // 숫자 배열의 오름차순 정렬. 비교 함수의 반환값이 0보다 작으면 a를 우선하여 정렬
    points.sort((a, b) => a - b);
    console.log(points); // [1, 2, 5, 10, 25, 40, 100]
    
    // 숫자 배열에서 최소/최대값 취득
    console.log(points[0], points[points.length - 1]); // 1 100
    
    // 숫자 배열의 내림차순 정렬. 비교 함수의 반환값이 0보다 작으면 b를 우선하여 정렬
    points.sort((a, b) => b - a);
    console.log(points); // [100, 40, 25, 10, 5, 2, 1]
    
    // 숫자 배열에서 최소/최대값 취득
    console.log(points[points.length - 1], points[0]); // 1 100
    ```

### 객체 정렬

```javascript
const todos = [
  {id: 4, content: 'JavaScript'},
  {id: 1, content: 'HTML'},
  {id: 2, content: 'CSS'},
];

// 비교 함수 -> 매개 변수 key는 프로퍼티 키
function compare(key) {
  // 프로퍼티 값이 문자열인 경우 - 산술 연산으로 비교하면 NaN이 나오므로 비교 연산 사용
  // 비교 함수는 양수/음수/0을 반환하면 되므로 - 산술 연산 대시 비교 연산을 사용할 수 있음
  return (a, b) => (a[key] > b[key] ? 1 : (a[key] < b[key] ? -1 : 0));
}

// id를 기준으로 오름차순 정렬
todos.sort(compare('id'));
console.log(todos);
/*
[
  { id: 1, content: 'HTML' },
  { id: 2, content: 'CSS' },
  { id: 4, content: 'JavaScript' },
*/

// content를 기준으로 오름차순 정렬
todos.sort(compare('content'));
console.log(todos);
/*
[
  { id: 2, content: 'CSS' },
  { id: 1, content: 'HTML' },
  { id: 4, content: 'JavaScript' },
*/
```

- 자바스크립트가 지원하는 sort 메서드의 정렬 알고리즘은 현재(ES10 이후) `timsort 알고리즘을 사용`
  - `ES10 이전`에는 동일한 값의 요소가 중복되어 있을 때 초기 순서와 변경될 수 있는 불안정한 알고리즘인 `quicksort 알고리즘`을 썼음

## `Array.prototype.forEach`

- 조건문이나 반복문의 로직 흐름 상 이해하기 어려운 부분을 보완해서 나온 메서드
- for문을 대체할 수 있으며, 내부에서 반복문을 통해 `자신을 호출한 배열을 순회`하고 수행해야 할 처리를 `콜백 함수로 전달받아 반복 호출`

```javascript
const numbers = [1, 2, 3];
const pows = [];

// forEach 메서드는 numbers 배열의 모든 요소를 순회하면서 콜백 함수를 반복 호출함
numbers.forEach(item => pows.push(item ** 2));
console.log(pows);
[1, 4, 9]
```

- numbers 배열의 모든 요소를 순회하면서 콜백 함수를 반복 호출함
  - 콜백함수는 forEach 메서드를 호출한 배열의 `요소값`과 `인덱스`, forEach 메서드를 호출한 배열(`this`)을 `순차적으로 전달`함

    ```javascript
    // forEach 메서드는 콜백 함수를 호출하면서 3개(요소값, 인덱스, this)의 인수를 전달
    [1, 2, 3].forEach((item, index, arr) => {
      console.log(`요소값: ${item}, 인덱스: ${index}, this: ${JSON.stringify(arr)}`);
    });
    /*
      요소값: 1, 인덱스: 0, this: [1,2,3]
      요소값: 2, 인덱스: 1, this: [1,2,3]
      요소값: 3, 인덱스: 2, this: [1,2,3]
    */
    ```

- 원본 배열을 변경하지 않지만 콜백 함수를 통해 변경할 수 있음

    ```javascript
    const numbers = [1, 2, 3];
    
    // 콜백 함수의 세 번째 매개변수 arr은 원본 배열 numbers를 가리킴
    // 따라서 콜백 함수의 세 번째 매개변수 arr을 직접 변경하면 원본 배열 numbers가 변경됨
    number.forEach((item, index, arr) => { arr[index] = item ** 2;});
    console.log(numbers); // [1, 4, 9]
    ```

- 콜백 함수는 일반 함수로 호출되므로 콜백 함수 내부의 this는 undefined를 가리킴

### forEach 메서드의 폴리필

```javascript
if (!Array.prototype.forEach) {
  Array.prototype.forEach = function (callback, thisArg) {
    if (typeof callback !== 'function') {
      throw new TypeError(callback + ' is not a function');
    }

    // this로 사용할 두 번째 인수를 전달받지 못하면 전역 객체를 this로 사용함
    thisArg = thisArg || window;

    // for 문으로 배열을 순회하면서 콜백 함수 호출
    for (var i = 0; i < this.length; i++) {
      // call 메서드를 통해 thisArg를 전달하면서 콜백 함수를 호출
      // 이때 콜백 함수의 인수로 배열 요소, 인덱스, 배열 자신을 전달
      callback.call(thisArg, this[i], i, this);
    }
  }
}
```

- forEach 메서드는 for 문과 달리 break, continue 문을 사용할 수 없음

### 정리

- for문에 비해 성능은 좋지 않으나, 가독성은 더 좋으므로 복잡한 코드나 높은 성능이 필요한 경우가 아니라면 forEach 메서드를 사용하는 것이 좋음

## `Array.prototype.map`

- 자신을 호출한 배열의 모든 요소를 순회하면서 인수로 전달받은 콜백 함수를 반복 호출
- 콜백 함수의 반환값들로 구성된 `새로운 배열 반환`
- 원본 배열은 `변하지 않음`
- forEach 메서드와 map 메서드의 공통점은 배열의 모든 요소를 순회하면서 인수로 전달받은 콜백 함수 반복 호출함
  - map 메서드는 하므로 언제나 undefined를 반환하는 forEach 메서드와 차이가 있음

| 구분   | forEach                | Map                         |
|------|------------------------|-----------------------------|
| 반환 값 | 언제나 undefined를 반환      | 콜백 함수의 반환값들로 구성된 새로운 배열을 반환 |
| 용도   | 단순히 반복문을 대체하기 위한 고차 함수 | 요소값을 다른 값으로 매핑한 새로운 배열을 생성  |

> map 메서드를 호출한 배열과 map 메서드가 생성하여 반환한 배열은 1:1 매핑함

```javascript
const numbers = [1, 4, 9];

const roots = numbers.map(item => Math.sqrt(item));

console.log(roots);   // [1, 2, 3]
console.log(numbers); // [1, 4, 9]
```

<img alt="javascript_hof1.png" src="/static/images/javascript_hof1.png" width="700"/>

```javascript
// map 메서드는 콜백 함수를 호출하면서 3개(요소값, 인덱스, this)의 인수를 전달함
[1, 2, 3].map((item, index, arr) => {
  console.log(`요소값: ${item}, 인덱스: ${index}, this: ${JSON.stringify(arr)}`);
  return item;
})

/*
  요소값: 1, 인덱스: 0, this: [1,2,3]
  요소값: 2, 인덱스: 1, this: [1,2,3]
  요소값: 1, 인덱스: 2, this: [1,2,3]
*/
```

## `Array.prototype.filter`

- 배열의 모든 요소를 순회하면서 인수로 전달받은 콜백 함수를 반복 호출
  - 콜백 함수의 `반환값이 true인 요소`로만 구성된 `새로운 배열 반환`
- 주로 필터링 조건을 만족하는 특정 요소만 추출하여 새로운 배열을 만들고 싶을 때 사용

> filter 메서드가 생성하여 반환한 새로운 배열의 length 프로퍼티 값은 filter 메서드를 호출한 배열의 length 프로퍼티 값과 같거나 작음

<img alt="javascript_hof2.png" src="/static/images/javascript_hof2.png" width="700"/>

```javascript
// filter 메서드는 콜백 함수를 호출하면서 3개(요소값, 인덱스, this)의 인수를 전달
[1, 2, 3].filter((item, index, arr) => {
  console.log(`요소값: ${item}, 인덱스: ${index}, this: ${JSON.stringify(arr)}`);
  return item % 2;
});

/*
  요소값: 1, 인덱스 0, this: [1,2,3]
  요소값: 2, 인덱스 1, this: [1,2,3]
  요소값: 3, 인덱스 2, this: [1,2,3]
*/
```

### filter 메서드로 특정 요소 제거하기

```javascript
class Users {
  constructor() {
    this.users = [
      {id: 1, name: 'Lee'},
      {id: 2, name: 'Kim'}
    ];
  }

  // 요소 추출
  findById(id) {
    // id가 일치하는 사용자만 반환함
    return this.users.filter(user => user.id === id);
  }

  // 요소 제거
  remove(id) {
    // id가 일치하지 않는 사용자 제거
    this.users = this.users.filter(user => user.id !== id);
  }
}

const users = new User();

let user = users.findById(1);
console.log(user); // [{ id: 1, name: 'Lee' }]

// id가 1인 사용자를 제거
users.remove(1);

user = users.findById(1);
console.log(user); // []
```

- 특정 요소를 제거할 경우 특정 요소가 중복되어 있다면 `중복된 요소가 모두 제거됨`
- 특정 요소 하나만 제거하려먼 indexOf 메서드를 통해 특정 요소의 인덱스를 취득한 다음 splice 메서드 사용

## `Array.prototype.reduce`

- 배열의 모든 요소를 순회하면서 인수로 전달받은 콜백 함수를 반복 호출
  - 콜백 함수의 반환값을 다음 순회 시에 콜백 함수의 첫 번째 인수로 전달하면서 콜백 함수를 호출하여 `하나의 결과값을 만들어 반환`
- 원본 배열은 변경되지 않음
- 4개의 인수
  - 초기값 또는 콜백 함수의 이전 반환값
  - reduce 메서드를 호출한 배열의 요소값과 인덱스
  - reduce 메서드를 호출한 배열 자체(this)

```javascript
// 1부터 4까지 누적 연산
const sum = [1, 2, 3, 4].reduce((accumulator, currentValue, index, array) => accumulator + currentValue, 0);
console.log(sum); // 10
```

| 구분      | accumulator | currentValue | index | array     | 반환값                            |
|---------|-------------|--------------|-------|-----------|--------------------------------|
| 첫 번째 순회 | 0(초기값)      | 1            | 0     | [1,2,3,4] | 1(accumulator + currentValue)  |
| 두 번째 순회 | 1           | 2            | 1     | [1,2,3,4] | 3(accumulator + currentValue)  |
| 세 번째 순회 | 3           | 3            | 2     | [1,2,3,4] | 6(accumulator + currentValue)  |
| 네 번째 순회 | 6           | 4            | 3     | [1,2,3,4] | 10(accumulator + currentValue) |

![javascript_hof3.png](/static/images/javascript_hof3.png)

- 초기값과 배열의 첫 번째 요소값을 콜백 함수에게 인수로 전달하면서 호출
- 다음 순회에는 콜백 함수의 반환값과 두 번째 요소값을 콜백 함수의 인수로 전달하면서 호출

### 평균 구하기

```javascript
const values = [1, 2, 3, 4, 5, 6];

const average = values.reduce((acc, cur, i, {length}) => {
  // 마지막 순회가 아니면 누적값을 반환하고 마지막 순회면 누적값으로 평균을 구해 반환
  return i === length - 1 ? (acc + cur) / length : acc + cur;
}, 0);

console.log(average); // 3.5
```

### 최대값 구하기

```javascript
const values = [1, 2, 3, 4, 5];

const max = values.reduce((acc, cur) => (acc > cur ? acc : cur), 0);
console.log(max); // 5
```

- 최대값을 구할 때에는 reduce 메서드보다 Math.max 메서드를 사용하는 게 더 직관적임

    ```javascript
    const values = [1, 2, 3, 4, 5];
    
    const max = Math.max(...values);
    console.log(max); // 5
    ```

### 요소의 중복 횟수 구하기

```javascript
const fruits = ['banana', 'apple', 'orange', 'orange', 'apple'];

const count = fruits.reduce((acc, cur) => {
  // 첫 번째 순회 시 acc는 초기값인 {}이고 cur은 첫 번째 요소인 'banana'
  // 초기값으로 전달받은 빈 객체에 요소값인 cur을 프로퍼티 키로, 요소의 개수를 프로퍼티 값으로 할당
  // 만약 프로퍼티 값이 undefined(처음 등장하는 요소)이면 프로퍼티 값을 1로 초기화
  acc[cur] = (acc[cur] || 0) + 1;
  return acc;
}, {});

// 콜백 함수는 총 5번 호출
/*
  {banana: 1} => {banana: 1, apple: 1} => {banana: 1, apple: 1, orange: 1}
   => {banana: 1, apple: 1, orange: 2} => {banana: 1, apple: 2, orange: 2}
*/

console.log(count); // { banana: 1, apple: 2, orange: 2 }
```

### 중첩 배열 평탄화

```javascript
const values = [1, [2, 3], 4, [5, 6]];

const flatten = values.reduce((acc, cur) => acc.concat(cur), []);
// [1] => [1, 2, 3] => [1, 2, 3, 4] => [1, 2, 3, 4, 5, 6]

console.log(flatten); // [1, 2, 3, 4, 5, 6]
```

- 중첩 배열을 평탄화 할 때는 ES10에 도입된 Array.prototype.flat 메서드를 사용하는게 더 직관적임

    ```javascript
    [1, [2, 3, 4, 5]].flat(); // -> [1, 2, 3, 4, 5]
    
    // 인수 2는 중첩 배열을 평탄화하기 위한 깊이 값
    [1, [2, 3, [4, 5]]].flat(2); // -> [1, 2, 3, 4, 5]
    ```

### 중복 요소 제거

```javascript
const values = [1, 2, 1, 3, 5, 4, 5, 3, 4, 4];

const result = values.reduce(
  (unique, val, i, _values) =>
    // 현재 순회 중인 요소의 인덱스 i가 val의 인덱스와 같다면 val은 처음 순회하는 요소
    // 현재 순회 중인 요소의 인덱스 i가 val의 인덱스와 다르다면 val은 중복된 요소
    // 처음 순회하는 요소만 초기값 []가 전달된 unique 배열에 담아 반환하면 중복된 요소는 제거됨
    _values.indexOf(val) === i ? [...unique, val] : unique,
  []
);

console.log(result); // [1, 2, 3, 4, 5]
```

- 중복 요소를 제거할 때는 reduce 메서드보다 filter 메서드를 사용하는 방법이 더 직관적임

    ```javascript
    const values = [1, 2, 1, 3, 5, 4, 5, 3, 4, 4];
    
    const result = values.filter((val, i, _values) => _values.indexOf(val) === i);
    console.log(result); // [1, 2, 3, 5, 4]
    ```

- 중복되지 않는 유일한 값의 집합인 Set을 사용할 수 있음

    ```javascript
    const values = [1, 2, 1, 3, 5, 4, 5, 3, 4, 4];
    
    const result = [...new Set(values)];
    console.log(result); // [1, 2, 3, 5, 4]
    ```

> map, filter, soe, every, find 같은 배열의 고차 함수는 reduce 메서드로 구현할 수 있음

### 주의할 점

- reduce 메서드를 호출할 때는 `언제나 초기값을 전달하는 것이 안전함`(⭐⭐⭐)
  - `빈 배열`로 reduce 메서드를 `호출하는 경우`와 `특정 프로퍼티 값을 합산`하는 경우에는 오류가 발생할 우려가 있으므로 초기값을 전달하는 것이 좋음

## `Array.prototype.some`

- 콜백 함수의 반환값이 단 한번이라도 참이면 true, 모두 거짓이면 false를 반환
  - 배열의 요소 중에 콜백 함수를 통해 정의한 조건을 만족하는 요소가 `1개 이상 존재하는지 확인하여 불리언 타입으로 반환`
  - `빈 배열의 경우 언제나 false 반환`

```javascript
// 배열의 요소 중 10보다 큰 요소가 1개 이상 존재하는지 확인
[5, 10, 15].some(item => item > 10); // -> true

// 배열의 요소 중 0보다 작은 요소가 1개 이상 존재하는지 확인
[5, 10, 15].some(item => item < 0); // -> false

// 배열의 요소 중 'banana'가 1개 이상 존재하는지 확인
['apple', 'banana', 'mango'].some(item => item === 'banana'); // -> true

// some 메서드를 호출한 배열이 빈 배열인 경우 언제나 false를 반환
[].some(item => item > 3); // -> false
```

## `Array.prototype.every`

- 콜백 함수의 반환값이 모두 참이면 true, 단 한 번이라도 거짓이면 false를 반환
  - 배열의 모든 요소가 콜백 함수를 통해 정의한 조건을 `모두 만족`하는지 확인하여 `불리언 타입으로 반환`
  - `빈 배열인 경우 언제나 true 반환`

```javascript
// 배열의 모든 요소가 3보다 큰지 확인
[5, 10, 15].every(item => item > 3); // -> true

// 배열의 모든 요소가 10보다 큰지 확인
[5, 10, 15].every(item => item > 10); // -> false

// every 메서드를 호출한 배열이 빈 배열인 경우 언제나 true 반환
[].every(item => item > 3); // -> true
```

## `Array.prototype.find`

- ES6에 도입된 find 메서드는 호출한 배열의 요소를 순회하면서 인수로 전달된 콜백 함수를 호출하여 `반환값이 true인 첫 번째 요소를 반환`함
  - 존재하지 않는다면 `undefined를 반환`

```javascript
const users = [
  {id: 1, name: 'Lee'},
  {id: 2, name: 'Kim'},
  {id: 2, name: 'Choi'},
  {id: 1, name: 'Lee'},
];

// id가 2인 첫 번째 요소 반환 -> 배열이 아닌 요소를 반환함
users.find(user => user.id === 2); // -> {id: 2, name: 'Kim'}
```

- filter 메서드와 달리 find 메서드는 `첫 번째 요소만을 반환`하므로 배열이 아닌 `해당 요소값을 결과로 반환함`

## `Array.prototype.findIndex`

- ES6에 도입된 메서드로 호출한 배열의 요소를 순회하면서 인수로 전달된 콜백 함수를 호출하여 `반환값이 true인 첫 번째 요소의 인덱스를 반환`함
  - 존재하지 않는다면 `-1 반환`

```javascript
const users = [
  {id: 1, name: 'Lee'},
  {id: 2, name: 'Kim'},
  {id: 2, name: 'Choi'},
  {id: 3, name: 'Park'}
];

// id가 2인 요소의 인덱스 구하기
users.findIndex(user => user.id === 2); // -> 1

// name이 'Park'인 요소의 인덱스 구하기
users.findIndex(user => user.name === 'Park'); // -> 3

// 콜백 함수 추상화
function predicate(key, value) {
  // key와 value를 기억하는 클로저 반환
  return item => item[key] === value;
}

// id가 2인 요소의 인덱스 구하기
users.findIndex(predicate('id', 2)); // -> 1

// name이 'Park'인 요소의 인덱스 구하기
users.findIndex(predicate('name', 'Park')); // -> 3
```

## `Array.prototype.flatMap`

- ES10에서 도입된 flatMap 메서드는 map 메서드를 통해 생성된 새로운 배열을 평탄화함

```javascript
const arr = ['hello', 'world'];

// map과 flat을 순차적으로 실행하는 효과
arr.map(x => x.split('')).flat();
// -> ['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd']

// flatMap은 map을 통해 생성된 새로운 배열을 평탄화함
arr.flatMap(x => x.split(''));
// -> ['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd']
```

> flat 메서드처럼 인수를 전달하여 평탄화 깊이는 지정하지 못하고 평탄화 깊이를 지정하려면 map + flat 메서드 조합으로 각각 호출해야 함

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 529 ~ 551p
