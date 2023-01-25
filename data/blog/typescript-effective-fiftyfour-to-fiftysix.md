---
title: 코드를 작성하고 실행하기 - Item 54 ~ 56
date: '2023-01-25'
tags: ['typescript']
draft: false
summary: 객체를 순회하는 노하우, DOM 계층 구조 이해하기, 정보를 감추는 목적으로 private 사용하지 않기
layout: PostSimple
authors: ['default']
---

# 코드를 작성하고 실행하기 - Item 54 ~ 56

## Item 54) 객체를 순회하는 노하우

### 예제 1)

```tsx
const obj = {
  one: 'uno',
  two: 'dos',
  three: 'tres',
}
for (const k in obj) {
  const v = obj[k]
  // obj에 인덱스 시그니처가 없기 때문에
  // 엘리먼트는 암묵적으로 'any' 타입입니다.
}
```

- obj 객체를 순회하는 루프 내의 상수 k와 관련된 오류

```tsx
const obj = { /* ... */ }'
// const obj = {
//	 one: 'uno',
//	 two: 'dos',
//	 three: 'tres',
// };
for (const k in obj) { // const k: string
	// ...
}
```

- `k`의 타입은 `string`인 반면, `obj` 객체에는 `one`, `two`, `three` 세 개의 키만 존재하므로 `k`와 `obj` 객체의 키 타입이 서로 다르게 추론되어 오류가 발생함

### 해결 1)

```tsx
let k: keyof typeof obj // "one" | "two" | "three" 타입
for (k in obj) {
  const v = obj[k] // 정상
}
```

- k의 타입을 더욱 구체적으로 명시

### 예제 2)

```tsx
const x = { a: 'a', b: 'b', c: 2, d: new Date() }
foo(x) // 정상
```

- `foo` 함수는 `a, b, c` 속성 외에 `d`를 가지는 `x` 객체로 호출이 가능함
- `foo` 함수는 `ABC` 타입에 할당 가능한 어떠한 값이든 매개변수로 허용함
- 즉, ABC 타입에 할당 가능한 객체는 a, b, c외에 다른 속성이 존재할 수 있으므로 타입스크립트는 ABC 타입의 키를 string 타입으로 선택해야 함

### keyof 키워드를 사용한 문제점

```tsx
function foo(abc: ABC) {
  let k: keyof ABC
  for (k in abc) {
    // let k: "a" | "b" | "c"
    const v = abc[k] // string | number 타입
  }
}
```

- `k`가 `a | b | c` 타입으로 한정되어 문제가 된 것처럼, `v`도 `string | number` 타입으로 한정되어 범위가 너무 좁아 문제가 됨
- `d: new Date()`가 있는 예제처럼, `d` 속성은 `Date` 타입뿐만 아니라 어떠한 타입이든 될 수 있으므로 `v`가 `string | number` 타입으로 추론된 것은 잘못이며 런타임의 동작을 예상하기 어려움

### Object.entries를 사용한 해결

```tsx
function foo(abc: ABC) {
  for (const [k, v] of Object.entries(abc)) {
    k // string 타입
    v // any 타입
  }
}
```

- Object.entries를 사용한 루프가 직관적이지는 않으나 복잡한 기교 없이 사용할 수 있음

### 프로토타입 오염의 위험성

- for-in 구문을 사용하면, 객체의 정의에 없는 속성이 갑자기 등장함

  ```
  > Object.prototype.z = 3;
  > const obj = {x: 1, y: 2};
  > for (const k in obj) { console.log(k); }
  x
  y
  z
  ```

  - 실제 작업에서는 Object.prototype에 순회 가능한 속성을 절대로 추가하면 안됨
  - for-in 루프에서 k가 string 키를 가지게 된다면 프로토타입 오염의 가능성을 의심해봐야함

## Item 55) DOM 계층 구조 이해하기

- 타입스크립트에서는 `DOM` 엘리먼트의 계층 구조를 파악하기 용이함
  - `Element`와 `EventTaret`에 달려 있는 `Node`의 구체적인 타입을 안다면 타입 오류를 디버깅할 수 있고, 언제 타입 단언을 사용해야 할지 알 수 있음

### 예제 - div 경계를 넘어 마우스를 움직이는 경우를 추적

```tsx
function handleDrag(eDown: Event) {
  const targetEl = eDown.currentTarget
  targetEl.classList.add('dragging')
  // 개체가 'null'인 것 같습니다.
  // 'EventTarget' 형식에 'classList' 속성이 없습니다.
  const dragStart = [eDown.clientX, eDown.clientY]
  // 'Event' 형식에 'clientX' 속성이 없습니다.
  // 'Event' 형식에 'clientY' 속성이 없습니다.
  const handleUp = (eUp: Event) => {
    targetEl.classList.remove('dragging')
    // 개체가 'null' 인 것 같습니다.
    // 'EventTarget' 형식에 'classList' 속성이 없습니다.
    targetEl.removeEventListener('mouseup', handleUp)
    // 개체가 'null' 인 것 같습니다
    const dragEnd = [eUp.clientX, eUp.clientY]
    // 'Event' 형식에 'clientX' 속성이 없습니다.
    // 'Event' 형식에 'clientY' 속성이 없습니다.
    console.log(
      'dx, dy = ',
      [0, 1].map((i) => dragEnd[i] - dragStart[i])
    )
  }
  targetEl.addEventListener('mouseup', handleUp)
  // 개체가 'null' 인 것 같습니다
}
const div = document.getElementById('surface')
div.addEventListener('mousedown', handleDrag)
// 개체가 'null' 인 것 같습니다
```

- `EventTarget` 타입 오류가 일어나는 이유

  - DOM 계층 구조에서의 HTML 코드의 예제

    - `<p id="quote">and <i>yet</i> it moves</p>`

      - 브라우저에서 자바스크립트 콘솔을 열고 `p` 엘리먼트의 참조를 얻어 보면, `HTML` `ParagraphElement` 타입이라는 것을 알 수 있음

      ```tsx
      const p = document.getElementsByTagName('p')[0]
      p instanceof HTMLParagraphElement
      // 참(true)
      ```

    - `HTMLParagraphElement`는 `HTMLElement`의 서브타입이고, `HTMLElement`는 `Element`의 서브 타입임
    - `Element`는 `Node`의 서브타입이고, `Node`는 `EventTarget`의 서브타입임

| 타입                | 예시                         |
|-------------------|----------------------------|
| EventTarget       | window, XMLHttpRequest     |
| Node              | document, Text, Comment    |
| Element           | HTMLElement, SVGElement 포함 |
| HTMLElement       | `<i>`, `<b>`               |
| HTMLButtonElement | `<button> `                |

### EventTarget

- DOM 타입 중 가장 추상화된 타입
- 이벤트 리스너를 추가하거나 제거하고, 이벤트를 보내는 것 밖에 할 수 없음

```tsx
function handleDrag(eDown: Event) {
  const targetEl = eDown.currentTarget
  targetEl.classList.add('dragging')
  // 개체가 'null'인 것 같습니다.
  // 'EventTarget' 형식에 'classList' 속성이 없습니다.
  // ...
}
```

- 아까 오류난 부분을 보자면, `Event`의 `currentTarget` 속성의 타입은 `EventTarget | null`이므로 `null` 가능성이 오류로 표시되었고, 또한 `EventTarget` 타입에 `classList` 속성이 없기 때문에 오류로 표기됨
- `eDown.currentTarget`은 실제로 `HTML Element` 타입이지만, 타입 관점에서는 `window`나 `XMLHttpRequest`가 될 수도 있음

### Node

- Element가 아닌 Node인 경우를 몇가지 예로 들어 보면 텍스트 조각과 주석이 있음

```tsx
<p>
	And <i>yet</i> it moves
	<!-- quote from Galileo -->
</p>
```

- 가장 바깥쪽의 엘리먼트는 `HTMLParagraphElement`이고 `children`과 `childNodes` 속성을 가지고 있음

```
> p.children
HTMLCollection [i]
> p.childNodes
NodeList(5) [text, i, text, comment, text]
```

- `children`은 자식 엘리먼트`(<i>yet</i>)`를 포함하는 배열과 유사한 구조인 `HTMLCollection`인 반면, `childNodes`는 배열과 유사한 `Node`의 컬렉션인 `NodeList`임
- `childNodes`는 엘리먼트`(<i>yet</i>)`뿐만 아니라 텍스트 조각`("And", "it moves")`과 주석`("quote from Galileo")`까지도 포함하고 있음

### Element | HTMLElement

`Element`

- `SVG` 태그의 전체 계층 구조를 포함하면서 `HTML`이 아닌 엘리먼트가 존재하는데, 바로 `Element`의 또 다른 종류인 `SVGElement`임
  - `<html>`은 `HTMLHtmlElement`이고 `<svg>`는 `SVGSvgElement`임

`HTMLElement`

- 자신만의 고유한 속성을 가지고 있음
- `HTMLImageElement`에는 `src` 속성이 있고, `HTMLInputElement`에는 `value` 속성이 있음
  - 이런 속성에 접근하려면, 타입 정보 역시 실제 엘리먼트 타입이어야 하므로 상당히 구체적으로 타입을 지정해야 함
- 보통은 HTML 태그 값에 해당하는 button 같은 리터럴 값을 사용하여 DOM에 대한 정확한 타입을 얻을 수 있음

  ```tsx
  document.getElementsByTagName('p')[0] // HTMLParagraphElement
  document.createElement('button') // HTMLButtonElement
  document.querySelector('div') // HTMLDivElement
  ```

  - 그러나 항상 정확한 타입을 얻을 수 있는 것은 아니고 특히 `document.getElementById`에서 문제가 발생함

    ```tsx
    document.getElementById('my-div') // HTMLElement
    ```

  - 일반적으로 타입 단언문을 지양해야 하지만, `DOM` 관련해서는 타입스크립트보다 우리가 더 정확히 알고 있는 경우이므로 단언문을 사용해도 좋음

    ```tsx
    document.getElementById('my-div') as HTMLDivElement
    ```

- `strictNullCheck`가 설정된 상태라면 `null`인 경우를 체크해야 함

  - 실제 코드에서 document.getElementById가 null일 가능성이 있다면 if 분기문을 추가해야 함

    ```tsx
    const div = document.getElementById('my-div')!
    ```

### EventTarget Error Handling

- 아이템의 두 번째 예제로 `EventTarget` 이후에는 다음과 같은 오류가 발생함

  ```tsx
  function handleDrag(eDown: Event) {
    // ...
    const dragStart = [eDown.clientX, eDown.clientY]
    // 'Event'에 'clientX' 속성이 없습니다.
    // 'Event'에 'clientY' 속성이 없습니다.
    // ...
  }
  ```

  - `clientX`와 `clientY`에서 발생한 오류의 원인은, `handleDrag` 함수의 매개변수는 `Event`로 선언된 반면 `clientX`와 `clientY`는 보다 구체적인 `MouseEvent` 타입에 있기 때문임
    - `UIEvent` - 모든 종류이 사용자 인터페이스 이벤트
    - `MouseEvent` - 클릭처럼 마우스로부터 발생되는 이벤트
    - `TouchEvent` - 모바일 기기의 터치 이벤트
    - `WheelEvent` - 스크롤 휠을 돌려서 발생되는 이벤트
    - `KeyboardEvent` - 키 누름 이벤트

- DOM에 대한 타입 추론은 문맥 정보를 폭넓게 활용함

  - `mousedown` 이벤트 핸들러를 인라인 함수로 만들면 더 많은 문맥 정보를 사용할 수 있고 대부분의 오류를 제거할 수 있음

    ```tsx
    function addDragHandler(el: HTMLElement) {
    	el.addEventListener('mousedown', eDown => {
    		const dragStart = [eDown.clientX, eDown.clientY];
    		const handleUp = (eUp: MouseEvent) => {
    			el.classList.remove('dragging');
    			el.removeEventListener('mouseup', handleUp);
    			const dragEnd = [eUp.clientX, eUp.clientY];
    			console.log('dx, dy = ', [0, 1].map(i => dragEnd[i] - dragStart[i]));
    		}
    		el.addEventListner('mouseup', handleUp);
    	}
    }

    const div = document.getElementById('surface');
    if (div) {
    	addDragHandler(div);
    }
    ```

  - 코드 마지막의 `if` 구문은 `#surface` 엘리먼트가 없는 경우를 체크함

## Item 56) 정보를 감추는 목적으로 private 사용하지 않기

- 자바스크립트는 클래스에 비공개 속성을 만들 수 없기때문에 언더스코어(\_)를 접두사로 붙이던 것이 관례였음

  ```tsx
  class Foo {
    _private = 'secret123'
  }
  ```

- 그러나 언더스코어는 단순히 비공개라고 표시한 것 뿐, 일반적인 속성과 동일하게 클래스 외부로 공개됨

  ```tsx
  const f = new Foo()
  f._private // 'secret123'
  ```

- 타입스크립트에는 `public`, `protected`, `private` 접근 제어자를 사용해 공개 규칙을 강제할 수 있는 것처럼 오해 할 수 있음

  ```tsx
  class Diary {
    private secret = 'cheated on my English test'
  }

  const diary = new Diary()
  diary.secret
  // 'secret' 속성은 private이며
  // 'Diary' 클래스 내에서만 접근할 수 있음
  ```

  - 그러나 `public`, `protected`, `private` 같은 접근 제어자는 타입스크립트 키워드이므로 컴파일 후에 제거됨
  - 컴파일 코드

    ```tsx
    class Diary {
      constructor() {
        this.secret = 'cheated on my English test'
      }
    }
    const diary = new Diary()
    diary.secret
    ```

    - 타입스크립트의 접근제어자는 단지 컴파일 시점에만 오류를 표시해줄 뿐이며, 아무런 효력이 없음
    - 심지어 단언문을 사용하면 타입스크립트 상태에서도 `private` 속성에 접근 가능

- 정보를 은닉하기 위해서 private 키워드를 사용하면 안됨

  - 대신 클로저 함수를 이용해 원하고자하는 효과를 낼 수 있음

    ```tsx
    declare function hash(text: string): number

    class PasswordChecker {
      checkPassword: (password: string) => boolean
      constructor(passwordHash: number) {
        this.checkPassword = (password: string) => {
          return hash(password) === passwordHash
        }
      }
    }

    const checker = new PasswordChecker(hash('s3cret'))
    checker.checkPassword('s3cret') // 결과는 true
    ```

    - 생성자 외부에서 passwordHash 변수에 접근할 수 없으므로 정보를 숨기는 목적에는 달성했으나 몇가지 문제가 있음
      - passwordHash를 생성자 외부에서 접근할 수 없으므로, 메서드 정의가 생성자 내부에 존재하게 되면, 인스턴스를 생성할때마다 메서드의 복사본이 생기므로 메모리 낭비가 발생함
      - 동일한 클래스로부터 생성된 인스턴스라고해도 서로 비공개 데이터에 접근하는 것이 불가능함

  ### 비공개 필드 사용(#) - 표준화 진행중

  - 접두사로 `#`을 붙여서 타입 체크와 런타이 모두에서 비공개로 만드는 역할을 함

    ```tsx
    class PasswordChecker {
      #passwordHash: number

      constructor(passwordHash: number) {
        this.#passwordHash = passwordHash
      }

      checkPassword(password: string) {
        return hash(password) === this.#passwordHash
      }
    }

    const checker = new PasswordChecker(hash('s3cret'))
    checker.checkPassword('secret') // 결과는 false
    checker.checkPassword('s3cret') // 결과는 true
    ```

  - 2021년 기준으로 비공개 필드는 자바스크립트 표준화 3단계이며, 타입스크립트에서 사용이 가능함

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 268 ~ 281p
