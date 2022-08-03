---
title: 차세대 자바스크립트와 TypeScript
date: '2022-08-03'
tags: ['typescript']
draft: false
summary: 최신 자바스크립트의 기능 소개와 타입스크립트에서의 사용 예제
layout: PostSimple
authors: ['default']
---

# 차세대 자바스크립트와 TypeScript

## let 및 const

- core-js와 결합한 타입스크립트가 지원하는 ECMAScript 표준을 모아 볼 수 있는 [링크](https://kangax.github.io/compat-table/es6/)를 참고하여, 이 기능이 어떻게 작동하는지, 무엇이 제한되는지, 어떤 브라우저가 지원하는지 확인할 수 있음
- ES6에서 추가된 let, const 키워드
- `const` 키워드

  ```tsx
  const userName = 'Max'
  // userName = 'Maximilian';
  let age = 30

  age = 29
  ```

  - const 키워드로 찾는 선언하는 변수는 변경할 수 없는 것이 목적(상수의 개념)
  - 새로운 값을 할당하려고 하면 그 값이 정확한 타입이어도 에러 발생

- `let` vs `var` 키워드로 알아보는 차이점
  - `var`는 `let`과 `const`의 개념이 등장하기 전에 사용된 변수를 선언 키워드
  - 변수를 사용할 수 있는 유효 범위가 다름
    - `var`의 경우 `함수` 레벨 스코프
      - 잠재된 함수 호이스팅 발생
      - 예측 불가능한 값의 변경
    - `let`, `const`는 `블록` 레벨 스코프
      - 코드 블럭안에서 유효함

## 화살표 함수

- ES6 이후 버전에 자바스크립트에 추가된 함수

  ```tsx
  const add = (a:number, b:number) => a + b;

  const printOutput = (output: string | number) => void = output => console.log(output);
  ```

  - `function` 키워드를 사용하지 않고 등호와 `()` 부등호`{}`로 화살표를 만들고 함수를 정의
  - `return`을 생략하고도 값을 반환할 수 있음
  - 표현식이 `하나`만 있는 경우에 부등호`{}`를 생략할 수 있음

## 기본값 함수 매개변수

```tsx
const add = (a:number, b:number = 1) => a + b;

const printOutput = (a: number | string) => void = output => console.log(output);

printOutput(add(5));
```

- 타입이 일치하는 기본값을 설정하여 구현할 수 있는 기능 제공
- 매개변수에 전달된 `순서와 동일하게 전달`되므로, 기본값이 할당되어 있는 매개 변수의 순서에 값을 넘기면 에러가 발생
  - 되도록이면 `기본값 인수를 허용하지 않는 매개변수`가 기본값 매개변수 `앞`에 먼저 오도록 지정하는 것을 권장

## 스프레드 연산자(…)

```tsx
const hobbies = ['Sports', 'Cooking'];
const activeHobbies = ['Hiking'];

activeHobbies.push(...hobbies);

const person = {
	name: 'Max'.
	age: 30
}

const copiedPerson = { ...person };
```

- 최신 자바스크립트에서 사용 가능한 스프레드 연산자
- `배열` 혹은 `객체`의 모든 값을 추출하고자 할 때 주로 사용함
  - `hobbies` 배열의 모든 요소를 꺼내 값의 목록으로서 추가
  - 객체에서는 배열과 달리 단일 값이 아닌 `키-값` 쌍이 추가됨
  - `copiedPerson`에는 새 객체를 생성하여 `키-값` 쌍에 추가했으므로 메모리의 객체를 지시하는 포인터뿐 아니라 원본 객체의 완벽한 복사본도 만들 수 있음

## 나머지 매개변수

```tsx
const add = (...numbers: number[]) => {
  let result = 0
  return numbers.reduce((curResult, curValue) => {
    return curResult + curValue
  }, 0)
}

const addNumbers = add(5, 10, 2, 3.7)
console.log(addedNumbers)
```

- 위 처럼 여러 값을 입력해야 하는 경우 나머지 매개변수를 사용해 보다 유연하게 처리할 수 있음
  - 전달된 모든 매개변수 또는 일반적으로 들어오는 쉼표로 분리된 값의 목록을 배열로 병합합
- 여러 인수를 지원하고자 할 때 지원할 인수의 수를 알고 있는 경우, `튜플` 타입을 사용해 고정된 길이의 배열을 병합할 수 있음

  ```tsx
  const add = (...numbers: [number, number, number]) => {
    let result = 0
    return numbers.reduce((curResult, curValue) => {
      return curResult + curValue
    }, 0)
  }

  const addNumbers = add(5, 10, 2)
  console.log(addedNumbers)
  ```

## 배열 및 객체 구조 분해 할당

```tsx
const hobbies = ['Sports', 'Cooking']

const [hobby1, hobby2, ...remainingHobbies] = hobbies

console.log(hobbies, hobby1, hobby2)

const { firstName: userName, age } = person

console.log(userName, age)
```

- 구조 분해 할당을 통해 코드의 양을 줄일 수 있음
- 배열이나 객체에서 필요한 요소를 `추출`해 새로운 변수에 `할당`
  - 새로운 상수나 변수로 복사되는 것이지 원본을 변경하는 것이 아님
  - 배열은 정렬된 목록이므로 요소는 `순서대로 추출됨`
  - 순서는 보장되지 않으므로 위치별로 요소를 추출하지 않고 `키 이름`으로 요소를 추출함
  - 할당 시 쌍점(`:`)을 이용해 이름을 변경할 수 있음(별칭 가능)
    - 타입스크립트의 타입 배정으로 인식하지 않음

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
