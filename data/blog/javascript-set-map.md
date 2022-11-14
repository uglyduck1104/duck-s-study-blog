---
title: Set과 Map
date: '2022-06-13'
tags: ['javascript']
draft: false
summary: Set과 Map의 개념과 활용
layout: PostSimple
authors: ['default']
---

# Set과 Map

## Set

- 중복되지 않는 유일한 값들의 집합
  - 배열과 유사하나 다음과 같은 차이가 있음

| 구분                   | 배열  | Set 객체 |
|----------------------|-----|--------|
| 동일한 값을 중복하여 포함할 수 있음 | O   | X      |
| 요소 순서에 의미가 있음        | O   | X      |
| 인덱스로 요소에 접근할 수 있음    | O   | X      |
- Set은 수학적 집합을 구현하기 위한 `자료구조`
  - `교집합`, `합집합`, `차집합`, `여집합`을 구현할 수 있음

### Set 객체의 생성

- Set 객체는 Set `생성자 함수`로 생성함
  - 인수를 전달하지 않을 경우 빈 Set 객체가 생성됨
- Set `생성자 함수`는 `이터러블`을 인수로 전달받아 Set 객체를 생성함
  - 이터러블의 `중복된 값`은 Set 객체에 요소로 `저장되지 않음`

```jsx
const set1 = new Set([1, 2, 3, 3]);
console.log(set1); // Set(3) {1, 2, 3}

const set2 = new Set('hello');
console.log(set2); // Set(4) {"h", "e", "l", "o"}
```

### 중복 요소 제거

```jsx
// 배열의 중복 요소 제거
const uniq = array => array.filter((v, i, self) => self.indexOf(v) === i);
console.log(uniq([2, 1, 2, 3, 4, 3, 4])); // [2, 1, 3, 4]

// Set을 사용한 배열의 중복 요소 제거
const uniq = array => [...new Set(array)];
console.log(uniq([2, 1, 2, 3, 4, 3, 4])); // [2, 1, 3, 4]
```

### 요소 개수 확인

- Set 객체의 `요소 개수`를 확인할 때는 `Set.prototype.size` 프로퍼티를 사용함

    ```jsx
    const { size } = new Set([1, 2, 3, 3]);
    console.log(size); // 3
    ```

  - size 프로퍼티는 setter 함수 없이 `getter 함수만 존재`하는 접근자 프로퍼티

### 요소 추가

- Set 객체에 요소를 추가할 때는 `Set.prototype.add` 메서드 사용

    ```jsx
    const set = new Set();
    console.log(set); // Set(0) {}
    
    set.add(1);
    console.log(set); // Set(1) {1}
    ```

  - add 메서드는 새로운 요소가 추가된 `Set 객체를 반환`함
    - 따라서 add 메서드를 연속적으로 호출(`method chaining`)할 수 있음
- 중복된 요소의 추가는 허용되지 않고 별도의 에러를 발생하지 않고 무시됨
- 객체나 배열과 같이 자바스크립트의 `모든 값을 요소로 저장`할 수 있음

### 요소 존재 여부 확인

- `Set.prototype.has` 메서드 사용
  - 특정 요소의 존재 여부를 나타내는 불리언 값을 반환

    ```jsx
    const set = new Set([1, 2, 3]);
    
    console.log(set.has(2)); // true
    console.log(set.has(4)); // false
    ```

### 요소 삭제

- `Set.prototype.delete` 메서드 사용
  - delete 메서드는 삭제 성공 여부를 나타내는 `불리언 값을 반환`
  - 삭제하려는 `요소값`을 인수로 전달
  - Set 객체는 `순서에 의미가 없어` 배열과 같이 인덱스를 가지지 않음
- 삭제 성공 여부를 나타내는 불리언 값을 반환하므로 `Set.prototype.add` 메서드와 달리 연속적으로 호출할 수 없음

### 요소 일괄 삭제

- Set 객체의 모든 요소를 일괄 삭제하려면 `Set.prototype.clear` 메서드를 사용함
  - 언제나 `undefined를 반환`

      ```jsx
      const set = new Set([1, 2, 3]);
      
      set.clear();
      console.log(set); // Set(0) {}
      ```

### 요소 순회

- Set 객체의 요소를 순회하는 메서드 지원
- `Set.prototype.forEach`
  - `Array.prototype.forEach` 메서드와 유사하게 콜백 함수와 forEach 메서드의 콜백 함수 내부에서 `this`로 사용될 객체를 인수로 전달
    - **첫 번째 인수**: 현재 순회 중인 요소값
    - **두 번째 인수**: 현재 순회 중인 요소값
    - **세 번째 인수**: 현재 순회 중인 Set 객체 자체

    ```jsx
    const set = new Set([1, 2, 3]);
    
    set.forEach((v, v2, set) => console.log(v, v2, set));
    
    /*
    1 1 Set(3) {1, 2, 3}
    2 2 Set(3) {1, 2, 3}
    3 3 Set(3) {1, 2, 3}
    */
    ```

  - Set 객체는 `이터러블`이므로 `for…of` 문으로 순회할 수 있음

### 집합 연산

- `교집합`
  - 집합 A와 집합 B의 `공통 요소`
  - `Set 객체`로 연산

      ```jsx
      Set.prototype.intersection = function (set) {
        const result = new Set();
      
        for(const value of set){
          // 2개의 set의 요소가 공통되는 요소이면 교집합의 대상
          if(this.has(value)) result.add(value);
        }
        
        return result;
      };
      
      const setA = new Set([1, 2, 3, 4]);
      const setB = new Set([2, 4]);
      
      // setA와 setB의 교집합
      console.log(setA.intersection(setB)); // Set(2) {2, 4}
      // setA와 setB의 교집합
      console.log(setA.intersection(setA)); // Set(2) {2, 4}
      ```

  - `Filter` 고차함수로 구현

      ```jsx
      Set.prototype.intersection = function (set) {
        return new Set([...this].filter(v => set.has(v)));
      };
      ```


- `합집합`
  - 집합 A와 B의 `중복 없는 모든 요소`
  - `Set 객체`로 연산

      ```jsx
      Set.prototype.union = function (set) {
        // this(Set 객체)를 복사
        const result = new Set(this);
        
        for(const value of set){
          // 합집합은 2개의 Set 객체의 모든 요소로 구성된 집합. 중복된 요소 미포함
          result.add(value);
        }
      
        return result;
      };
      
      const setA = new Set([1, 2, 3, 4]);
      const setB = new Set([2, 4]);
      
      // setA와 setB의 합집합
      console.log(setA.union(setB)); // Set(4) {1, 2, 3, 4}
      // setB와 setA의 합집합
      console.log(setB.union(setA)); // Set(4) {2, 4, 1, 3}
      ```

  - `ES6 스프레드 문법` 사용

      ```jsx
      Set.prototype.union = function (set) {
        return new Set([...this, ...set]);
      };
      ```


- `차집합`
  - 집합 A에는 존재하지만 집합 B에는 존재하지 않는 요소로 구성
  - `Set 객체`로 연산

      ```jsx
      Set.prototype.difference = function (set){
        // this(Set 객체)를 복사
        const result = new Set(this);
      
        for (const value of set){
          // 차집합은 어느 한쪽 집합에만 존재하지만 다른 한쪽 집합에는 존재하지 않는 요소로 구성된 집합
          result.delete(value);
        }
      
        return result;
      };
      
      const setA = new Set([1, 2, 3, 4]);
      const setB = new Set([2, 4]);
      
      // setA와 setB의 합집합
      console.log(setA.difference(setB)); // Set(2) {1, 3}
      // setB와 setA의 합집합
      console.log(setB.difference(setA)); // Set(0) {}
      ```

  - `Filter` 고차 함수 구현

      ```jsx
      Set.prototype.difference = function (set) {
        return new Set([...this].filter(v => !set.has(v)));
      };
      ```


- `부분 집합과 상위 집합`
  - 집합 A가 집합 B에 포함되는 경우 집합 A는 집합 B의 `부분 집합`이며, 집합 B는 집합 A의 `상위 집합`

      ```jsx
      // this가 subset의 상위 집합인지 확인
      Set.prototype.isSuperset = function (subset){
        for(const value of subset) {
          // superset의 모든 요소가 subset의 모든 요소를 포함하는지 확인
          if(!this.has(value)) return false;
        }
      
        return true;
      };
      
      const setA = new Set([1, 2, 3, 4]);
      const setB = new Set([2, 4]);
      
      // setA가 setB의 상위 집합인지 확인
      console.log(setA.isSuperset(setB)); // true
      // setB가 setA의 상위 집합인지 확인
      console.log(setA.isSuperset(setA)); // false
      ```

      ```jsx
      // this가 subset의 상위 집합인지 확인
      Set.prototype.isSuperset = function (subset){
        const supersetArr = [...this];
        return [...subset].every(v => supersetArr.includes(v));
      };
      
      const setA = new Set([1, 2, 3, 4]);
      const setB = new Set([2, 4]);
      
      // setA가 setB의 상위 집합인지 확인
      console.log(setA.isSuperset(setB)); // true
      // setB가 setA의 상위 집합인지 확인
      console.log(setA.isSuperset(setA)); // false
      ```

## Map

- 키와 값의 쌍으로 이루어진 컬레션
  - Map 객체는 객체와 유사하지만 차이점이 있음

| 구분            | 객체                      | Map 객체       |
|---------------|-------------------------|--------------|
| 키로 사용할 수 있는 값 | 문자열 또는 심벌 값             | 객체를 포함한 모든 값 |
| 이터러블          | X                       | O            |
| 요소 개수 확인      | Object.keys(obj).length | map.size     |

### Map 객체의 생성

- Map 객체는 `Map 생성자 함수`로 생성
  - Map 생성자 함수에 인수를 전달하지 않으면 빈 Map 객체 생성
  - Map 생성자 함수는 이터러블을 인수로 전달받아 Map 객체를 생성
  - 인수로 전달되는 `이터러블`은 `키`와 `값`의 `쌍으로 이루어진 요소`로 구성

    ```jsx
    const map1 = new Map([['key1', 'value1'], ['key2', 'value']]);
    console.log(map1); // Map(2) {"key1" => "value1", "key2" => "value2"}
    
    const map2 = new Map([1, 2]); // TypeError: Iterator value 1 is not an entry object
    ```

- Map 생성자 함수의 인수로 전달한 이터러블에 `중복된 키를 갖는 요소`가 존재하면 `값이 덮어써짐`
  - Map 객체에는 중복된 키를 갖는 요소가 존재할 수 없음

### 요소 개수 확인

- Map 객체의 요소 개수 확인
  - Map 객체의 `요소 개수`를 확인할 때는 `Map.prototype.size` 프로퍼티를 사용함

      ```jsx
      const { size } = new Map([['key1', 'value1'],['key2', 'value2']]);
      console.log(size); // 2
      ```

    - Set 객체와 동일하게 `size` 프로퍼티는 setter 함수 없이 `getter 함수만 존재`하는 접근자 프로퍼티

### 요소 추가

- Map 객체에 요소를 추가할 때는 `Map.prototype.set` 메서드 사용

    ```jsx
    const map = new Map();
    console.log(map); // Map(0) {}
    
    map.set('key1', 'value1');
    console.log(map); // Map(1) {"key1" => "value1"}
    ```

  - set 메서드는 새로운 요소가 추가된 `Map 객체를 반환`함
    - 따라서 set 메서드를 연속적으로 호출(`method chaining`)할 수 있음
- Map 객체는 키 타입에 제한이 없기때문에 `객체를 포함한 모든 값을 키로 사용`할 수 있음

### 요소 취득

- `Map.prototype.get`
  - get 메서드의 인수로 키를 전달 → Map 객체에서 인수로 전달한 키를 갖는 값을 반환
  - 존재하지 않으면 `undefined`를 반환

    ```jsx
    const map = new Map();
    
    const lee = { name: 'Lee' };
    const kim = { name: 'Kim' };
    
    map
    	.set(lee, 'developer')
    	.set(kim, 'designer');
    
    console.log(map.get(lee)); // developer
    console.log(map.get('key')); // undefined
    ```

### 요소 존재 여부 화인

- Map 객체에 특정 요소가 존재하는지 확인하려면 `Map.prototype.has` 메서드 사용
  - `has` 메서드는 특정 요소의 존재 여부를 나타내는 `불리언 값을 반환`

    ```jsx
    const lee = { name: 'Lee' };
    const kim = { name: 'Kim' };
    
    const map = new Map([[lee, 'developer'], [kim, 'designer']]);
    
    console.log(map.has(lee)); // true
    console.log(map.has('key')); // false
    ```

### 요소 삭제

- Map 객체의 요소를 삭제하려면 `Map.prototype.delete` 메서드 사용
  - `delete` 메서드는 삭제 성공 여부를 나타내는 불리언 값을 반환

    ```jsx
    const lee = { name: 'Lee' };
    const kim = { name: 'Kim' };
    
    const map = new Map([[lee, 'developer'], [kim, 'designer']]);
    
    map.delete(kim);
    console.log(map); // Map(1) { {name: "Lee"} => "developer" }
    ```

  - delete 메서드는 삭제 성공 여부를 나타내는 불리언 값을 반환
    - 따라서 set 메서드와 달리 연속적으로 호출할 수 없음

### 요소 순회

- Map 객체의 요소를 순회하는 메서드 지원
- `Map.prototype.forEach`
  - `Array.prototype.forEach` 메서드와 유사하게 콜백 함수와 forEach 메서드의 콜백 함수 내부에서 `this`로 사용될 객체를 인수로 전달
    - **첫 번째 인수**: 현재 순회 중인 요소값
    - **두 번째 인수**: 현재 순회 중인 요소키
    - **세 번째 인수**: 현재 순회 중인 Map 객체 자체

    ```jsx
    const lee = { name: 'Lee' };
    const kim = { name: 'Kim' };
    
    const map = new Map([[lee, 'developer'], [kim, 'designer']]);
    
    map.forEach((v, k, map) => console.log(v, k, map));
    /*
    developer {name: "Lee"} Map(2){
      {name: "Lee"} => "developer",
      {name: "Kim"} => "designer",
    }
    designer {name: "Kim"} Map(2){
      {name: "Lee"} => "developer",
      {name: "Kim"} => "designer",
    }
    */
    ```

  - Map 객체는 `이터러블`이므로 `for…of` 문으로 순회할 수 있음

| Map 메서드                 | 설명                                                     |
|-------------------------|--------------------------------------------------------|
| Map.prototype.`keys`    | Map 객체에서 `요소 키`를 값으로 갖는 이터러블이면서 동시에 이터레이터인 객체를 반환      |
| Map.prototype.`values`  | Map 객체에서 `요소 값`를 값으로 갖는 이터러블이면서 동시에 이터레이터인 객체를 반환      |
| Map.prototype.`entries` | Map 객체에서 `요소 키`와 `요소 값`으로 갖는 이터러블이면서 동시에 이터레이터인 객체를 반환 |

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 643 ~ 659p
