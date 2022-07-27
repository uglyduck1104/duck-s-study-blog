---
title: TypeScript 기본 & 기본 타입
date: '2022-07-27'
tags: ['typescript']
draft: false
summary: 타입스크립트는 자바스크립트의 슈퍼셋으로서, 코드를 실행하여 타입스크립트 코드를 자바스크립트로 컴파일하는 컴파일러
layout: PostSimple
authors: ['default']
---

# TypeScript 사용하는 이유

## 타입 스크립트란?

- 타입스크립트는 자바스크립트의 슈퍼셋으로서, 코드를 실행하여 타입스크립트 코드를 자바스크립트로 컴파일하는 `컴파일러`
  - 단독으로 실행할 수 없으며, 컴파일된 자바스크립트를 output 함
- 자바스크립트를 기반으로 새로운 기능과 장점을 추가하는 언어
  - 나은 구문과 작업을 보다 손쉽게 수행할 수 있게 해주는 방법 제공
  - 코드의 에러를 `컴파일 단계`에서 캐치할 수 있음
- 브라우저와 같은 자바스크립트 환경에서 실행할 수 없음

# TypeScript 기본 & 기본 타입

## 타입 이해하기

### 타입 사용하기

| number  | 1, 5.3, -10        | 모든 숫자, 정수와 실수에는 차이가 없음 |
| ------- |--------------------| -------------------------------------- |
| string  | 'Hi’, “Hi”, \`HI\` | 간단한 텍스트                          |
| boolean | true, false        | 참 또는 거짓이라는 두 값               |

### 문제 인식

`Javascript`

```jsx
function add(n1, n2) {
  return n1 + n2
}

const number1 = '5'
const number2 = 2.8

const result = add(number1, number2)
console.log(result) // 52.8
```

- Javascript는 수학적 방식으로 처리하는 게 아닌 문자열로 처리함
  - 값이 할당되기 전에는 무슨 데이터 타입인지 알지 못함
  - 할당 된 후에도 우리가 인지하는 수학적 계산에 의한 결과 값을 얻지 못함
- 복잡도가 큰 로직에서 에러를 찾아내기 어려울 수 있음

`Typescript`

```tsx
function add(n1: number, n2: number) {
  return n1 + n2
}

const number1 = '5'
const number2 = 2.8

const result = add(number1, number2)
console.log(result) // 52.8
```

```bash
app.ts:8:20 - error TS2345: Argument of type '"5"' is not assignable to parameter of type 'number'.
# 문자열 5라는 유형의 인수는 '숫자형' 타입의 매개변수에 할당할 수 없습니다.

8 const result = add(number1, number2);
```

- 함수에 있는 타입 배정을 매개변수에 추가하여, `타입 추론`을 할 수 있게 수정
- 두 매개변수가 `숫자형 타입`이어야 하며, 어느 것이 기본값이어야 하는지 상관없다고 입력할 수 있음
- 전달하고자 하는 함수의 매개 변수 `타입`이 맞지 않을 경우, 에러를 발생함
- 컴파일 하는 동안에만 에러가 고정되어 나타남

  > 브라우저에는 `내장 타입스크립트 지원이 없으므로`, 런타임에서 자바스크립트가 다른 식으로 작동하지 않도록 변경하지 않음

### TypeScript 타입 vs Javascript 타입

`Javascript`(typeof 연산자 사용)

```jsx
function add(n1, n2) {
  if (typeof n1 !== 'number' || typeof n2 !== 'number') {
    throw new Error('Incorrect input!!!')
  }
  return n1 + n2
}
const number1 = '5'
const number2 = 2.8

const result = add(number1, number2)
console.log(result)
```

- typeof 연산자를 사용하여 타입이 `number`와 같은 지 런타임 검사를 실행할 수 있음
- 자바스크립트는 숫자형, 문자열, 불리언과 같은 타입들을 알고 있지만 런타임 단계전에 알 수 없으므로 타입스크립트에 비해 먼저 버그를 잡아낼 수 없음

`Typescript`

```tsx
function add(n1: number, n2: number) {
  return n1 + n2
}

const number1 = 5
const number2 = 2.8

const result = add(number1, number2)
console.log(result)
```

- 타입스크립트는 자바스크립트에 비해 `훨씬 많은 타입을 알고 있으므로`, 자바스크립트의 런타임 검사보다 더 유연하고 강력하게 `타입 검사`를 할 수 있음
- 하지만, 자바스크립트 엔진에 내장되어 있지 않으므로 타입스크립트로 `개발하는 도중에만` 타입 추론 기능을 사용할 수 있음

> 타입스크립트의 주요 **원시 타입은 모두 소문자**임

### 숫자 문자열 및 불리언 작업하기

```tsx
function add(n1: number, n2: number, showResult: boolean, phrase: string) {
  const result = n1 + n2
  if (showResult) {
    console.log(phrase + result)
  } else {
    return n1 + n2
  }
}

const number1 = 5
const number2 = 2.8
const printResult = true
const resultPhrase = 'Result is '

const result = add(number1, number2, printResult, resultPhrase)
console.log(result)
```

- 자바스크립트와 타입스크립트는 정수와 실수를 구분하지 않음
- `console.log(phrase + result);`
  - n1, n2 매개변수 값은 `문자열과 같이 연산`되는 경우에 자바스크립트에서는 `문자열로 인식`하므로, if 조건절을 만나기 전에 `수학적 계산을 변수에 할당하여 값을 저장`하고 문자열과 연산해주는 것이 안전함

### 타입 할당 및 타입 추론하기

- 특정 변수나 상수에 어떤 타입을 사용했는지 타입스크립는 `타입 추론`(type interface)를 통해 내장 기능을 활용함
- 변수를 선언하여 값을 할당할 때, 지정한 타입값을 직접 지정하는 방식보다 어떤 값을 저장할 지 먼저 타입스크립트에 알려주는 편이 좋음

  ```tsx
  let number1: number
  number1 = 5
  const number2 = 2.8
  const printResult = true
  let resultPhrase = 'Result is '
  ```

- 타입을 잘못 사용하고 있는지 확인하고 에러를 통해 알려주는 핵심 작업을 수행함

## 핵심 타입 & 개념

### 객체 형태

| object | \{age:30\} | 자바스크립트 객체, 더 특정한 타입의 객체도 가능함 |
| ------ |------------| ------------------------------------------------- |

```tsx
const person: {
  name: string
  age: number
} = {
  // const person = {
  name: 'Maximilian',
  age: 30,
}

console.log(person.name)
```

- 키 타입을 명시하여, 타입스크립트가 이 코드를 `추론`할 수 있도록 해야 함
- `타입`과 `타입 배정`은 자바스크립트가 아닌 `타입스크립트`에만 해당하므로, 컴파일이 완료된 자바스크립트에는 타입과 관련된 코드는 생략됨
  - 타입스크립트는 작업 중인 객체를 이해할 수 있도록 해주는 객체 타입 표현 방식일 뿐 실제로 자바스크립트의 객체를 생성하지는 않음

### 배열 타입

| Array | [1, 2, 3] | 어떤 자바스크립트 배열이든 지원하며 배열의 타입을 유연하게, 제한적으로도 지정할 수 있음 |
| ----- | --------- | --------------------------------------------------------------------------------------- |

```tsx
// const person: {
// name: string;
// age: number;
// } = {
const person = {
  name: 'Maximilian',
  age: 30,
  hobbies: ['Sports', 'Cooking'],
}

let favoriteActivities: string[]
favoriteActivities = ['Sports']

console.log(person.name)

for (const hobby of person.hobbies) {
  console.log(hobby.toUpperCase())
}
```

- 타입스크립트는 `hobbies`가 문자열의 배열 타입으로 이해하므로, `person.hobbies`를 입력하면 `hobbies`를 문자열의 배열이라고 인식함
  - 문자열로 지정되는 hobby를 사용하여 어떤 작업이든 가능하게 해줌
  - 가령 map과 같이 문자열에서 사용하지 못하는 메서드를 인식하여, Error를 발생시킴

### 튜플 작업하기

| Tuple | [1,2] | 타입스크립트에서 추가된 길이가 고정된 배열, 타입도 고정할 수 있음 |
| ----- | ----- | ----------------------------------------------------------------- |

```tsx
const person: {
  name: string
  age: number
  hobbies: string[]
  role: [number, string]
} = {
  name: 'Maximilian',
  age: 30,
  hobbies: ['Sports', 'Cooking'],
  role: [2, 'author'],
}

// person.role.push('admin');
// person.role[1] = 10; // Error!
```

- 배열의 `길이`와, `타입`을 고정
  - 위에서 정의한 구조와 `정확히 일치`하게 값을 할당해야 함
- push 메서드는 타입스크립트에서 예외적으로 튜플에서 허용되지만, 잘못된 값을 할당하지는 않음

> `튜플`을 사용하여 `작업 중인 데이터 타입`과 `예상되는 데이터 타입`을 보다 명확하게 `파악`할 수 있음

### 열거형으로 작업하기

| Enum | enum \{ NEW, OLD \} | 자바스크립트에는 존재하지 않는 타입, 0부터 시작하는 숫자로 전역적인 상수 식별자 개념으로 쓰임 |
| ---- |---------------------| --------------------------------------------------------------------------------------------- |

```tsx
// const ADMIN = 0;
// const READ_ONLY = 1;
// const AUTHOR = 2;

enum Role { ADMIN, READ_ONLY, AUTHOR };

const person {
	name: 'Maximilian',
	age: 30,
	hobbies: ['Sports', 'Cooking'],
	role: Role.ADMIN
};

if(person.role === Role.AUTHOR) {
	console.log('is auth');
}
```

- `Enum` 키워드를 사용해 라벨을 0부터 시작하는 숫자로 할당하게 해줌
  - 시작 숫자는 =(등호)를 통해 초기화하면 값을 할당한 식별자 다음의 식별자들은 증가됨
- 백그라운드에서 매핑된 값이 있는 식별자가 필요할 때 훌륭한 선택지가 될 수 있음

### Any 타입

| Any | \*  | 모든 종류의 값, 특정되지 않은 타입 |
| --- | --- | ---------------------------------- |

- 아주 유연하고 훌륭한 타입 같지만, 타입스크립트가 주는 모든 장점을 any가 상쇄시키므로 가급적 쓰지 않는 것이 좋음
  - 런타임 검사를 수행하는 경우, 특정 값에 수행하고자 하는 작업의 범위를 좁히기 위해서 any를 사용하고 그 외의 경우 사용하지 않는 것이 좋음
  - any 타입을 사용하는 모든 위치에서는 타입스크립트 컴파일러가 작동을 하지 않게 되므로 any 속성 또는 any 변수가 어떤 값도 저장하지 않아 컴파일러가 검사할 부분이 없어짐

### 조합(Union) 타입

```tsx
function combine(input1: number | string, input2: number | string) {
  let result
  if (typeof input1 === 'number' && typeof input2 === 'number') {
    result = input1 + input2
  } else {
    result = input1.toString() + input2.toString()
  }
  return result
}

const combinedAges = combine(30, 26)
console.log(combinedAges)

const combinedNames = combine('Max', 'Anna')
console.log(combinedNames)
```

- 파이브 기호(`|`)를 통해 두 가지 이상의 타입을 필요한 만큼 사용함
- 타입스크립트는 유니언 타입만 이해할뿐 유니언 타입 내에 무엇이 있는지는 분석하지 못함
  - 타입스크립트는 그저 여러 타입이 입력됐으니, 더하기 연산자를 사용할 수 없는 타입도 있을 것이라고 이해함
- `유니언 타입`을 사용하여 작업할 때에는 추가적인 `런타임 타입 검사`를 통해 코드에 적용한 매개변수를 보다 유연하게 처리할 수 있음
  - 타입에 따라 함수 내에 다른 로직을 적용할 수 있음

### 리터럴 타입

```tsx
function combine(
  input1: number | string,
  input2: number | string,
  resultConversion: 'as-number' | 'as-text'
) {
  let result
  if (
    (typeof input1 === 'number' && typeof input2 === 'number') ||
    resultConversion === 'as-number'
  ) {
    result = input1 + input2
  } else {
    result = input1.toString() + input2.toString()
  }
  return result
}

const combinedAges = combine(30, 26, 'as-number')
console.log(combinedAges) // 56

const combinedStringAges = combine('30', '26', 'as-number')
console.log(combinedStringAges) // 56

const combinedNames = combine('Max', 'Anna', 'asText')
console.log(combinedNames) // MaxAnna
```

- 스트링이나 숫자 등과 같은 핵심 타입과 마찬가지로 특정 문자열을 특정하여 값을 허용할 수 있음
  - `resultConversion`은 `‘as-number’`과 `‘as-text’` 두 값중 하나여야 하고 다른 값은 허용하지 않으며 해당 값들을 보장받을 수 있음
  - 해당하지 않는 값을 대입하면 컴파일 에러를 일으킴
- 유니언 타입을 결합하는 데 사용

### 타입 알리어스 / 사용자 정의 타입

```tsx
type Combinable = number | string
type ConversionDescriptor = 'as-number' | 'as-text'

function combine(input1: Combinable, input2: Combinable, resultConversion: ConversionDescriptor) {
  let result
  if (
    (typeof input1 === 'number' && typeof input2 === 'number') ||
    resultConversion === 'as-number'
  ) {
    result = input1 + input2
  } else {
    result = input1.toString() + input2.toString()
  }
  return result
}
```

- 항상 유니언 타입을 반복하는 것이 번거울 때 `별칭`을 이용하여 유니언 타입을 저장할 수 있는 타입을 생성
- 사용자 정의 타입인 `Combinable`, `ConversionDescriptor`를 추가
  - 사용자 정의 타입에 유니언 타입을 저장해서 `Combinable`을 참조하게 함
  - 코드의 양을 줄이고 재사용에 용이함
- 타입 별칭을 사용하여 타입을 직접 생성할 수 있음

  ```tsx
  type User = { name: string; age: number }
  const u1: User = { name: 'Max', age: 30 } // this works!
  ```

  - 타입 별칭을 사용하면 불필요한 반복을 피하고 타입을 중심에서 관리할 수 있음

- `단순화 전`

  ```tsx
  function greet(user: { name: string; age: number }) {
    console.log('Hi, I am ' + user.name)
  }

  function isOlder(user: { name: string; age: number }, checkAge: number) {
    return checkAge > user.age
  }
  ```

- `단순화 후`

  ```tsx
  type User = { name: string; age: number }

  function greet(user: User) {
    console.log('Hi, I am ' + user.name)
  }

  function isOlder(user: User, checkAge: number) {
    return checkAge > user.age
  }
  ```

## 함수 & 타입

### 함수 반환 타입 및 무효

```tsx
function add(n1: number, n2: number) {
  return n1 + n2
}

function printResult(num: number): void {
  console.log('Result: ' + num)
}

printResult(add(5, 12))

// let someValue: undefined;
```

- 반환하고자 하는 타입을 명시적으로 할당할 수 있음
  - 추론이 가능한 동일한 타입이어야 함
  - 변수와 마찬가지로 타입 추론에 대한 작업을 수행할 수 있도록 하는 것이 좋음
- `void` vs `undefined`
  - `void`
    - 자바스크립트에는 반환 타입이 없지만, 타입스크립트에는 `아무것도 반환하지 않는 타입`을 명시할 때 사용
  - `undefined`
    - 드물게 쓰이지만 void와 다르게 `실제 값을 반환하지 않을 때` 사용

### 타입의 기능을 하는 함수

```tsx
function add(n1: number, n2: number){
	return n1 + n2;
}

printResult(add(5, 12));

let combineValues;

combineValues = add;
combineValues = 5; // Uncaught TypeError: combineValues is not a function

console.log(combineValues(8, 8);
```

- 변수에 함수를 할당해 함수를 실행
  - 타입스크립트 관점에서는 `combineValues`가 단지 `any` 타입이라는 것만 알 수 있음
  - 원하는 결과와 다르게 타입스크립트는 컴파일 에러를 발생하지 않고, 런타임 에러를 발생함

```tsx
let comvineValues = Function
```

- `combineValues`가 함수를 할당하게 명시
  - 변수에 함수가 되어야 함을 명확히하기 위해 타입을 함수로 설정

```tsx
let combineValues: (a: number, b: number) => number

combineValues = add
```

- 함수가 어떻게 동작해야 할지 명확하게 정의
  - 따라서 `combineValues`의 매개변수가 `number`이고 반환타입이 number임을 명확하게 정의함

### 함수 타입 및 콜백

```tsx
function addAndHandle(n1: number, n2: number, cb: (num: number) => void) {
  const result = n1 + n2
  cb(result)
}

addAndHandle(10, 20, (result) => {
  console.log(result)
})
```

- `addAndHandle` 함수는 두 숫자를 결합한 다음 두 숫자를 조건에 부함하는 `callback`으로 호출
- 함수 내에 `callback`을 전달하면 타입스크립트는 해당 결과가 `number`가 될 것이라고 추론할 수 있고 `addAndHandle` 함수에서 인수 하나를 `callback`으로 가져온다고 명시했으므로, 타입스크립트는`result` 인수가 `number` 타입임을 알고 있음
- 반환값이 `void` 타입을 명시했이므로, 호출자에서 어떤 반환문을 사용해도 무시됨

### Unknown 타입

```tsx
let userInput: unknown
let userName: string

userInput = 5
userInput = 'Max'

if (typeof userInput === 'string') {
  userName = userInput
}

// userName = userInput; // Type 'unknown' is not assignable to type 'string'
```

- 타입스크립트에서 기본 제공되는 `any`와는 다른 타입으로써, `any` 처럼 에러 발생없이 어떤 값이든 저장할 수 있음
  - `any`보다는 제한적으로 사용됨
- `userName`에 `userInput`을 반환하는 경우 에러를 발생함
  - `Type 'unknown' is not assignable to type 'string'`
    - string에 unknow 타입을 할당 할 수 없음
- `unknown` 타입을 사용할 경우, if 문을 만들어 `typeof userInput`을 입력하고 문자열과 같은 지 검증
  - 추가적인 타입 검사를 통해 어떤 작업을 수행할지 명시
- unknown보다는 `유니언 타입` 사용을 권장

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
