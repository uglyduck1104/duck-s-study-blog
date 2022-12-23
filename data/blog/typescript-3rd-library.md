---
title: 타사 라이브러리 및 TypeScript
date: '2022-12-23'
tags: ['typescript']
draft: false
summary: TypeScript와 함께 JavaScript(!) 라이브러리 사용하기, declare를 Last Resort로 사용하기, 클래스 변환자 - class-transformer, 클래스 검증자 - class-validation
layout: PostSimple
authors: ['default']
---

# 타사 라이브러리 및 TypeScript

> \***\*Don't reinvent the wheel(바퀴를 다시 발명하지 마라)\*\***

## TypeScript와 함께 JavaScript(!) 라이브러리 사용하기 - [Lodash](https://lodash.com/)

- 자바스크립트 유틸리티 라이브러리

### cdn 설치

```html
<script src="lodash.js"></script>
```

### npm 설치

```bash
npm i --save lodash
```

- 웹팩으로 동작하는 자바스크립트 프로젝트에서 사용

### 프로젝트 적용

```jsx
import _ from 'lodash'

console.log(_.shuffle([1, 2, 3]))
```

- 바닐라 자바스크립트로 빌드되었고, 바닐라 자바스크립트를 위해 빌드되었으로 타입스크립트로 위에 설치 명령으로 설치되면 컴파일 오류가 발생함
- 여러 종속성이 있는 `node_module` 폴더의 하위 폴더에 `lodash` 경로를 확인할 수 있음
  - 사용 가능한 자바스크립트 파일을 확인할 수 있음
- 타입스크립트가 패키지 내부를 이해하지 못하므로 별도의 설정이 필요함
  - 정확히 말하자면 `lodash`가 `export`하는 방식을 이해하지 못함

### 문제 해결

- `@types/lodash` 라고 불리는 npm 패키지를 설치

  ```bash
  npm i --save-dev @type/lodash
  ```

- [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/lodash)라는 깃허브 저장소에는 모든 종류의 써드파티 라이브러리 변환이 들어가 있는 대형 저장소로써, 하위의 lodash 폴더를 찾을 수 있음
- 실제 로직이 들어가 있지 않고, 타입스크립트에게 무언가 어떻게 동작하고 패키지 안에 어떤 것이 포함되어있는지에 대한 정보를 전달(`.d.ts` 파일 형식)
  - 자바스크립트를 타입스크립트로 변환

> 이처럼 변환 타입스크립트 패키지의 도움으로 보통 `@types/` 뒤에 패키지 이름이 오는 방식임

## declare를 Last Resort로 사용하기

- 대부분의 3rd 파티 라이브러리에는 `types` 패키지가 존재하지만, 그렇지 않은 경우에 `declare` 키워드를 사용해 특정 변수를 사용할 수 있음
- `declare var GLOBAL: any;`
  - 예를 들어 전역 변수를 선언한다고 가정했을 때, `declare` 키워드는 기본적으로 타입스크립트에게 반드시 존재함을 전달하는 것과 같음
- `declare` 키워드가 존재한다는 것은 타입스크립트의 특성 혹은 변수를 선언할 수 있게 하거나 타입스크립트는 일반적으로 알지 못하지만, 개발자는 확실이 어딘가에 존재하고 있는 것을 알고 있는 패키지나 전역 변수를 알려줌

## 클래스 변환자: [class-transformer](https://github.com/typestack/class-transformer)

### 설치

```bash
npm install class-transformer --save
npm install reflect-metadata --save
```

### `product.model.ts`

```tsx
export class Product {
  title: string
  price: number

  constructor(t: string, p: number) {
    this.title = t
    this.price = p
  }

  getInformation() {
    return [this.title, `$${this.price}`]
  }
}
```

- 제목과 가격을 가지는 `product` 모델 클래스
- 문자열과 가격을 입력하면 `getInformation`을 통해 배열 형태의 값을 반환

### `app.ts`

```tsx
import 'reflect-metadata'
import { plainToClass } from 'class-transformer'

import { Product } from './product.model'

const products = [
  { title: 'A Carpet', price: 29.99 },
  { title: 'A Book', price: 10.99 },
]

// const p1 = new Product('A Book', 12.99);

// const loadedProducts = products.map(prod => {
//   return new Product(prod.title, prod.price);
// });

const loadedProducts = plainToClass(Product, products)

for (const prod of loadedProducts) {
  console.log(prod.getInformation())
}
```

- 주석을 살펴보면 보통 `json` 형태로 특정 데이터(`products`) 형태를 서버로부터 받을 때를 가정하면, `map`을 사용해 데이터를 순회하고 모델의 인스턴스로 변환하는 작업을 수동으로 진행해야 함
- 클래스 변환기(`transformer`)를 사용해 첫번째 인자로는 인스턴스로 변환하고자 하는 모델, 두번째 인자로는 `json` 형태로 전달받은 데이터 즉, 변환할 데이터를 인자로 전달하면 자동으로 변환해줌

> 주로 타입스크립트를 대상으로 개발되었지만, 실질적으로 바닐라 자바스크립트에서도 동작하므로 타입스크립트 지정 기능을 사용하지는 않음

## 클래스 검증자: [class-validation](https://github.com/typestack/class-validator#readme)

- 타입스크립트 데코레이터의 개념 상에서 빌드되며, 클래스 내의 데코레이터의 도움을 받아 검증(`validation`), 규칙(`rule`) 추가하게 해줌

### 설치

```bash
npm install class-validator --save
```

### `product.model.ts`

```tsx
import { IsNotEmpty, IsNumber, IsPositive } from 'class-validator'

export class Product {
  @IsNotEmpty()
  title: string
  @IsNumber()
  @IsPositive()
  price: number

  constructor(t: string, p: number) {
    this.title = t
    this.price = p
  }

  getInformation() {
    return [this.title, `$${this.price}`]
  }
}
```

- 데코레이터 개념은 패키지 없이 맨 처음부터 개발된 개념으로써 해당 패키지를 사용하면 자체적으로 빌드하지 않아도 될 정도로 정교함
- 특정 데코레이터를 import해서 값에 대해 검증하고 하는 속성을 추가할 수 있음
  - `isNotEmpty` - 필수 값 검증
  - `IsNumber` - 숫자 검증
  - `IsPositive` - 양수 값 검증
- 데코레이터를 사용하기 위해 선행작업이 필요함(`tsconfig.json`)

  ```json
  {
    //...
    // Experimental Options
    "experimentalDecorators": true
  }
  ```

  - `experimentalDecorators` 값을 `true`로 지정해야 함

### `app.ts`

```tsx
import 'reflect-metadata'
import { plainToClass } from 'class-transformer'
import { validate } from 'class-validator'

import { Product } from './product.model'

const products = [
  { title: 'A Carpet', price: 29.99 },
  { title: 'A Book', price: 10.99 },
]

const newProd = new Product('', -5.99)
validate(newProd).then((errors) => {
  if (errors.length > 0) {
    console.log('VALIDATION ERRORS!')
    console.log(errors)
  } else {
    console.log(newProd.getInformation())
  }
})

// const p1 = new Product('A Book', 12.99);

// const loadedProducts = products.map(prod => {
//   return new Product(prod.title, prod.price);
// });

const loadedProducts = plainToClass(Product, products)

for (const prod of loadedProducts) {
  console.log(prod.getInformation())
}
```

- 제목이 빈 문자열이 되면 안되고, 숫자는 양수가 되어야 하는 조건에 부합하지 않는 검증할 수 있는 값을 인스턴스화함
- `validate` 메서드를 사용해 `async / await` 키워드를 사용해 `error` 배열의 길이에 따라 검증
- 에러가 있더라도 `then`을 종료하므로 캐치 블럭(`catch block`)으로 가져오지 않음
  - 즉, 항상 `then` 블럭으로 감

> **Referenced**
- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
