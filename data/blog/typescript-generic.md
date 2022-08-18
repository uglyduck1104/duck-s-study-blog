---
title: 제네릭
date: '2022-08-18'
tags: ['typescript']
draft: false
summary: 타입스크립트에 내장된 기본 타입이며 다른 타입과 연결 관계를 가지고 있음
layout: PostSimple
authors: ['default']
---

# 제네릭

## 내장 제네릭 & 제네릭이란?

- 타입스크립트에 내장된 기본 타입이며 다른 타입과 연결 관계를 가지고 있음

```tsx
const names: Array<string> = [] // string[]
// names[0].split(' ');

const promise: Promise<string> = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('This is done!')
  }, 2000)
})

promise.then((data) => {
  data.split(' ')
})
```

- 제네릭 타입은 다른 타입과 연결되는 종류인데 다른 타입이 `어떤 타입`이어야하는지에 대해서는 크게 상관하지 않음
  - 제네릭 타입을 정의하기 위해서는 홑화살괄호`<>` 내에 배열에 전달되어야 하는 데이터의 타입을 지정
- 자바스크립트에서 지원하는 `프로미스`를 타입스크립트에서는 하나의 타입으로 지정할 수 있음
  - `프로미스`가 어떤 타입의 데이터를 반환하는 지 다른 타입과 함께 연결할 수 있음
  - 즉, 프로미스가 반환하는 타입이 `어떤 타입인지에 대한 정보`를 타입스크립트에게 알려줌
- 제네릭 타입을 통해 반환하거나, 저장하고자 하는 데이터의 타입을 명시하여 `타입 안정성`을 확보할 수 있음

## 제네릭 함수 생성하기

```tsx
function merge<T, U>(objA: T, objB: U) {
  return Object.assing(objA, objB)
}

const mergedObj = merge({ name: 'Max' }, { age: 30 })
```

- merge 함수의 전달되는 매개 변수의 타입을 알 수 없을 때, 제네릭 함수를 이용해 두 매개변수가 서로 다른 타입이 될 수 있다고 타입스크립트에게 알려줄 수 있음
  - object로 타입을 정의하는 것보다 명확하게 `타입을 명시`할 수 있으며, 타입스크립트는 병합된 객체에 저장된 요소가 두 객체 타입의 `인터섹션`이라고 인식할 수 있음
  - 제네릭 타입은 `T`, `U` 처럼 알파벳 순서로 명시할 수 있음(`관례`)

## 제약 조건 작업

```tsx
function merge<T extends object, U extends object>(objA: T, objB: U) {
  return Object.assingn(objA, objB)
}

const mergedObj = merge({ name: 'Max' }, { age: 30 })
```

- merge 함수는 제네릭 함수이므로 정확한 타입이 무엇이어야 하는지 상관하지 않고, 매개 변수로 전달받은 객체의 정확한 구조도 상관없음
  - 다만 `Object.assign`에는 두 객체를 병합하는 기능을 하므로 객체(`object`)여야만 한다는 타입 제약 조건을 통해 특정 제약 조건을 설정해야 함
  - 제네릭 타입 뒤에 `extends` 키워드를 통해 제약 조건을 설정할 수 있음

## 다른 일반 함수

```tsx
interface Lengthy {
	length: number;
}

function countAndDescribe<T extends Lengthy>(element: T): [T, string]{
	let descriptionText = 'Got no value.';
	if (element.length === 1 {
		descriptionText = 'Got 1 element.'
	} else if (element.length > 1){
		descriptionText = 'Got ' + element.length + ' elements.';
	}
	return [element, descriptionText];
}

console.log(countAndDescribe(['Sports', 'Cooking']));
```

- 매개변수에 `interface`에서 정의한 `Lengthy`라는 제약 조건을 통해 `length`의 속성을 가지는 타입의 길이와 요소의 길이에 따라서 `descriptionText`를 반환(`튜플- 길이가 정해져 있는 배열`)하는 함수를 정의
- 타입스크립트는 문자열이나 T 타입 요소를 지니는 배열이 반환된다고 추론

## keyof 제약 조건

```tsx
function extractAndConvert<T extends object, U extends keyof T>(
	obj: T,
	key: U
){
	return 'Value: ' obj[key];
}

extractAndConvert({name: 'Max'} 'name');
```

- 두 개의 매개변수를 받는 함수가 첫 번째 매개변수의 값을 취득하기 위해, 두 번째 매개변수가 키의 역할을 하고자 할 때, `keyof` 제약 조건을 통해 객체 내 속성으로 접근할 수 있음
  - 함수를 호출할 때, 존재하지 않는 속성에 접근하는 실수를 방지할 수 있음

## 제네릭 클래스

```tsx
class DataStorage<T> {
  private data: T[] = []

  addItem(item: T) {
    this.data.push(item)
  }

  removeItem(item: T) {
    this.data.splice(this.data.indexOf(item), 1)
  }

  getItems() {
    return [...this.data]
  }
}

const textStorage = new DataStorage<string>()
textStorage.addItem('Max')
textStorage.addItem('Manu')
textStorage.removeItem('Max')
console.log(textStorage.getItems()) // ["Manu"]

const numberStorage = new DataStrotage<number>()

const objStorage = new DataStorage<object>()
objStorage.addItem({ name: 'Max' })
objStorage.addItem({ name: 'Manu' })
// ...
objStorage.removeItem({ name: 'Max' })
console.log(objStorage.getItems()) // '[{name: "Max"}];
```

- `DataStorage`가 인스턴스로 생성될때 전달되는 매개변수의 타입을 열어두고, 유연성을 확보할 수 있음
  - `문자열`이나 `숫자형` 또는 원하는 요소를 저장하여, 작업을 수행할 수 있음
- 하지만 `객체`나 `배열`(참조 타입)로 작업을 하는 경우 객체를 전달하면 indexof가 작동하지 않는 근본적인 문제가 발생함

  - 배열의 마지막 요소를 식별할 수 없으므로 항상 배열의 마지막 요소를 제거함

    ```tsx
    removeItem(item: T){
    	if(this.data.indexOf(item) === -1){
    		return;
    	}
    	this.data.splice(this.data.indexOf(item), 1);
    }
    ```

  - `indexOf` 메소드는 `item`(요소)을 찾지 못하면 `-1`을 반환하므로, -1과 같을 경우에 `return`하는 분기를 추가

```tsx
const objStorage = new DataStorage<object>()
const maxObj = { name: 'Max' }
objStorage.addItem(maxObj)
objStorage.addItem({ name: 'Manu' })
// ...
objStorage.removeItem(maxObj)
console.log(objStorage.getItems())
```

- `maxObj` 상수에 객체를 전달하여 작업을 수행하면 메모리에 있는 정확히 같은 객체를 사용할 수 있음
- object 타입은 아까와도 말했듯이, 원시타입보다 명확하지 않으므로 클래스 선언 시, 제한을 두는 것을 권장함

  ```tsx
  class DataStorage<T extends string | number | boolean> {
  	private data: T[] = [];

  	addItem(item: T){ ... }

  	removeItem(item: T){ ... }

  	getItems() { ... }
  }
  ```

  - 원시 타입으로 제약조건을 주고 인스턴스를 생성할 때, `object` 보다는 명시적인 `원시 값`을 지정해주는 것이 좋음

## 제네릭 유틸리티 타입

- 제네릭 타입을 사용하는 `내장 타입`, 특정 `유틸리티` 기능을 제공하는 제네릭 타입을 지원
  - 컴파일 단계에만 존재하며, 이 타입들이 추가적인 타입 `안전성`과 `유연성`을 제공해 줌

```tsx
interface CourseGoal {
  title: string
  description: string
  completeUntil: Date
}

function createCourseGoal(title: string, description: string, date: Date): CourseGoal {
  let courseGoal: Partial<CourseGoal> = {}
  courseGoal.title = title
  courseGoal.description = description
  completeUntil.completeUntil = date
  return courseGoal as CourseGoal
}
```

### Partial

- 만든 타입 전체를 모든 속성을 선택적인 객체 타입으로 바꿈
- `CourseGoal`을 타입이 아니므로, 리턴하는데 에러가 발생

### Readonly

```tsx
const names: Readonly<string[]> = ['Max', 'Sports']
// name.push('Manu');
// name.pop();
```

- 문자열의 배열을 저장하는데 이 문자열의 배열은 `읽기만 가능`하다고 알려줌
  - 할당한 변수의 원본 배열을 `변경할 수 없음`(`push`, `pop`)
- 배열 뿐만아니라 객체에 대해서도 Readonly를 사용할 수 있음

## 제네릭 타입 vs 유니언 타입

### 제네릭 타입

- 특정 타입을 고정하건, 생성한 전체 클래스 인스턴스에 걸쳐 같은 함수를 사용하거나, 전체 함수에 걸쳐 같은 타입을 사용할 때 유용함
- 한 타입으로 고정

### 유니언 타입

- 모든 메소드 호출이나 모든 함수 호출마다 다른 타입을 지정하고자 하는 경우 유용함

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
