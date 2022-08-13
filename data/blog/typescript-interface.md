---
title: 인터페이스
date: '2022-08-13'
tags: ['typescript']
draft: false
summary: 타입스크립트에서의 인터페이스는 객체의 구조나 형태를 설명하는데 사용
layout: PostSimple
authors: ['default']
---

# 인터페이스

- 타입스크립트에서의 인터페이스는 객체의 `구조`나 `형태`를 설명하는데 사용

## 첫 번째 인터페이스

```tsx
interface Person {
	name: string;
	age: number;

	greet(phrase: string): void;
}

let user1: Person;

user1 = {
	name: 'Max',
	age: 30,
	greet(phrase: string): {
		console.log(phrase + ' ' + this.name);
	}
}

user1.greet('Hi there - I am');
```

- 인수의 `이름`과 `타입`, 메서드의 `반환 타입` 순서로 정의
- `interface` 키워드로 인터페이스를 생성
  - 타입스크립트에만 존재 즉, 컴파일된 자바스크립트 파일에서는 해당 키워드를 볼 수 없음
  - 클래스와 마찬가지로 `첫 글자는 대문자`를 사용해야 함(관례)
- 클래스와 달리 객체의 청사진으로써 사용되지 않고 `사용자 정의 타입`으로만 사용
  - 속성의 이름과 속성에 저장될 값의 타입을 가진 속성이나 필드의 정의를 추가
  - 구체적인 값이 아닌 `구조`만 있으므로 `기본값을 할당할 수 없음`
- 객체의 구조와 인터페이스의 구조 차이점
  - `객체`의 구조
    - `쉼표(,)`로 구분
  - `인터페이스`의 구조
    - `세미콜론(;)`으로 구분

## 클래스와 인터페이스 사용하기

```tsx
// type Greetable = {
interface Greetable {
  name: string

  greet(phrase: string): void
}

class Person implements Greetable {
  name: string
  age = 30

  constructor(n: string) {
    this.name = n
  }

  greet() {
    console.log(phrase + ' ' + this.name)
  }
}

let user1: Greetable

user1 = new Person('Max')

user1.greet('Hi there - I am')
```

- 타입과 인터페이스는 정의하는데, 비슷해보이나 가장 큰 차이점으로 `인터페이스`는 `객체의 구조를 설명`하기 위해서만 사용됨
  - 주로 구체적인 구현이 아닌 서로 다른 클래스 간의 `기능을 공유`하기 위해 사용
- `인터페이스`를 자주 사용하는 이유는 `클래스`가 인터페이스를 이행하고 준수해야 하는 약속처럼 사용되기때문임
  - 인터페이스를 사용하면 여러 클래스 간에 공유가 가능함
  - 인터페이스로 구조를 정의하고 클래스가 `Greetable` 인터페이스를 준수해야 한다고 타입스크립트에 알려줘야 함
- `implements` 키워드를 사용해 인터페이스의 이름을 추가
  - `상속`의 개념과 달리 `여러개`의 인터페이스를 추가할 수 있음
  - `필수`로 준수해야하는 필드의 속성 이외에 추가하고 싶은 속성을 확장할 수 있음

## readonly 인터페이스 속성

```tsx
interface Greetable {
  readonly name: string

  greet(phrase: string): void
}
```

- 인터페이스 내에 모든 객체의 속성이 한 번만 설정되어야 하는 `readonly` 제어자도 추가할 수 있음
  - `public`, `private` 접근 제어자는 지정 불가능
  - 객체가 초기화되면 변경할 수 없도록 지정 가능
- 읽기 전용으로 지정한 속성은 타입스크립트가 자동으로 추론함

## 인터페이스 확장하기

```tsx
interface Named {
  readonly name: string
}

interface Greetable extends Named {
  greet(phrase: string): void
}

class Person implements Greetable, Named {
  name: string
  age = 30

  constructor(n: string) {
    this.name = n
  }

  greet() {
    console.log(phrase + ' ' + this.name)
  }
}
```

- 여러 인터페이스를 하나의 인터페이스로 병합할 수 도 있고 둘 이상을 확장할 수 있음
  - `Person` Class에 `Greetable`, `Named`를 구현하여 `greet` 메소드와 `name`을 모두 갖도록 할 수 있음
  - 확장하고자 하는 인터페이스를 추가하려면 `쉼표(,)`를 사용해 구분할 수 있음(`다중 상속`)
- `Named` 인터페이스를 확장하여 `Greetable`을 기반으로 하는 모든 객체가 `greet` 메소드와 함께 `name`도 가질 수 있도록 새로운 인터페이스를 형성할 수 있음

## 인터페이스 함수 타입 사용하기

```tsx
// type AddFn = (a: number, b: number) => number;
interface AddFn {
  (a: number, b: number): number
}

let add: AddFn

add = (n1: number, n2: number) => {
  return n1 + n2
}
```

- 함수의 구조를 정의하는 데 사용할 수 있음
  - `타입` 키워드를 사용하거나 `인터페이스` 함수 타입을 사용할 수 있음
- 숫자의 타입의 인수를 전달받아 숫자 타입으로 반환

## 옵셔널 매개변수 & 속성

```tsx
interface Named {
	readonly name?: string;
	outputName?: string;
}

const Person implements Greetable {
	name?: string;
	age = 30;

	constructor(n?: string){
		if (n) {
			this.name = n;
		}
	}

	greet(phrase: string) {
		if(this.name) {
			console.log(phrase + ' ' + this.name);
		} else {
			console.log('Hi');
		}
	}
}
```

- 특정 속성이 이 인터페이스를 구현하는 클래스내에 있을 수 있지만 반드시 그렇지 않은 경우 `옵셔널 매개변수(?)`를 사용할 수 있음
  - 옵셔널 매개변수 및 속성으로 선언한 값에 대해 if 조건절을 사용해 관계를 느슨하게 연결할 수 있음

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
