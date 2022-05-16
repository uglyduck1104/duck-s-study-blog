---
title: 프로퍼티 어트리뷰트
date: '2022-05-16'
tags: ['javascript']
draft: false
summary: 자바스크립트 엔진은 프로퍼티를 생성할 때 프로퍼티의 상태를 나타내는 프로퍼티 어트리뷰트를 기본값으로 자동 정의함
layout: PostSimple
authors: ['default']
---

# 프로퍼티 어트리뷰트

## 프로퍼티 어트리뷰트와 디스크립터 객체

> 자바스크립트 엔진은 프로퍼티를 생성할 때 프로퍼티의 상태를 나타내는 프로퍼티 어트리뷰트를 기본값으로 자동 정의함

### 프로퍼티 어트리뷰트

- 자바스크립트 엔진이 관리하는 내부 상태값인 `내부 슬롯`을 나타냄
- 직접 접근할 수 없으나 `Object.getOwnPropertyDescriptor` 메서드로 간접적인 확인이 가능함

### `Object.getOwnPropertyDescriptor`

```javascript
const person = {
	name: 'Lee'
};

console.log(Object.getOwnPropertyDescriptor(person, 'name'));
// {value: "Lee", writable: true, enumerable: true, configurable: true}
```

- 하나의 프로퍼티에 대해 `프로퍼티 디스크립터 객체`를 반환
- 존재하지 않는 프로퍼티나 상속받은 프로퍼티는 `undefined` 반환
- 첫 번째 매개변수: `객체의 참조` 전달
- 두번째 매개변수: `프로퍼티 키를 문자열`로 전달
- Object.getOwnPropertyDescriptors
  - ES8에 도입됐으며, 모든 프로퍼티의 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체들을 반환

## 데이터 프로퍼티와 접근자 프로퍼티

### 데이터 프로퍼티

- `키`와 `값`으로 구성된 일반적인 프로퍼티

| 프로퍼티 어트리뷰트       | 프로퍼티 디스크립터 객체의 프로퍼티 | 설명                                                                                                                                                                                      |
|------------------|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [[Value]]        | value               | - 프로퍼티 키를 통해 `프로퍼티 값에 접근`하면 반환되는 값<br/>- 프로퍼티키를 통해 프로퍼티 값을 변경하면 [[Value]]에 값을 재할당                                                                                                       |
| [[Writable]]     | writable            | - 프로퍼티 값의 `변경 가능 여부`를 나타냄(불리언 값)<br/>- [[Writable]]의 값이 `false`인 경우 해당 프로퍼티의 [[Value]]의 값을 변경할 ` 없는 `읽기 전용 프로퍼티`가 됨                                                                     |
| [[Enumerable]]   | enumerable          | - 프로퍼티 `열거 가능 여부`를 나타냄(불리언 값)<br/>- [[Enumerable]]의 값이 `false`인 경우 해당 프로퍼티는 for ... in문이나 Object.keys 메서드 등으로 `열거할 수 없음`                                                                |
| [[Configurable]] | configurable        | - 프로퍼티의 `재정의 가능 여부`를 나타냄(불리언 값)<br/>- [[Configurable]]의 값이 `false`인 경우 해당 프로퍼티의 삭제, 프로퍼티 어트리뷰트 값의 `변경이 금지`됨<br/>- [[Writable]]이 true인 경우 [[Value]]의 변경과 [[Writable]]을 false로 변경하는 것은 허용 |

```javascript
const person = {
	name: 'Lee'
};

// 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체를 취득
consolelog(Object.getOwnPropertyDescriptor(person, 'name'));
// {value: "Lee", writable: true, enumerable: true, configurable: true}
```

- 프로퍼티가 생성될 떄 [[Value]]의 값은 프로퍼티 값으로 초기화 됨
- [[Writable]], [[Enumerable]], [[Configurable]]의 값은 true로 초기화됨

### 접근자 프로퍼티

- 자체적으로 값을 갖지 않고 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 호출되는 `접근자 함수로 구성된 프로퍼티`

| 프로퍼티 어트리뷰트       | 프로퍼티 디스크립터 객체의 프로퍼티 | 설명                                                                                      |
|------------------|---------------------|-----------------------------------------------------------------------------------------|
| [[Get]]          | get                 | - 데이터 프로퍼티의 `값을 읽을 때` 호출<br/>- 프로퍼티 키로 값에 접근하면 프로퍼티 어트리뷰트 [[Get]]의 값 혹은 `getter` 함수 호출  |
| [[Set]]          | set                 | - 데이터 프로퍼티의 `값을 저장할 때` 호출<br/>- 프로퍼티 키로 값에 저장하면 프로퍼티 어트리뷰트 [[Set]]의 값 혹은 `setter` 함수 호출 |
| [[Enumerable]]   | enumerable          | - 데이터 프로퍼티와 동일                                                                          |
| [[Configurable]] | configurable        | - 데이터 프로퍼티와 동일                                                                          |
- 접근자 프로퍼티는 자체적으로 값을 가지지 않으며 읽거나 저장할때만 관여함

### 접근자 프로퍼티와 데이터 프로퍼티의 구별

```javascript
Object.getOwnPropertyDescriptor(Object.prototype, '__proto__');
// {get: f, set: f, enumerable: false, configurable: true}

Object.getOwnPropertyDescriptor(function() {}, 'prototype');
// {value: {...}, writable: true, enumerable: false, configurable: false}
```

- 일반 객체의 `__proto__`는 접근자 프로퍼티
- 함수 객체의 `prototype`은 데이터 프로퍼티

## 프로퍼티 정의

- 새로운 프로퍼티 추가, 어트리뷰터 정의, 재정의등을 말함
  - 프로퍼티의 값을 갱신하거나 열거하거나 재정의 가능하도록 할 것인지에 대한 정의 가능

### `Object.defineProperty`

- 프로퍼티의 어트리뷰트 `정의`
  - 객체의 참조, 문자열, 디스크립터 객체를 인자로 전달

| 프로퍼티 디스크립터 객체의 프로퍼티 | 대응하는 프로퍼티 어트리뷰트  | 생략했을 때의 기본값 |
|---------------------|------------------|-------------|
| value               | [[Value]]        | undefined   |
| get                 | [[Get]]          | undefined   |
| set                 | [[Set]]          | undefined   |
| writable            | [[Writable]]     | false       |
| enumerable          | [[Enumerable]]   | false       |
| configurable        | [[Configurable]] | false       |
- 한번에 하나의 프로퍼티만 정의 가능

```javascript
const person = {}

// 데이터 프로퍼티 정의
Object.defineProperty(person, 'firstName', {
	value: 'Ungmo',
	writable: true,
	enumerable: true,
	configurable: true
});

Object.defineProperty(person, 'lastName', {
	value: 'Lee',
});

// 접근자 프로퍼티 정의
Object.defineProperty(person, 'fullName', {
	// getter 함수
	get() {
		return `${this.firstName} ${this.lastName}`;
	},
  // setter 함수
	set(name) {
		[this.firstName, this.lastname] = name.split(' ');
	},
	enumerable: true,
	configurable: true
});
```

- 여러개의 프로퍼티를 정의하려면 `Object.defineProperties` 사용

## 객체 변경 방지

- 객체는 변경 가능한 값이므로 재할당 없이 직접 변경 가능
  - 프로퍼티 `추가, 삭제, 갱신`
  - Object.defineProperty, Object.defineProperties 메서드로 `재정의 가능`

| 구분       | 메서드                      | 프로퍼티 추가 | 프로퍼티 삭제 | 프로퍼티 값 읽기 | 프로퍼티 값 쓰기 | 프로퍼티 어트리뷰트 재정의 |
|----------|--------------------------|---------|---------|-----------|-----------|----------------|
| 객체 확장 금지 | Object.preventExtensions | X       | O       | O         | O         | O              |
| 객체 밀봉    | Object.seal              | X       | X       | O         | O         | X              |
| 객체 동결    | Object.freeze            | X       | X       | O         | X         | X              |

### `Object.preventExtensions` (객체 확장 금지)

- 확장이 금지된 객체는 프로퍼티 `추가가 금지`됨
- 프로퍼티 추가는 금지되지만 `삭제는 가능`
- 확장이 가능한 객체 여부 확인 메서드
  - `Object.isExtensible(object)` → Boolean

### `Object.seal` (객체 밀봉)

- 프로퍼티 `추가` 및 `삭제`, 어트리뷰트 `재정의` 금지
- 밀봉된 객체는 `읽기`와 `쓰기`만 가능
  - configurable → false
- 밀봉 객체 여부 확인 메서드
  - `Object.isExtensible(object)` → Boolean

### `Object.freeze` (객체 동결)

- 프로퍼티 `추가` 및 `삭제`, 어트리뷰트 `재정의` 금지, 프로퍼티 값 `갱신` 금지
- 동결된 객체는 `읽기`만 가능
- 밀봉 객체 여부 확인 메서드
  - `Object.isFrozen(object)` → Boolean

> 위의 변경 방지 메서드는 `얕은 변경 방지`로 직속 프로퍼티만 제한을 줄 수 있으며, `중첩 객체는 영향을 주지 못함`

> **Referenced**

- 이응모, 『모던 자바스크립트 Deep Dive』, 위키북스(2022.4.25), 220 ~ 233p