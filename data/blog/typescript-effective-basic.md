---
title: 타입스크립트 알아보기
date: '2022-08-28'
tags: ['typescript']
draft: false
summary: 자바스크립트 프로그래밍이 타입스크립트라는 명제는 참이지만 반대는 성립하지 않음
layout: PostSimple
authors: ['default']
---

# 타입스크립트 알아보기

## 타입스크립트와 자바스크립트의 관계 이해하기

### 개요

- 타입스크립트는 문법적으로도 자바스크립트의 상위 집합(`superset`)임
- 자바스크립트 파일이 `.js`(또는 `.jsx`) 확장자를 사용하는 반면, 타입스크립트 파일은 `.ts`(또는 `.tsx`) 확장자를 사용함
  - 파일의 확장자를 js → ts로 바꾼다고 해도 달라지는 것은 없음
  - 이러한 특성은 자바스크립트 코드를 타입스크립트 코드로 마이그레이션하는 데 엄청난 이점이 됨
- 자바스크립트 프로그래밍이 타입스크립트라는 명제는 참이지만 반대는 성립하지 않음
  - 타입스크립트 프로그램이지만 자바스크립트가 아닌 프로그램이 존재함

### 타입 추론

```tsx
let city = 'new york city'
console.log(city.toUppercase())
// TypeError: city.toUppercase is not a function
```

- 앞의 코드에는 타입 구문이 없으나, 타입스크립트의 타입 체커는 문제점을 찾아냄

```tsx
let city = 'new york city'
console.log(city.toUppercase())
// 'toUppercase' 속성이 'string' 형식에 없습니다.
// 'toUpperCase'을(를) 사용하시겠습니까?
```

- city 변수가 문자열이라는 것을 알려 주지 않아도, 타입스크립트는 초깃값으로부터 타입을 추론함

### 타입스크립트 타입 시스템의 기본 원칙

```tsx
const x = 2 + '3' // 23
const y = '2' + 3 // 23
```

- 다른 언어였다면, 런타임 오류가 될 만한 코드지만, 타입스크립트의 타입 체커는 정상으로 인식하며, 두 줄 모두 문자열 `23`이 되는 자바스크립트 `런타임 동작으로 모델링`됨

```tsx
const a = null + 7 // 자바스크립트에서는 a값이 7이 됨
// '+' 연산자를 ... 형식에 적용할 수 없습니다.
const b = [] + 12 // 자바스크립트에서는 b값이 12가 됨
// '+' 연산자를 ... 형식에 적용할 수 없습니다.
alert('Hello', 'TypeScript') // "Hello" 경고를 표시합니다.
// 0-1개의 인수가 필요한데 2개를 가져왔습니다.
```

- 런타임 오류가 발생하지 않는 코드임에도 불구하고 타입 체커는 문제점을 표시함

> `자바스크립트의 런타임 동작을 모델링`하는 것은 타입스크립트 타입 시스템의 기본 원칙

- 앞에서 봤던 경우들처럼 단순히 런타임 동작을 모델링하는 것 뿐만 아니라 의도치 않은 이상한 코드가 오류로 이어질 수도 있다는 점까지 고려해야 함

## 타입스크립트 설정 이해하기

- 타입스크립트 컴파일러는 매우 많은 설정을 가지고 있음

### `CLI`

```bash
$ tsc --noImplicitAny program.ts
```

- `tsc` 커맨드 옵션으로 직접 특정 파일에 대해 옵션 적용

### `tsconfig.json`

```bash
{
	"compilerOptions": {
		"noImplicitAny": true
	}
}
```

- `tsc —init`을 실행해서 설정 파일을 간단히 생성할 수 있음

### noImplicitAny 옵션

- 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어

```tsx
function add(a, b) {
  return a + b
}

// function add(a: any, b: any): any
```

- 에디터에서 `add` 부분에 마우스를 올려보면, 타입 스크립트가 함수의 타입을 `any`라고 추론하는 것을 확인할 수 있음
  - `any` 타입을 지정하지 않았지만, 암묵적 `any` 타입으로 간주됨
- `any` 타입을 매개변수에 사용하면 타입 체커는 속절없이 무력해짐
  - `noImplicitAny` 옵션을 설정해 암묵적인 `any` 타입으로 간주되는 코드를 오류로 출력하게 하고, 명시적으로 `any` 타입으로 지정하는 것이 좋음
- 타입스크립트는 타입 정보를 가질 떄 가장 효과적이므로, 되도록이면 `noImplicitAny`를 설정해야 함

### strictNullChecks 옵션

- `null`과 `undefined`가 모든 타입에서 허용되는지 확인하는 설정

```tsx
const x: number = null
// 'null' 형식은 'number' 형식에 할당할 수 없습니다.
```

- 만약 null을 허용하려고 한다면, 의도를 명시적으로 드러냄으로써 오류를 수정할 수 있음

```tsx
const x: number | null = null
```

- null과 undefined 관련된 오류를 잡아 내는 데 많은 도움이 되지만, 코드 작성을 어렵게 하므로 가급적 새 프로젝트를 시작할 때 초반에 설정하는 것을 권장함

## 코드 생성과 타입이 관계없음을 이해하기

### 타입스크립트 컴파일러 역할

1. 최신 `타입스크립트`/`자바스크립트`를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일 함
2. 코드의 타입 오류를 체크

- 타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 전혀 주지 않음
- 자바스크립트의 실행 시점에도 타입은 영향을 미치지 않음

### 타입 오류가 있는 코드도 컴파일이 가능함

- 컴파일은 타입 체크와 독립적으로 동작하므로 타입 오류가 있는 코드도 컴파일이 가능함
  - 오류가 있음에도 컴파일이 되는게 실제로 도움이 될 수 있음
    - 문제가 된 오류를 수정하지 않더라도 애플리케이션의 다른 부분은 테스트 할 수 있음
  - 컴파일 시 오류가 있을 때 중단하려면 `tsconfig.json`에 `noEmitOnError`를 설정

### 런타임에는 타입 체크가 불가능

- 타입스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 모두 제거됨
- 타입 정보를 유지하는 방법으로 타입 정보를 명시적으로 저장하는 `태그` 기법을 사용

### 인터페이스를 사용해 타입 정보 유지

```tsx
interface Square {
  kind: 'square'
  width: number
}
interface Rectangle {
  kind: 'rectangle'
  height: number
  width: number
}

type Shape = Square | Rectangle

function cacluateArea(shape: Shape) {
  if (shape.kind === 'rectangle') {
    shape // 타입이 Rectangle
    return shape.width * shape.height
  } else {
    shape // 타입이 Square
    return shape.width * shape.width
  }
}
```

- 런타임에 타입 정보를 손쉽게 유지할 수 있음
- `type Shape = Square | Rectangle;`
  - `Rectangle`은 타입으로 참조됨

### 클래스 선언 방식으로 타입 유지

```tsx
class Square {
  constructor(public width: number){}
}

class Rectangle extends Square {
	constructor(public width: number, public height: number){
		super(width);
	}
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
	if (shape instanceof Rectangle) {
		shape; // 타입이 Rectangle
		return shape.width * shape.height;
	} else {
		shape; // 타입이 Square
		return shape.width * shape.width;
	}
}
```

- `Rectangle`을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없음
- `shape instanceof Rectangle` 부분에서는 값으로 참조됨

### 타입 연산은 런타임에 영향을 주지 않음

`typescript`

```tsx
function asNumber(val: number | string): number {
  return val as number
}
```

`javascript`

```tsx
function asNumber(val) {
  return val
}
```

- 변환된 자바스크립트 파일을 보면, 아무런 정제 과정이 없음
- 값을 정제하기 위해서는 런타임의 타입을 체크하고 자바스크립트 연산을 통해 변환을 수행해야 함

```tsx
function asNumber(val: number | string): number {
  return typeof val === 'string' ? Number(val) : val
}
```

### 런타임의 타입은 선언된 타입과 다를 수 있음

```tsx
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn()
      break
    case false:
      turnLightOff()
      break
      defeault: console.log('실행되지 않을까 걱정입니다.')
  }
}
```

- 타입스크립트는 일반적으로 실행되지 못하는 코드를 찾아내지만 위에서는 `strict`를 설정하더라도 찾아내지 못함
- `:boolean`이 타입 선언문이므로 런타임에 제거됨
- 자바스크립트였다면 실수로 `setLightSwitch`를 `ON`으로 호출할 수도 있음
- 이는 타입스크립트에서도 마찬가지며 런타임 타입과 선언된 타입이 언제든지 달라질 수 있다는 것을 명심해야 함

### 타입스크립트 타입으로는 함수를 오버로드 할 수 없음

- C++ 같은 언어는 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용함(함수 오버로딩)
- 타입스크립트는 런타임과의 동작과 무관하므로 함수 오버로딩은 불가능하지만 타입 수준에서는 동작함

```tsx
type Combinable = string | number
function add(a: number, b: number): number;
function add(a: string, b: string): string;
function add(a: Combinable, b: Combinable) { return a + b };
```

- 하나의 함수에 대해 여러 개의 선언문은 작성할 수 있지만, 구현체는 오직 하나임

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않음

- 자바스크립트 변환 시점에 타입과 타입 연산자가 제거되므로 런타임 성능에 아무런 영향을 주지 않음
- 타입스크립트 팀이 제시한 주의사항
  - 런타임 오버헤드는 없지만, 빌드타임 오버헤드가 있음
    - 컴파일은 일반적으로 상당히 빠른 편이며 증분(`incremental`) 빌드 시에 더욱 체감됨
    - 오버헤드가 커질 시 빌드 도구에서 `transpile only`를 설정하여 타입 체크를 건너뛸 수 있음
  - 타입스크립트가 컴파일 하는 코드는 오래된 성능 오버헤드를 포기하고 런타임 환경을 선택할지, 호환성을 포기하고 성능을 중심의 네이티브 구현체를 선택할지 문제는 컴파일 타깃과 언어 레벨의 문제이지 타입과는 무관함

## any 타입 지양하기

- 코드에 타입을 추가할 수 있으므로 점진적이며, 언제든지 타입 체커를 해제할 수 있기 떄문에 선택적임

```tsx
let age: number
age = '12'
// '12' 형식은 'number' 형식에 할당할 수 없습니다.
age = '12' as any // OK
```

- 위와 같이 `any` 타입 단언문을 사용하거나 `any` 타입을 사용하게 되면 타입스크립트의 수많은 장점을 누릴 수 없음

### any 타입에는 타입 안정성이 없음

```tsx
let age: number
age = '12' as any
age += 1 // 런타임에 정상, age는 "121"
```

- 앞선 예제처럼 `as any`를 사용하게 되면 타입 체커는 선언에 따라 `number` 타입으로 판단할 것이고 혼돈은 걷잡을 수 없게 됨

### any 함수 시그니처를 무시해버림

- 함수를 작성할 때는 시그니처를 명시해함
  - 호출하는 쪽은 약속된 타입의 입력을 제공
  - 함수는 약속된 타입의 출력을 반환
- 그러나 any 타입을 사용하면 이러한 약속을 어길 수 있음

```tsx
function calculateAge(birthDate: Date): number {
  // ...
}

let birthDate: any = '1990-01-19'
calculateAge(birthDate) // 정상
```

- `birthDate` 매개변수는 `string`이 아닌 Date 타입이어야 하지만, `any` 타입을 사용하면 `calculateAge`의 시그니처를 무시하게 됨
- 자바스크립트에서는 종종 암시적으로 타입이 변환되기 때문에 이런 경우 특히 문제가 될 수 있음

### any 타입에는 언어 서비스가 적용되지 않음

- any 타입인 심벌을 사용하면 편집기 상에서 자동완성 기능과 적절한 도움말을 제공받지 못함
- 타입스크립트의 모토는 `확장 가능한 자바스크립트`이므로, 확장의 중요한 부분인 언어 서비스를 제대로 누려야 동료의 생산성이 향상됨

### any 타입은 코드 리팩토링 때 버그를 감춤

```tsx
interface ComponentProps {
  onSelectItem: (item: any) => void
}
```

- `onSelectItem` 콜백이 있는 컴포넌트를 `interface`로 정의하고 선택하려는 아이템의 타입을 `any`로 설정

```tsx
function renderSelector(props: ComponentProps) {
  /* ... */
}

let selectedId: number = 0

function handleSelectItem(item: any) {
  selectedId = item.id
}

renderSelector({ onSelectItem: handleSelectItem })
```

- `onSelectItem`에 아이템 객체를 필요한 부분만 전달하도록 컴포넌트를 개선하여 `CmponentProps`의 시그니처를 다음 처럼 변경

```tsx
interface ComponentProps {
  onSelectItme: (id: number) => void
}
```

- `any`가 아니라 구체적인 타입을 사용했다면, 타입 체커가 오류를 발견할 수 있는 문제임

### any는 타입 설계를 감춰버림

- 애플리케이션 상태 같은 객체를 정의하려면 꽤 복잡한 상황을 마주하지만 이럴때일수록 더욱이 `any`를 사용하면 안됨
- 객체를 정의하는 경우 상태 객체의 설계를 감춰버리므로 깔끔하고 명확한 코드 작성을 위해서는 제대로 된 타입 설계를 해야함
  - `any` 타입을 사용하면 타입 설계가 불분명해져 설계가 잘 되었는지, 어떻게 되어 있는지 전혀 알 수 없음

### any는 타입시스템의 신뢰도를 떨어뜨림

- 타입 체커는 실수를 잡아주고 코드의 신뢰도를 높이는 것이 목표이기 때문에, `any` 타입을 쓰지 않으면 런타임에 발견될 오류를 미리 잡을 수 있고 신뢰도 또한 높일 수 있음
- 코드 내에 존재하는 수많은 any 타입으로 인해 자바스크립트보다 일을 더 어렵게 만들기도 함
  - 타입 오류를 고쳐야 하고, 여전히 개발자 머릿속에 실제 타입을 기억해야 하기 때문임
  - 타입이 실제 값과 일치한다면, 타입 정보를 타입스크립트가 기억하므로 타입 정보를 따로 기억해 둘 필요가 없음

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 1 ~ 32p
