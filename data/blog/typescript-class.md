---
title: 클래스
date: '2022-08-07'
tags: ['typescript']
draft: false
summary: 앱이나 애플리케이션 로직의 일부를 관리하는 상태와 행위를 가진 객체로 분할하고 클래스라는 개념을 이용해 객체의 청사진(이론적 개념)을 만듦
layout: PostSimple
authors: ['default']
---

# 클래스

## 클래스 및 인스턴스

- 앱이나 애플리케이션 로직의 일부를 관리하는 상태와 행위를 가진 `객체`로 분할하고 `클래스`라는 개념을 이용해 `객체의 청사진`(이론적 개념)을 만듦

### 클래스

- `객체`의 형태, 포함해야 할 `속성`과 `메소드`를 정의하는 데 도움이 됨
- `객체`의 생성 속도를 높여주며, `객체 리터럴 표기법`을 사용하는 것에 대한 대안
- `클래스`를 사용하여 동일한 구조와 메소드를 포함한 여러 객체를 쉽게 만들 수 있음

### 객체

- `객체`는 코드로 작업을 수행하면서 사용할 수 있는 구체적인 요소들, 데이터를 저장하고 메소드를 실행하기 위해 메소드를 저장하는 데 사용하는 데이터 구조
- `객체`의 형태, 포함해야 하는 데이터, 클래스를 기반으로 객체를 쉽게 만들 수 있으려면 어떤 메소드가 필요한지 정의할 수 있기 때문에 이를 클래스 내의 `인스턴스`라고 부름

## 퍼스트 클래스 만들기

```tsx
class Department {
  name: string

  constructor(n: string) {
    this.name = n
  }
}
```

- `Department`라는 이름의 클래스를 생성
  - 관례상 클래스를 명시하기 위해 첫 글자는 대문자로 입력
- 클래스 블록 스코프 안에 `속성`과 `생성자 메소드`를 정의
  - `속성`
    - 언뜻 객체 리터럴과 같아 보이지만, 클래스에서는 `const`와 `let` 같은 변수 키워드를 사용하지 않고 키 이름만 정의
  - `생성자 메소드`
    - 타입스크립트 뿐만 아니라, 자바스크립트에도 인식하는 예약어
    - 객체에 대한 초기화와 this 키워드로 필드의 속성에 접근할 수 있음

### 객체 생성

```tsx
const accounting = new Department('Accounting')
console.log(accounting)
```

- `new` 키워드를 통해 생성할 객체의 `이름`, `생성자 인수`를 전달하여 객체를 생성하고 `const` 변수를 선언하여 인스턴스화

## 자바스크립트로 컴파일하기

`app.ts`

```tsx
class Department {
  name: string

  constructor(n: string) {
    this.name = n
  }
}

const accounting = new Department('Accounting')
console.log(accounting)
```

- 타입스크립트로 작성된 클래스 정의 방식의 소스코드는 `dist/app.js` 엔트리 포인트 경로에 컴파일된 자바스크립트의 소스코드와 차이가 있음

  - `dist/app.js`(ES6 옵션으로 컴파일)

    ```tsx
    'use strict'
    class Department {
      constructor(n) {
        this.name = n
      }
    }

    const accounting = new Department('Accounting')
    console.log(accounting)
    ```

    - 본질적으로는 같은 코드지만, 타입스크립트로 작성된 코드와는 다르게 name 필드는 클래스 코드 블록에 존재하지 않음
      - ES6에서는 name 필드와 타입 배정도 사라졌음
    - 최신 자바스크립트에서는 지원되나 버전에 따라 상이한 버전의 코드를 볼 수 있음

  - `dist/app.js`(ES5 옵션으로 컴파일)

    ```tsx
    'use strict'
    var Department = (function () {
      function Department(n) {
        this.name = n
      }
      return Department
    })()
    var accounting = new Department('Accounting')
    console.log(accounting)
    ```

    - 생성자 메소드가 아닌 내장된 `생성자 함수`를 통해 자바스크립트로 객체 청사진을 만드는 방식

- 이처럼 객체에 대한 `청사진`을 만드는 아이디어는 아주 오랫동안 자바스크립트에서 고민해왔고, 이어서 최신 자바스크립트에서는 보다 깔끔한 구문의 `클래스` 개념을 도입함
- 타입스크립트의 강력한 `컴파일` 방식을 지원함으로써 객체의 청사진을 정의할 수 있음

## 생성자 함수 및 this 키워드

- 위에서 본 바와 같이 과거에는 생성자 함수만 있었지만, 최근에 도입된 클래스 개념에서는 `문법적 설탕(Syntax sugar)`을 추가해 초기화 코드를 실행하는 데 `생성자 함수를 추가` 할 수 있음

```tsx
class Department {
  name: string

  constructor(n: string) {
    this.name = n
  }

  describe() {
    console.log('Department: ' + this.name)
  }
}

const accounting = new Department('Accounting')

accounting.describe()
```

- 생성자와 마찬가지로 this 키워드를 사용해 `상위 쿨래스의 필`드를 참조
  - 여기서 `this`는 일반적으로 생성된 클래스의 구체적인 `인스턴스`를 참조함

### this 키워드의 제한적인 사용

```tsx
class Department {
  name: string

  constructor(n: string) {
    this.name = n
  }

  describe() {
    console.log('Department: ' + this.name)
  }
}

const accounting = new Department('Accounting')

accounting.describe()

const accountingCopy = { describe: accounting.describe }

accountingCopy.describe() // Department: undefined
```

- 예를 들어 `accoutingCopy`와 같이 객체를 하나 더 추가한 후 describe 메소드로서 호출하면 컴파일 에러는 발생하지 않지만 런타임에서는 `Department: undefined`가 출력
  - `accountingCopy`에 대입한 객체는 `객체 리터럴`로 생성됐으므로 클래스를 기반으로 하지않고 단지 `더미 객체로서 생성`됨
  - describe 속성의 값은 `describe 메서드를 지시`하므로, 함수 실행 값으로 전달하지 않고 `함수 자체를 전달함`
  - 메소드가 실행됨에 따라 this가 `객체를 참조`하지 않으므로 문제가 발생

### this 매개변수에 타입 배정

```tsx
class Department {
  name: string

  constructor(n: string) {
    this.name = n
  }

  describe(this: Department) {
    console.log('Department: ' + this.name)
  }
}

const accounting = new Department('Accounting')

accounting.describe()

const accountingCopy = { name: 'Dummy', describe: accounting.describe }

accountingCopy.describe() // Department: undefined
```

- 이런 문제를 해결하기 위해 `this`를 호출하는 `describe` 메소드에 `매개변수`를 추가
  - 타입스크립트는 `this`가 무엇으로 참조되어야 하는지 인식하기 위해, `describe`가 실행될 때 `this`가 `Department` 클래스에 기반한 인스턴스 참조해야 한다는 것을 지정해줘야 함
- 따라서 `this`는 이름 속성을 가진 `Department`를 기반으로 하기 때문에, `name` 속성을 추가해야 한다는 컴파일 에러를 참고하여 문제를 해결할 수 있음

## private, public 접근 제어자

```tsx
class Department {
  public name: string;
  private employees string[] = [];

  constructor(n: string) {
    this.name = n;
  }

  describe(this: Department) {
    console.log('Department: ' + this.name);
  }

  addEmployee(employee: string) {
    this.employees.push(employee);
  }

  printEmployeeInformation() {
    console.log(this.employees.length);
    console.log(this.employees);
  }
}

const accounting = new Department('Accounting');

accounting.addEmployee('Max');
accounting.addEmployee('Manu');

accounting.describe();
accounting.printEmployeeInformation();
```

- 위 코드에서의 `private`의 의미는 `employees`가 생성된 객체 외부가 아닌 내부에서만 접근할 수 있는 속성으로써, 객체 외부에서 값을 함부로 변경하는 것을 방지함
  - `private` 키워드를 사용해 `addEmployee` 메소드를 통해서만 값을 변경할 수 있게 `제어`함
- `public` 키워드는 기본값과 같아서 name 앞에 붙이지 않아도 자동으로 적용됨
  - `private` 키워드와 달리 외부에서 값을 할당하여 `변경`할 수 있음
- `타입스크립트`에서만 동작하는 기능으로써, 컴파일을 수행하는 버전에 따라 해당 키워드(`private`)를 인식하지 못할 수 있고 `컴파일` 도중에 검사를 수행하며 `런타임` 도중에는 작동하지 않음

## 약식 초기화

```tsx
class Department {
  // private id: string;
  // public name: string;
  private employees string[] = [];

  constructor(private id: string, public name: string) {
    // this.id = id;
    // this.name = name;
  }

  describe(this: Department) {
    console.log(`Department (${this.id}): ${this.name}`);
  }

  addEmployee(employee: string) {
    this.employees.push(employee);
  }

  printEmployeeInformation() {
    console.log(this.employees.length);
    console.log(this.employees);
  }
}

const accounting = new Department('d01', 'Accounting');

accounting.addEmployee('Max');
accounting.addEmployee('Manu');

accounting.describe();
accounting.printEmployeeInformation();
```

- 늘어나는 필드와 더불어 생성자 매개변수에 반복할 필요없이 `약식 초기화`를 통해서 코드의 양을 줄일 수 있음
  - 생성자 함수 내에서 초기화하면 `이중 초기화 코드를 방지`할 수 있고, 더불어 클래스에 대해 동일한 이름으로 속성을 만들고 싶다는 `명시적인 명령`을 타입스크립트에 알릴 수 있음
- 접근 제어자를 지닌 모든 인수에 대해 동일한 이름의 속성이 생성되고, 인수에 대한 값이 생성된 속성에 저장됨

## `읽기 전용` 속성

```tsx
class Department {
  // private readonly id: string;
  // public name: string;
  private employees string[] = [];

  constructor(private readonly id: string, public name: string) {
    // this.id = id;
    // this.name = name;
  }

  describe(this: Department) {
    console.log(`Department (${this.id}): ${this.name}`);
  }

  addEmployee(employee: string) {
    this.employees.push(employee);
  }

  printEmployeeInformation() {
    console.log(this.employees.length);
    console.log(this.employees);
  }
}

const accounting = new Department('d01', 'Accounting');

accounting.addEmployee('Max');
accounting.addEmployee('Manu');

accounting.describe();
accounting.printEmployeeInformation();
```

- 타입스크립트에서만 지원되는 기능으로써, 초기화 한 후에 `변경되면 안되는` 특정 필드가 있는 경우 주로 사용
  - `최초`의 생성자 함수에서 `초기화 한 후`에는 클래스 내 메소드에서 값을 할당해도 변경이 되지 않음
- private, public 접근 제어자 뒤에 `readonly`를 명시함

## 상속

```tsx
class ITDepartment extends Department {
  admins: string[]
  constructor(id: string, public admin: string[]) {
    super(id, 'IT')
    this.admins = admins
  }
}

const it = new ITDepartment('d01', ['Max'])

it.addEmployee('Max')
it.addEmployee('Manu')

it.describe()
it.name = 'NEW NAME'
it.printEmployeeInformation()
```

- Department(부서)의 세부 부서를 정의하기 위해서는 `상속`의 개념을 통해 상위 클래스의 생성자를 포함한 모든 것을 자동으로 가져올 수 있음
  - 중요한건 `하나의 클래스`에서만 상속할 수 있음
  - 하위 클래스를 인스턴스화할 때 이 생성자가 자동으로 사용됨
- 상속하는 클래스로 `super`를 추가하고 이를 함수처럼 실행해야 함
  - `super`는 기본 클래스의 생성자를 호출
  - 또, 상속받는 클래스에서 특수한 속성을 추가하여 구성할 수 있음

## 속성 및 protected 접근 제어자 재정의

```tsx
class AccountingDepartment extends Department {
  constructor(id: string, private reports: string[]) {
    super(id, 'Accounting')
  }

  addEmployee(name: string) {
    if (name === 'Max') {
      return
    }
    this.employee.push(name)
  }

  addReports(text: string) {
    this.reports.push(text)
  }

  printReports() {
    console.log(this.reports)
  }
}

const accounting = new AccountingDepartment('d2', [])

accounting.addReport('Something went wrong...')

accounting.printReports()
```

- `Department`를 확장한 `AccountingDepartment` 클래스를 정의
  - 상위의 속성을 상속받고 `addReport` 메서드를 통해 전달된 인수의 텍스트를 `reports` 문자 배열에 추가해서 `printReports` 메서드를 호출하여 메시지를 출력
- 기본 클래스의 메소드나 속성을 재정의 할 수 있도록 지원
  - Accoungting 코드 블록에 `addEmployee` 메서드를 재정의
  - `employee`의 필드는 상위 `Department`에서 상속받지만 `private` 접근 제어자로 선언되었으므로, `protected` 키워드로 변경하여 `Department` 클래스를 확장하는 모든 클래스에서 사용 가능하도록 변경해야 함
- `확장한 클래스`에서는 기본 클래스의 메소드를 무시할 수 있으며, 클래스의 `고유한 구현`을 추가할 수 있음

## getter & setter

- 바닐라 자바스크립트에서도 지원하고 타입스크립트도 지원하는 기능

```tsx
class AccountingDepartment extends Department {
  private lastReport: string

  get mostRecentReport() {
    if (this.lastReport) {
      return this.lastReport
    }
    throw new Error('No report found.')
  }

  // ...
}

const accounting = new AccountingDepartment('d2', [])
// accounting.addReport('Something went wrong...');
```

- 속성이 `private`이고 타입이 문자열인 `lastReport`를 선언하고 이를 `reports` 속성으로 초기화
- `lastReport`는 `private`으로 지정되어 있어서 메소드 내에서는 접근 가능하지만 점 표기법으로 접근하는 것이 불가능함

  - `get` 키워드를 통해 `getter`를 생성하여 값을 가지고 올 때 함수나 메소드를 실행하는 속성을 추가함
    - `getter` 메소드는 꼭 무언가를 반환해야하므로, `this.lastReport`를 반환하여 캡슐화함
    - `if`문을 작성하여 값이 참인지 확인할 수 있으며, 정의되어 있다면 `lastReport`를 반환하고 그렇지 않다면 다른 로직이 반환되도록 해줌
  - `getter` 메소드는 실행하는 게 아닌 일반 속성처럼 접근해야 함

    ```tsx
    // O
    console.log(accounting.mostRecentReport)

    // X
    console.log(accounting.mostRecentReport())
    ```

- `set` 키워드를 통해 값을 읽고 설정하는 데 사용할 수 있음(`setter`)

  ```tsx
  class AccountingDepartment extends Department {
    private lastReport: string
    // getter ...
    // setter
    set mostRecentReport(value: string) {
      if (!value) {
        throw new Error('Please pass in a valid value!')
      }
      this.addReport(value)
    }
    // ...
  }

  const accounting = new AccountingDepartment('d2', [])
  ```

  ```tsx
  accounting.mostRecentReport = 'Year End Report'
  ```

  - `Year End Report`를 입력하고 저장하면 제대로 작동하여 setter 메소드를 통해 `reports` 목록에 포함됨

- `Getter` 메소드와 `Setter` 메소드를 통해 로직을 캡슐화하고 속성을 읽거나 설정하려 할 때 실행되어야 하는 추가적인 로직을 추가하는데 유용함

## 정적 메서드 & 속성

- ES6 이상의 자바스크립트와 타입스크립트에도 있는 기능으로써, 정적 속성과 메소드를 사용해 클래스의 인스턴스에 접근할 수 없는 속성과 메소드를 클래스에 추가할 수 있음

  - 클래스 이름을 호출하지 않고 직접 접근이 가능함

    ```tsx
    class Department {
      static fiscalYear = 2020;
      protected employees string[] = [];

      constructor(private readonly id: string, public name: string) {
        // console.log(this.fiscalYear);
        console.log(Department.fiscalYear);
      }

      static createEmployee(name: string){
        return { name: name };
      }
      //...
    }

    const employee1 = Department.createEmployee('Max');
    console.log(employee1, Department.fiscalYear);
    ```

- 클래스를 그룹화 메커니즘으로 사용할 수 있으며, 필드 또한 `static` 키워드를 통해 정적 속성을 추가할 수 있음
- `static`으로 지정된 정적 멤버(속성) 혹은 메서드는 접근할 수 없음
  - 정적 속성과 정적 메소드의 전체적인 개념은 `인스턴스`와 분리되어 있으므로 인스턴스화해서 사용할 수 없거나, `this` 키워드를 통해 접근할 수 없음
  - 클래스 내부에서 접근하려면 `Department.fiscalYear`과 같이 접근할 수 있음

## 추상 클래스

- 특정 클래스를 사용하여 작업하거나 특정 클래스를 확장시키는 작업을 할 경우, 특정 메소드를 구현하거나 `재정의`하도록 해야 하는 경우가 있음
- 빈 메소드를 기본 클래스(`Department`)에 입력하고 기본 클래스 기반의 확장된 클래스를 메소드(`describe`)에 추가하고 재정의하게 할 수 있음

  ```tsx
  abstract class Department {
    //...

    abstract describe(this: Department): void

    //...
  }

  const employee1 = Department.createEmployee('Max')
  console.log(employee1, Department.fiscalYear)
  ```

  - 클래스 내에 추상 메소드 키워드인 `abstract`를 추가하려면 클래스도 마찬가지로 `abstract` 키워드를 명시해야 함
    - 중괄호 쌍을 제거하고 쌍점을 추가하여 보유해야 하는 타입을 추가하지 않고 반환하도록 해야함
    - 이 메소드의 형태와 메소드의 구조가 어떤 것인지를 정의할 수 있음
  - 확장된 클래스에서는 반드시 `abstract` 키워드로 선언된 메소드를 재정의 해야함

## 싱글톤 & private 생성자

### 싱글톤

- 객체 지향 프로그래밍의 디자인 패턴으로써, 특정 클래스의 인스턴스를 정확히 `하나`만 갖도록 하는 패턴
  - 클래스를 기반으로 여러 객체를 만들 수는 없지만 항상 클래스를 기반으로 정확히 하나의 객체만 가질수 있도록 하는 경우에 유용할 수 있음

```tsx
class AccountingDepartment extends Department {

  private static instance: AccountingDepartment;
  // ...

  private constructor(id: string, private reports: string[]){
    super(id, 'Accounting');
  }

  static getInstance() {
    if (AccountingDepartment.instance) {
      return this.instance;
    }
    this.instance = new AccountingDepartment('d2'. []);
    return this.instance;
  }

  // ...
}

const accounting1 = AccountingDepartment.getInstance();
const accounting2 = AccountingDepartment.getInstance();

console.log(accounting, accounting2);
```

- 회계 부서는 하나로 구현해야 할 때, `new AccountingDepartment`를 여러 번 수동으로 호출하지 않도록 하기 위해서 `private` 키워드를 생성자 앞에 붙여서 지정할 수 있음
  - 클래스 내에서만 접근할 수 있는 `instance`, 즉 `AccountingDepartment` 인스턴스가 저장되도록 함
  - 여기서 저장하는 값은 클래스(`AccountingDepartment`) 자체가 됨
- if 문으로 인스턴스가 없는 경우에는 새로 인스턴스화하고, 이미 인스턴스가 존재하면 해당 인스턴스를 반환하는 로직을 구현할 수 있음

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
