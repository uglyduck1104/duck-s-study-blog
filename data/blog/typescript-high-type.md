---
title: 고급 타입
date: '2022-08-17'
tags: ['typescript']
draft: false
summary: 타입스크립트에서 다루는 고급 타입에 대한 소개
layout: PostSimple
authors: ['default']
---

# 고급 타입

## 인터섹션 타입

- 다른 타입을 결합할 때 주로 사용

### `app.ts`

```tsx
type Admin = {
	name: string;
	privileges: string[];
};

type Employee = {
	name: string;
	startDate: Date;
};

// interface ElevatedEmployee extends Employee, Admin {}

type ElevatedEmployee = Admin & Employee;

const e1: ElevatedEmployee = {
	name: 'Max',
	privileges: ['create-server'],
	startDate: new Date();
}
```

- `Admin` 타입과 `Employee` 타입을 결합하여 새로운 타입인 `EleveatedEmployee` 타입을 생성
- `e1` 변수를 선언하고 `ElevatedEmployee` 타입을 지정했으므로, 두 타입에서 지정한 `startDate`와 `privileges` 속성을 지녀야 함
- 인터섹션 타입은 상속과 밀접한 관련이 있음
  - 인터페이스로 선언하면 상속을 통해 타입과 동일하게 정의가 가능하지만, 코드가 길어지므로 인터섹션 타입으로도 대체할 수 있음

```tsx
type Combinable = string | number
type Numeric = number | boolean

type Universal = Combinable & Numeric
```

- 두 유니언 타입이 교차하는 `유니언 타입`이 되므로 타입스크립트는 `Universal`을 숫자형 타입으로 간주함
- 인터섹션 연산자는 어떤 타입과도 사용할 수 있으므로 타입 교차를 간단하게 구현할 수 있음

## 타입 가드에 대한 추가 정보

- 특정 속성이나 메소드를 사용하기 전에 존재하는지의 여부와 어떤 작업을 수행할 수 있는지를 확인하는 개념 또는 방식
  - 유니언 타입으로 문자열이나 숫자형을 지니는데 타입가드는 유니언 타입을 돕는 기능을 함
  - 유연성을 가지는 것은 장점으로 보일 수 있으나, 런타임 시 어떤 타입을 얻게 될지 알아야 함

### 예시 1) 유니언 타입

```tsx
type Combinable = string | number
type Numeric = number | boolean

type Universal = Combinable & Numeric

function add(a: Combinable, b: Combinable) {
  if (typeof a === 'string' || typeof b === 'string') {
    return a.toString() + b.toString()
  }
  return a + b
}
```

- `typeof a`가 `string`과 같은지 확인하거나 또는 `typeof b`가 `string`과 같은지 확인하여 같은 경우에는 `a.toString() + b.toString()`를 반환하도록 함
- 문자열 타입이 아닌 경우 연산을 통해 `a`와 `b`가 숫자형임을 알 수 있음
- 이와 같이 타입가드를 통해 유니온 타입이 지닌 유연성을 활용할 수 있으며, 런타임 시 코드가 정확하게 작동하게 해줌

### 예시 2) 객체 유니언 타입

```tsx
type UnknownEmployee = Employee | Admin

function printEmployeeInformation(emp: UnknownEmployee) {
  if ('privileges' in emp) {
    console.log('Privileges: ' + emp.privileges)
  }
  if ('startDate' in emp) {
    console.log('Start Date: ' + emp.startDate)
  }
}

printEmployeeInformation({ name: 'Manu', startDate: new Date() })
```

객체 속성은 항상 `object`이므로 타입 가드 시 유의해서 작성해야 함

- 자바스크립트의 `in` 키워드를 이용하여 해당 필드가 객체에 존재하는지 여부로 해결할 수 있음

### 예시 3) 클래스 유니언 타입

```tsx
class Car {
  drive() {
    console.log('Driving...')
  }
}

class Truck {
  drive() {
    console.log('Driving a truck...')
  }

  loadCargo(amount: number) {
    console.log('Loading cargo ...' + amount)
  }
}

type Vehicle = Car | Truck

const v1 = new Car()
const v2 = new Truct()

function useVehicle(vehicle: Vehicle) {
  vehicle.drive()
  if (vehicle instanceof Truck) {
    vehicle.loadCargo(1000)
  }
}

useVehicle(v1)
useVehicle(v2)
```

- 위의 예시와 같이 유니언 타입인 `Vehicle` 타입을 설정
- 바닐라 자바스크립트에 내장된 일반 연산자인 `instanceof`를 사용하여 `vehicle`이 `Truck`의 인스턴스인지 확인
  - 타입스크립트는 `Truck` 생성자 함수를 기반으로 `vehicle`이 생성되는지 확인할 수 있음

## 구별된 유니언

- 특수한 typeof 타입가드나 타입가드르 도와주는 유니언을 나타냄
  - 타입가드를 쉽게 구현할 수 있게 해주는 유니언 타입 패턴

```tsx
interface Bird {
  type: 'bird'
  flyingSpeed: number
}

interface Horse {
  type: 'horse'
  runningSpeed: number
}

type Animal = Bird & Horse

function moveAnimal(animal: Animal) {
  let speed
  switch (animal.type) {
    case 'bird':
      return animal.flyingSpeed
      break
    case 'horse':
      return animal.runnigSpeed
      break
  }
  console.log('Moving at speed: ' + speed)
}

moveAnimal({ type: 'bird', flyingSpeed: 10 })
```

- 인터페이스가 자바스크립트로 컴파일되지 않았기 때문에 `instanceof`를 사용할 수 없음
- 리터럴 타입 배정을 통해 해당 인터페이스가 `bird`의 타입이거나 `horse`의 타입임을 명시
  - `switch` 문을 통해 `animal.type`에 따라 `speed`를 다르게 반환
- 이처럼 유니언을 구성하는 모든 객체에는 하나의 `공통 속성`만 있고, 구별되는 키값을 `switch`문을 통해 `완전한 타입 안전성`을 갖추면서 객체에 어떤 속성을 사용할수 있는지 파악할 수 있음
  - 오타의 위험을 줄일 수 있음

## 형 변환

- 타입스크립트가 직접 감지하지 못하는 특정 타입의 값을 타입스크립트에 알려주는 역할

```tsx
// <input type="text" id="message-output" />

const userInputElement = document.getElementById('message-output')! // HTMLElement | null
userInputElement.value = 'Hi there!'
```

- 타입스크립트는 `HTML` 파일을 살펴보고 분석하지 못하므로, `null`이 아닌 `HTMLInputElement` 타입임을 타입스크립트에게 알려야 함

### 형 변환 방법

1. 변환하고자 하는 요소 앞이나 타입스크립트에 타입을 알려주고자 하는 위치 앞에 홑화살괄호 쌍(`<>`)추가하는 방법

   ```tsx
   const userInputElement = <HTMLInputElement>document.getElementById('message-output')!
   ```

2. `as` 키워드 사용

   ```tsx
   const userInputElement = document.getElementById('message-output')! as HTMLInputElement
   ```

- 리액트 프로젝트에서는 비슷한 구문이 있으므로, JSX와의 충돌을 방지하기 위한 대안을 구색
- 형변환은 두 가지 방식중에 하나의 방식으로 통일해서 작성하는 것을 권장함

### null 반환 예외 처리

```tsx
const userInputElement = document.getElementById('message-output')

if (userInputElement) {
  ;(userInputElement as HTMLInputElement).value = 'Hi there!'
}
```

- `null`을 반환할 수 있는 `dom`이 `null`을 반환하지 않을 것이라는 확신이 들지 않는 경우 `!`를 사용하지 않고 `if`문을 사용해 처리할 수 있음
- 표현식을 괄호 쌍으로 감싸서 괄호안을 먼저 평가되도록 한 후 `value`값을 취득

## 인덱스 속성

- 객체가 지닐 수 있는 속성에 대해 보다 유연한 객체를 생성할 수 있는 기능

```tsx
interface ErrorContainer {
  // { email: 'Not a valid email', username: 'Must start with a capital character!' }
  [prop: string]: string
}

const errorBag: ErrorContainer = {
  email: 'Not a valid email',
  username: 'Must start with a capital character!',
}
```

- 정확한 속성 이름을 모르고 속성의 개수도 모르며, 에러 컨테이너를 기반으로 이 객체에 추가되는 모든 속성은 문자열로 해석할 수 있는 속성 이름을 지녀야 한다는 것과 해당 속성에 대한 값 역시 문자열이어야 한다는 것을 알 수 있음

## 함수 오버로드

- 동일한 함수에 대해 여러 함수의 시그니처를 정의할 수 있는 기능
  - 다양한 매개변수를 지닌 함수를 호출하는 다양한 방법을 사용하여 함수 내에서 작업을 수행할 수 있게 함

```tsx
type Combinable = string | number
type Numeric = number | boolean

type Universal = Combinable & Numeric

function add(a: number, b: number): number
function add(a: string, b: string): string
function add(a: string, b: number): string
function add(a: number, b: number): string
function add(a: Combinable, b: Combinable) {
  if (typeof a === 'string' || typeof b === 'string') {
    return a.toString() + b.toString()
  }
  return a + b
}

const result = add(1, 5)
```

- 함수 오버로드는 자바스크립트에서 작동되는 특정 텍스트가 아니므로 컴파일 과정에서 타입스크립트가 제거함
  - 함수 정보와 함수 선언을 하나로 병합하여 오버로드된 함수들의 정보를 결합함

## Null 병합

- 변수에 할당되는 값이 빈 문자열이나 0이 아닌, `null`이나 `undefined`라면 null 병합 연산자 사용하여 유연하게 처리가 가능함

```tsx
const userInput = undefined

const storedData = userInput ?? 'DEFAULT'

console.log(storedDate) // DEFAULT
```

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
