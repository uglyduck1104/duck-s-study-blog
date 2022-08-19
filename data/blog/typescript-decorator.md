---
title: 데코레이터
date: '2022-08-19'
tags: ['typescript']
draft: false
summary: 데코레이터 팩토리 작업 및 다양한 데코레이터 예시
layout: PostSimple
authors: ['default']
---

# 데코레이터

## 클래스 데코레이터 예시

### `tsconfig.json`

```json
{
  "compilerOptions": {
    //...
    "experimentalDecorator": true
  }
}
```

- `tsconfig.json`에서 `experimentalDecorator` 값을 `true`로 변경

### `app.ts`

```tsx
function Logger(constructor: Function) {
  console.log('Logging...')
  console.log(contructor)
}

@Logger
class Person {
  name = 'Max'

  constructor() {
    console.log('Creating person object...')
  }
}

const pers = new Person()

console.log(pers)
```

- 클래스 앞에 `@`을 붙여 데코레이터를 더하고 작성한 `Logger` 함수를 정의
  - `@`: 코딩에서 읽히거나 찾게 되는 특별한 식별자 상징
- 데코레이터에서는 인수(`target`)를 받으므로 `Person`의 `contructor` 함수를 전달받음
  - 데코레이터는 실체화되기 전 클래스가 정의만 돼도 실행됨

## 데코레이터 팩토리 작업

```tsx
function Logger(logString: string) {
  return function (constructor: Function) {
    console.log(logString)
    console.log(contructor)
  }
}

@Logger('LOGGING - PERSON')
class Person {
  name = 'Max'

  constructor() {
    console.log('Creating person object...')
  }
}

const pers = new Person()

console.log(pers)
```

- 데코레이터를 위의 예시처럼, 직접 만드는 대신 `팩토리`를 정의할 수 있음
  - 어떤 대상에 데코레이터를 할당할 때 설정할 수 있도록 함
- `return`을 `Logger` 함수 내부에 입력하고 새 익명 함수를 반환
  - `Logger` 함수가 실행되면 바깥 코드가 실행되고, 유효한 데코레이터 함수이자 내부 코드인 반환 값이 따라 붙음
- 문자열 `logString`을 통해 함수가 실행될 때, 특정 문자열을 인수로 전달받아 출력할 수 있음
  - 팩토리 함수와 함께 실행되면 데코레이션 함수가 사용하는 값을 커스터마이징할 수 있음

## 데코레이터 팩토리 심화

```tsx
function WithTemplate(template: string, hookId: string) {
  return function (constructor: any) {
    const hookEl = document.getElementById(hookId)
    const p = new contructor()
    if (hookEl) {
      hookEl.innerHTML = template
      hookEl.querySelector('h1')!.textContent = p.name
    }
  }
}

// @Logger('LOGGING - PERSON')
@WithTemplate('<h1>My Person Object</h1>', 'app')
class Person {
  name = 'Max'

  constructor() {
    console.log('Creating person object...')
  }
}

const pers = new Person()

console.log(pers)
```

- 팩토리 함수와 팩토리 데코레이터를 통해 템플릿과 함께 새 데코레이터 팩토리를 만들 수 있음
- 템플릿을 문자열로 하고, 훅 아이디가 문자열인 `WithTemplate` 함수를 정의하여 내부 함수에 클래스 생성자 함수를 로드하여 템플릿을 렌더할 수 있음
- 데코레이터 함수 생성하면 생성된 데코레이터 함수를 언제든지 불러와서 클래스에 추가할 수 있음
- 타입스크립트를 사용하는 앵귤러 프레임워크도 이와 같이 동작하는데, 데코레이터를 마치 컴포넌트 데코레이터처럼 사용하고, 해당 컴포넌트 템플릿 등을 명시하는 오브젝트를 통과하게 함

## 다양한 데코레이터 추가

```tsx
function Logger(logString: string) {
	console.log('LOGGER FACTORY');
	return function(constructor: Function){
		console.log(logString);
		console.log(contructor);
	};
}

function WithTemplate(template: string, hookId: string){
	console.log('TEMPLATE FACTORY');
	return function(constructor: any){
		console.log('Rendering template');
		const hookEl = document.getElementById(hookId);
		const p = new contructor();
		if (hookEl){
			hookEl.innerHTML = template;
			hookEl.querySelector('h1')!.textContent = p.name;
		}
	}
}

// @Logger('LOGGING - PERSON')
@Logger('LOGGING')
@WithTemplate('<h1>My Person Object</h1>', 'app')
class Person {
	name = 'Max';

	constructor() {
		console.log('Creating person object...');
	}
}

const pers = new Person();

console.log(pers);
```

- 데코레이터는 데코레이터를 사용할 수 있는 어떤 곳 혹은 클래스에 하나보다 많은 데코레이터를 사용할 수 있음
- 하나 이상의 데코레이터를 추가하면 데코레이터 함수는 `bottom-up`로 실행됨

## 속성 데코레이터

```tsx
function Log(target: any, propertyName: string | Symbol) {
  console.log('Property decorator!')
  console.log(target, propertyName)
}

class Product {
  @Log
  title: string
  private _price: number

  set price(val: number) {
    if (val > 0) {
      this._price = val
    } else {
      throw new Error('Invalid price - should be positive!')
    }
  }

  constructor(t: string, p: number) {
    this.title = t
    this._price = p
  }

  getPriceWithTax(tax: number) {
    return this.price * (1 + tax)
  }
}
```

- 클래스 이외에도 다른 곳에 데코레이터를 사용할 수 있는데, `Product` 클래스 내부에 `Log` 데코레이터를 사용해 두 프로퍼티, `constructor` 생성자 함수, 메소드를 Log 함수를 통해 취득할 수 있음
- 첫 인수는 프로퍼티의 `타겟`을, 두 번째 인수는 프로퍼티의 `이름`을 받는 Log 데코레이터 함수를 정의
  - 타겟은 객체가 어떤 구조를 가질 지 알 수 없으므로 `any` 타입으로 설정
  - 프로퍼티 이름은 `문자열` 혹은 `심볼` 타입로 설정
- 객체의 프로토타입과 프로퍼티의 이름, 메소드의 정보를 취득할 수 있음

## 접근자 & 매개변수 데코레이터

```tsx
function Log(target: any, propertyName: string | Symbol) {
	console.log('Property decorator!');
	console.log(target, propertyName);
}

function Log2(
	target: any,
	name: string,
	descriptor: PropertyDescriptor
){
	console.log('Accessor decorator!');
	console.log(target);
	console.log(name);
	console.log(descriptor);
})

function Log3(
	target: any,
	name: string | Symbol,
	descriptor: PropertyDescriptor
){
	console.log('Method decorator!');
	console.log(target);
	console.log(name);
	console.log(descriptor);
})

function Log4(
	target: any,
	name: string | Symbol,
	position: number
){
	console.log('Parameter decorator!');
	console.log(target);
	console.log(name);
	console.log(position);
})

class Product {
	@Log
	title: string;
	private _price: number;

	@Log2
	set price(val: number){
		if(val > 0) {
			this._price = val;
		} else {
			throw new Error('Invalid price - should be positive!');
		}
	}

	constructor(t: string, p: number){
		this.title = t;
		this._price = p;
	}

	@Log3
	getPriceWithTax(@Log4 tax: number){
		return this.price * (1 + tax);
	}
}
```

- 프로퍼티 외에 접근자에 데코레이터를 더할 수 있음

### Log2 - 접근자 데코레이터

- `target`
  - 대상이 되는 프로토 타입의 정보
- `name`
  - 외부 액세서의 이름(`price`)을 출력
- `descriptor`
  - 프로토 타입의 프로퍼티 디스크립터
    - 열거되지 않지만 변경할 수 있음
    - get 함수를 제외한 `set` 함수 정보 취득

### Log3 - 메서드 데코레이터

- `target`
  - 대상이 되는 프로토 타입의 정보
- `name`
  - 메서드의 이름(`getPriceWithTax`)을 출력
- `descriptor`
  - 프로토 타입의 프로퍼티 디스크립터
    - 열거되지 않지만 변경할 수 있음
    - `value`, `writable` 속성 취득

### Log4 - 매개변수 데코레이터

- `target`
  - 대상이 되는 프로토 타입의 정보
- `name`
  - 매개변수를 사용하는 메서드의 이름(`getPriceWithTax`)을 출력
- `position`
  - 0부터 시작하는 인수의 수(`index`)

## 클래스 데코레이터에서 클래스 반환

```tsx
function WithTemplate(template: string, hookId: string){
	return function<T extends { new(...args: any[]): {name: string} }>(
		originalContstructor: T
	){
		return class extends originalContstructor {
			constructor(..._: any[]) {
				super();
				const hookEl = document.getElementById(hookId);
				if (hookEl){
					hookEl.innerHTML = template;
					hookEl.querySelector('h1')!.textContent = this.name;
				}
			}
		}
	}
}

// @Logger('LOGGING - PERSON')
@Logger('LOGGING')
@WithTemplate('<h1>My Person Object</h1>', 'app')
class Person {
	name = 'Max';

	constructor() {
		console.log('Creating person object...');
	}
}

const pers = new Person();

console.log(pers);
```

- 데코레이터 함수는 내부의 함수에서 값을 반환할 수 있음
  - 이를 이용해 클래스의 새로운 컨스트럭터 함수를 반환할 수 있는데 기존의 생성자 함수를 대체할 수 있음
  - 즉, 새 함수, 컨스트럭터 함수를 반환하거나 새 클래스를 반환할 수 있음
- `super()`
  - 오리지널 함수와 클래스를 저장
- `return class extends originalContstructor`
  - 오리지널 클래스를 확장해서 오리지널 클래스에 있던 모든 것을 저장
  - 새 커스텀 클래스로 기존의 클래스를 대체하여 추가 로직을 실행

## Autobind 데코레이터 만들기

```tsx
function Autobind(_: any, _2: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value
  const adjDescriptor: PropertyDescriptor = {
    configurable: true,
    enumerable: false,
    get() {
      const boundFn = originalMethod.bind(this)
      return boundFn
    },
  }
  return adjDescriptor
}

class Printer {
  message = 'This works!'

  @Autobind
  showMessage() {
    console.log(this.message)
  }
}

const p = new Printer()

const button = document.querySelector('button')!
button.addEventListener('click', p.showMessage)
```

- `button` 요소에 `showMessage` 메서드를 실행하기 위해서는 클래스내에 바인딩되어 있는 `this` 키워드의 컨텍스트나 레퍼런스가 호출되었을 때와 동일하지 않으므로 `bind` 메서드를 바인딩해야 함
- `this` 키워드를 메서드가 속해 있는 객체로 설정
- `PropertyDescriptor`의 속성을 재정의

## 데코레이터 유효성 검증

- `validator`가 타입스크립트 데코레이터를 이용하여 작동하는 방식

```tsx
interface ValidatorConfig {
	[property: string]: {
		[validatableProp: string] : string[] // ['required', 'positive']
	}
}

const registeredValidators: ValidatorConfig = {};

function Required(target: any, propName: string) {
	registredValidators[target.contructor.name] = {
		...registredValidators[target.contructor.name],
		[propName]: ['required']
	}
}

function PositiveNumber(target: any, propName: string) {
	registredValidators[target.contructor.name] = {
		...registredValidators[target.contructor.name],
		[propName]: ['positive']
	}
}

function validate(obj: any) {
	const objValidatorsConfig = registeredValidators[obj.constructor.name];
	if(!objValidatorConig){
		return true;
	}
	let isValid = true;
	for (const prop in objValidatorConfig){
		for (const validator of objValidatorConfig[prop]){
			switch (validator){
				case 'required':
					isValid = isValid && !!obj[prop];
					break;
				case 'positive':
					isValid = isValid && !!obj[prop] > 0;
					break;
			}
		}
	}
	return isValid;
}

class Course {
	@Required
	title: string;
	@PositiveNumber
	price: number;

	constructor(t: string, p: number){
		this.title = t;
		this.price = p;
	}
}

const courseForm = document.querySelector('form');
courseForm.addEventListner('submit'. event => {
	event.preventDefault();
	const titleEl = document.getElementById('title') as HTMLInputElement;
	const priceEl = document.getElementById('price') as HTMLInputElement;

	const title = titleEl.value;
	const price = +priceEl.value;

	const createdCourse = new Course(title, price);

	if(!validate(createdCourse)){
		alert('Invalid inputs, please try again!');
		return;
	}

}
```

- 데코레이터의 `유효성 검증`하는 예시를 확인할 수 있음
- 각각의 데코레이션 함수를 정의하고 `Course` 클래스 내에 데코레이션을(`@Required`, `@PoritiveNumber`) 사용

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
