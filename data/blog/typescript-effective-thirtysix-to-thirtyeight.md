---
title: 타입 설계 & any 다루기 - Item 36 ~ 38
date: '2022-12-08'
tags: ['typescript']
draft: false
summary: 해당 분야의 용어로 타입 이름 짓기, 공식 명칭에는 상표를 붙이기, any 타입은 가능한 한 좁은 범위에서만 사용하기
layout: PostSimple
authors: ['default']
---


# 타입 설계 & any 다루기 - Item 36 ~ 38

## Item 36) 해당 분야의 용어로 타입 이름 짓기

> 컴퓨터 과학에서 어려운 일은 단 두가지 뿐이다. 캐시 무효와와 이름 짓기
`필 칼튼(Phil Karlton)`
>
- 타입 설계에서 중요한 부분이라고 할 수 있는 이름 짓기는 엄선된 타입, 속성, 변수의 의도를 반영해야 하고 이는 코드와 타입의 추상화 수준을 높여줌
- 잘못 선택한 타입 이름은 코드의 의도를 왜곡하고 잘못된 개념을 심어줌

### Worst Case

```tsx
interface Animal {
	name: string;
	endangered: boolean;
	habitat: string;
}

const leopard: Animal = {
	name: 'Snow Leopard',
	endangered: false,
	habitat: 'tundra',
};
```

- 이 코드의 문제점은 다음과 같습니다.
  1. `name`은 매우 일반적인 용어이므로 동물의 학명인지 일반적인 명칭인지 알 수 없음
  2. `endangered` 속성이 멸종 위기를 표현하기 위해 `boolean` 타입을 사용한 것이 이미 멸종된 동물을 `true`로 해야 하는지 판단할 수 없음
  3. 서식지를 나타내는 `habitat` 속성은 너무 범위가 넓은 `string` 타입일 뿐만 아니라 서식지라는 뜻 자체도 불분명하기 때문에 다른 속성들 보다 모호함
  4. 객체의 변수명이 `leopard`이지만, `name` 속성의 값은 `'Snow Leopard'`이므로 객체의 이름과 속성이 name이 다른 의도로 사용된 것인지 불분명함

### Best Case

```tsx
interface Animal {
	commonName: string;
	genus: string;
	species: ConservationStatus;
	climates: KoppenClimate[];
}
type ConservationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
type KoppenClimate = | 
	'Af' | 'Am' | 'As' | 'Aw' | 
	'BSh' | 'Bsk' | 'BWh' | 'BWk' | 
	'Cfa' | 'Cfb' | 'Cfc' | 'Csa' | 'Csb' | 'Csc' | 'Cwa' | 'Cwb' | 'Cwc' | 
	'Dfa' | 'Dfb' | 'Dfc' | 'Dfd' | 
	'Dsa' | 'Dsb' | 'Dsc' | 'Dwa' | 'Dwb' | 'Dwc' | 'Dwd' |
	'EF' | 'ET';
const snowLeopard: Animal = {
	commonName: 'Snow Leopard',
	genus: 'Panthera',
	species: 'Uncia',
	sttus: 'VU', // 취약종
	climates: ['ET', 'EF', 'Dfd'], // 고산대 또는 아고산대
}
```

- 이 코드는 아래와 같이 개선되었음
  1. `name`은 `commonName`, `genus`, `species` 등 더 구체적인 용어로 대체
  2. `endangered`는 동물 보호 등급에 대한 `IUCN`의 표준 분류 체계인 `ConservationStatus` 타입의 `status`로 변경됨
  3. `habitat`은 기후를 뜻한 `climates`로 변경되었으며, 쾨펜 기후 분류를 사용함
- 해당 예제를 통해 `Worst Case`보다 데이터를 명확하게 표현하였으며 정보를 찾기위해 사람에 의존할 필요가 없음

### 정리

- 코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 용어들을 사용하여 사용자와 보다 유리한 소통을 하고 타입의 명확성을 올릴 수 있음
- 타입, 속성, 변수에 이름을 붙일 때 명심해야 할 세가지 규칙이 있음
  1. 작문과 달리 코드에서는 동의한 의미는 다른 용어를 사용해 의미론적으로 분리를 해야함
  2. `data`, `info`, `thing`, `item`, `object`, `entity` 같은 모호하고 의미 없는 이름은 피해야 함
  3. 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지 고려해야 함
    - `INodeList`보다는 `Directory`를 사용해 더 의미 있는 이름으로 정의하여 개념적인 측면으로 접근하고, 이는 곧 추상화의 수준이 상승됨과 더불어  의도치 않은 충돌의 위험성을 줄여 줌

## Item 37) 공식 명칭에는 상표를 붙이기

- 상표 기법은 타입 시스테에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있음
- 타입 시스템이므로 런타임 오버헤드를 없앨 수 있고 추가 속성을 붙일 수 없는 `string`이나 `number` 같은 내장 타입도 상표화 할 수 있음

### 예시 - 상표 기법 사용

- 절대 경로를 사용해 파일 시스템에 접근하는 함수를 가정하자면, 런타임에는 절대 경로(`'/'`)로 시작하는지 체크하기 쉽지만, 타입 시스템에서는 절대 경로를 판단하기 어렵기 때문에 상표 기법을 사용

```tsx
type AbsolutePath = string & {_brand: 'abs'};
function listAbsolutePath(path: AbsolutePath) {
	// ...
}
function isAbsolutePath(path: string): path is AbsolutePath {
	return path.startWith('/');
}
```

- `string` 타입이면서 `_brand` 속성을 가지는 객체를 만들 수는 없음
  - `AbsolutePath`는 온전히 타입 시스템의 영역임
- 만약 `path` 값이 절대 경로와 상대 경로 둘 다 될 수 있다면, 타입을 정제해 주는 타입 가드를 사용해서 오류를 방지할 수 있음

```tsx
function f(path: string) {
	if (isAbsolutePath(path)){
		listAbsolutePath(path);
	}
	listAbsolutePath(path);
	// 'string' 형식의 인수는 'AbsolutePath' 형식의
	// 매개변수에 할당될 수 없습니다.
}
```

- `if` 체크로 타입을 정제하는 방식은 매개변수 `path`가 절대 경로인지 또는 상대  경로인지에 따라 분기하기 때문에 분기하는 이유를 주석으로 붙이기 좋지만, 주석이 오류를 방지해 주는 것이 아니라는 점을 주의해야 함
- 반면, 로직을 분기하는 대신 오류가 발생한 곳에 `path as AbsolutePath`를 사용해서 오류를 제거할 수도 있지만 단언문을 쓰지 않고, 타입을 얻기 위한 유일한 방법은 `AbsolutePath` 타입을 매개변수로 받거나 타입이 맞는지 체크하는 것 뿐임

### 예시 - 속성 모델링

- 목록에서 한 요소를 찾기 위해 이진 검색을 하는 예시

```tsx
function binarySearch<T>(xs: T[], x: T): boolean {
	let low = 0, high = xs.length - 1;
	while (high >= low) {
		const mid = low + Math.floor((high - low) / 2);
		const v = xs[mid];
		if (v === x) return true;
		[low, high] = x > v ? [mid + 1, high] : [low, mid - 1];
	}
	return false;
}
```

- 이진 검색은 이미 정렬된 상태를 가정하기 때문에, 목록이 이미 정렬되어 있다면 문제가 없지만 목록이 정렬되어 있지 않다면 잘못된 결과를 초래함
- 타입스크립트 타입 시스템에서는 목록이 정렬되어 있다는 의도를 표현하기 어려우므로, 상표 기법을 사용해야 함

    ```tsx
    type SortedList<T> = T[] & {_brand: 'sorted'};
    
    function isSorted<T>(xs: T[]): xs is SortedList<t> {
    	for (let i = 1; i < xs.length; i++) {
    		if (xs[i] < xs[i-1]) {
    			return false;
    		}
    	}
    	return true;
    }
    
    function binarySearch<T>(xs: SortedList<T>, x: T): boolean {
    	// ...
    }
    ```

  - `binarySearch`를 호출하려면, 정렬되었다는 상표가 붙은 `SortedList` 타입의 값을 사용하거나 `isSorted`를 호출하여 정렬되었음을 증명해야 함
    - `isSorted`에서 목록 전체를 루프도는 것이 효율적인 방법은 아니지만 적어도 안정성은 확보할 수 있음
  - 예를 들어, 객체의 메서드를 호출하는 경우 `null`이 아닌 객체를 받거나 조건문을 사용해서 해당 객체가 `null`이 아닌지 체크하는 코드와 동일한 형태임

### 예시 - 상표 사용(number 타입)

```tsx
type Meters = number & {_brand: 'meters'};
type Seconds = number & {_brand: 'seconds'};

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000); // 타입이 Meters
const oneMin = seconds(60); // 타입이 Seconds
```

- number 타입에 상표를 붙여ㅕ도 산술 연산 후에는 상표가 없어지기 때문에 실제로 사용하는데는 무리가 있음
- 그러나 코드에 여러 단위가 혼합된 많은 수의 숫자가 들어 있는 경우, 숫자의 단위를 문서화하는 괜찮은 방법일 수 있음

    ```tsx
    const tenKm = oneKm * 10; // 타입이 number
    const v = oneKm / oneMin; // 타입이 number
    ```


## Item 38) any 타입은 가능한 한 좁은 범위에서만 사용하기

```tsx
function processBar(b: Bar) { /* ... */ }

function f() {
	const x = expressionReturningFoo();
	precessBar(x);
	// 'Foo' 형식의 인수는 'Bar' 형식의 매개변수에 할당될 수 없습니다.
}
```

- 문맥상으로 `x`라는 변수가 동시에 `Foo` 타입과 `Bar` 타입에 할당 가능하다면, 오류를 제거하는 방법을 두 가지임

    ```tsx
    function f1() {
    	const x: any = expressionReturningFoo(); // 🙅🏻‍♂️
    	processBar(x);
    }
    
    function f1() {
    	const x = expressionReturningFoo(); // 🙆🏻‍♂️
    	processBar(x as any);
    }
    ```

  - `f2`에 사용된 `x as any`형태가 권장되는 이유는 `any` 타입이 `processBar` 함수의 매개변수에서만 사용된 표현식이므로 다른 코드에 영향을 미치지 않음
- 타입스크립트가 함수의 반환 타입을 추론하여도 반환 타입을 명시하는 것이 좋음
  - any 타입이 함수 바깥으로 영향을 미치는 것을 방지할 수 있음
- `f1`은 오류를 제거하기 위해 `x`를 `any` 타입으로 선언했지만 `f2`는 오류를 제거하기 위해 `x`가 사용되는 곳에 `as any` 단언문을 사용함
  - `@ts-ignore`를 사용하면 `any`를 사용하지 않고 오류를 제거할 수 있음
  - 그러나 근본적인 원인을 해결한 것이 아니므로 다른 곳에서 큰 문제가 발생할 위험성이 있음

### any의 사용법

```tsx
const config: Config = {
	a: 1,
	b: 2,
	c: {
		key: value
		// 'foo' 속성이 'Foo' 타입에 필요하지만 'Bar' 타입에는 없습니다.
	}
}
```

- 단순히 생각하면 `config` 객체 전체를 `as any`로 선언해서 오류를 제거할 수 있지만, 객체 전체를 `any`로 단언하면 다른 속성들 역시 타입 체크가 되지 않는 부작용이 생기므로 최소한의 범위에서만 `any`를 사용하는 것이 좋음

    ```tsx
    const config: Config = {
    	a: 1,
    	b: 2, // 이 속성은 여전히 체크됨
    	c: {
    		key: value as any
    	}
    }
    ```

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 195 ~ 209p
