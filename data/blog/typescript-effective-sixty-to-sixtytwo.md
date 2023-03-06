---
title: 타입스크립트로 마이그레이션하기  - Item 60 ~ 62
date: '2023-03-06'
tags: ['typescript']
draft: false
summary: allowJS로 타입스크립트와 자바스크립트 같이 사용하기, 의존성 관계에 따라 모듈 단위로 전환하기, 마이그레이션의 완성을 위해 noImplicitAny 설정하기
layout: PostSimple
authors: ['default']
---

# 타입스크립트로 마이그레이션하기 - Item 60 ~ 62

## Item 60) allowJS로 타입스크립트와 자바스크립트 같이 사용하기

- 소규모 프로젝트는 타입스크립트로 한번에 전환할 수 있지만, 대규모 프로젝트의 경우 점진적으로 전환해야하므로 자바스크립트와 타입스크립트가 동시에 동작해야 함
- 타입스크립트와 자바스크립트가 공존하는 방법의 핵심은 `allowJs` 컴파일러 옵션인데 자바스크립트 파일과 타입스크립트 파일을 서로 `import` 할 수 있게 해줌
- 기존 빌드 과정에서 타입스크립트 컴파일러를 추가하기 위해서는 `allowJs` 옵션이 필요함
  - 또한, 모듈 단위로 타입스크립트로 전환하는 과정에서 테스트를 수행해야 하므로 allowJs 옵션이 필요함
- 번들러에 타입스크립트가 통합되어 있거나, 플러그인 방식으로 통합이 가능하다면 allowJs를 간단히 적용할 수 있음

  - ex) `npm install -D tsify`를 실행한 후 `browserify`를 사용하여 플러그인 추가

    ```bash
    $ browserify index.ts -p [ tsify --allowJs ] > bundle.js
    ```

- 대부분의 유닛 테스트 도구에는 동일한 역할을 하는 옵션이 있음

  - ex) `ts-jest`를 설치하고 `jest.config.js`에 전달할 타입스크립트 소스 지정

    ```json
    module.exports = {
      transform: {
        '^.+\\.tsx?$': 'ts-jest',
      }
    }
    ```

- 프레임워크 없이 빌드 체인을 직접 구성한 경우, `outDir` 옵션을 사용하여 타입스크립트가 `outDir`에 지정된 디렉터리에 소스 디렉터리와 비슷한 구조로 자바스크립트 코드를 생성하고 `outDir`로 지정된 디렉터리를 대상으로 기존 빌드 체인을 실행

## Item 61) 의존성 관계에 따라 모듈 단위로 전환하기

- 점진적 마이그레이션을 할 때는 모듈 단위로 각개격파는 것이 이상적임
  - 한 모듈을 골라서 타입 정보를 추가하면, 해당 모듈이 의존하는 모듈에서 비롯되는 타입 오류 발생
  - 따라서, 다른 모듈에 의존하지 않는 최하단 모듈부터 작업을 시작해 `bottom-up` 방식으로 최상단 모듈까지 완성
- 서드파티 라이브러리는 `@types` 모듈을 설치해 해당 라이브러리를 사용하는 타입정보를 해결
- 우선적으로 모듈에 의존하지 않은 외부 API 타입 정보도 추가하여 먼저 해결하는 것을 권장

### 모듈간의 의존성 파악

- [madge](https://github.com/pahen/madge)라는 의존성 관계도 도구를 사용해 의존성을 파악
- 마이그레이션할 때는 선택과 집중의 관점으로 타입 정보 추가만하고, 리팩터링은 진행하지 않음

### 선언되지 않은 클래스 멤버

- 자바스크립트는 클래스 멤버 변수를 선언할 필요가 없지만, 타입스크립트에서는 명시적으로 선언해야 함

  ```tsx
  class Greeting {
    greeting: string
    name: any
    constructor(name) {
      this.greeting = 'Hello'
      this.name = name
    }
    greet() {
      return this.greeting + ' ' + this.name
    }
  }
  ```

  - `greeting`에 대한 타입은 `string`으로 정확히 추론되나 `name`의 경우는 `any`로 채워짐

- 자바스크립트 코드를 타입스크립트로 전환하다 보면, 잘못된 부분을 발견하는 경우가 있지만 개선할 부분은 기록만해두고 리팩토링은 타입스크립트 전환 작어빙 완료된 후에 생각해야 함

### 타입이 바뀌는 값

```tsx
const state = {}
state.name = 'New York'
// '{}' 유형에 'name' 속성이 없습니다.
state.capital = 'Albany'
// '{}' 유형에 'capital' 속성이 없습니다.
```

- 자바스크립트일 때는 문제가 없지만, 타입스크립트가 되는 순간 오류가 발생함

```tsx
const state = {
  name: 'New York',
  capital: 'Albany',
} // 정상
```

- 한꺼번에 객체를 생성해서 간단한 오류를 해결할 수 있음

```tsx
interface State {
  name: string
  capital: string
}
const state = {} as State
state.name = 'New York' // 정상
state.capital = 'Albany' // 정상
```

- 임시 방편으로 타입 단언문을 사용해서 처리할 수 있음
- 당장은 마이그레이션이 급하므로 타입 단언문을 사용했으나 마이그레이션이 완료된 후에는 근본적인 문제를 제대로 해결해야 함

### JSDoc & @ts-check 혼용

- `JSDoc`과 `@ts-check`를 사용해 타입 정보를 추가한 상태라면 타입스크립트로 전환하는 순간 타입 정보가 무효화 되므로 `JSDoc` 타입 정보를 타입스크립트 타입으로 전환해 주는 `Quick Fix` 기능을 사용해 타입정보를 생성함
  - 타입 정보를 타입스크립트 타입으로 전환한 후에는 불필요해진 `JSDoc`을 제거

## Item 62) 마이그레이션의 완성을 위해 noImplicitAny 설정하기

- 프로젝트 전체를 `.ts`로 전환했다면 매우 큰 진척을 이룬 것이지만, `noImplicitAny`를 설정해 실제 오류를 처리하고 마이그레이션을 완료 지을 수 있음

```tsx
class Chart {
  indices: any

  // ...
}
```

- `Chart` 클래스에 속성 타입 선언을 추가할 때, `Quick Fix` 기능을 통해 누락된 모든 멤버 추가 기능을 사용했고 문맥 정보가 부족한 `indices`는 `any` 타입으로 추론됨

```tsx
class Chart {
  indices: number[]

  // ...
}
```

- 오류는 사라졌고 문제가 없는 것처럼 보이나 `number[]`는 잘못된 타입으로 지정되었음

```tsx
getRanges() {
	for (const r of this.indices){
		const low = r[0]; // 타입이 any
		const high = r[1]; // 타입이 any
		// ...
	}
}
```

- 명백히 `number[][]` 또는 `[number, number][]`가 정확한 타입이나 현재는 `indices`가 `number[]`로 선언되어 있기 때문에 `r`은 `number` 타입으로 추론됨
  - 이처럼 `noImplicitAny` 설정을 하지 않으면, 타입 체크는 매우 허술해짐
- 처음에는 noImplicitAny를 로컬에만 설정하고 작업하는 것이 좋음
  - 원격에서는 설정에 변화가 없기 때문에 빌드가 실패하지 않고 로컬에서만 오류로 인식되므로 수정된 부분만 커밋할 수 있어서 점진적 마이그레이션이 가능함
  - 한편 타입 체커가 발생하는 오류의 개수는 `noImplicitAny`와 관련된 작업의 진척도를 나타내는 지표로 활용할 수 있음
- 타입 체크의 강도를 높이는 방법중에 noImplicitAny는 상당히 엄격한 설정이며, strictNullChecks 같은 설정을 적용하지 않더라도 대부분의 타입 체크를 적용함
- 가장 강력한 설정은 `"strict": true`로써, 타입 체크의 강도는 팀 내의 모든 사람이 타입스크립트에 익숙해진 다음에 조금씩 높이는 것이 좋음

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 306 ~ 316p
