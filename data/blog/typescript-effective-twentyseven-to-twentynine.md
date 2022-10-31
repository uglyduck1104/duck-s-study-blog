---
title: 타입스크립트의 타입 시스템 - Item 27 ~ 29
date: '2022-10-31'
tags: ['typescript']
draft: false
summary: 함수형 기법과 라이브러리로 타입 흐름 유지하기, 유효한 상태만 표현하는 타입을 지향하기, 사용할 때는 너그럽게, 생성할 때는 엄격하게
layout: PostSimple
authors: ['default']
---

# 타입스크립트의 타입 시스템 - Item 27 ~ 29

## Item 27) 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 파이썬, C, 자바 등에서 볼 수 있는 표준 라이브러리가 자바스크립트에는 되어 있지 않음
- 수년간 많은 라이브러리는 표준 라이브러리의 역할을 대신하기 위해 노력해왔고 람다나 로대시같은 최근의 라이브러리는 함수형 프로그래밍의 개념을 자바스크립트 세계에 도입해 옴
- 이러한 라이브러리는 순수 자바스크립트로 구현되어 있으며 타입스크립트와 조합하여 사용하면 빛을 발하는 데, 그 이유는 타입 정보가 그대로 유지되면서 타입 흐름이 계속 전달되도록 하기 때문임
- 로대시와 같은 라이브러리를 쓸때는 신중해야 함
  - 서드파티 라이브러리 기반으로 코드를 짧게 줄이는 데 시간이 많이 든다면, 서드파티 라이브러리를 사용하지 않는 게 나음
  - 하지만, 타입스크립트로 작성하면 타입 정보를 참고하며 작업할 수 있기 때문에 서드파티 라이브러리 기반으로 바꾸는 데 시간이 훨씬 단축되므로, 사용하는 게 유리함

### 장점

- 데이터 가공이 정교해질수록 장점은 더욱 분명해짐
- 모든 NBA 팀의 선수 명단을 가지고 있다고 가정해보면 아래와 같음

  ```tsx
  interface BasketballPlayer {
    name: string
    team: string
    salary: number
  }
  declare const rosters: { [team: string]: BasketballPlayer[] }
  ```

  - 루프를 사용해 단순(flat) 목록을 만드려면 배열에 concat을 사용해야 함

    ```tsx
    const allPlayers = Object.values(roasters).flat()
    // 정상, 타입이 BasketballPlayer[]
    ```

  - flat 메서드는 다차원 배열을 평탄화 해줌

- allPlayer를 가지고 각 팀별로 연봉 순으로 정렬해서 최고 연봉 선수의 명단을 만든다고 가정

  ```tsx
  const teamToPlayers: { [team: string]: BasketballPlayer[] } = {}
  for (const player of allPlayers) {
    const { team } = player
    teamToPlayers[team] = teamToPlayers[team] || []
    teamToPlayers[team].push(player)
  }

  for (const players of Object.values(teamToPlayers)) {
    players.sort((a, b) => b.salary - a.salary)
  }

  const bestPaid = Object.values(teamToPlayers).map((players) => players[0])
  bestPaid.sort((playerA, playerB) => playerB.salary - playerA.salary)
  console.log(bestPaid)
  ```

  - 결과

    ```tsx
    ;[
      { team: 'GSW', salary: 37457154, name: 'Stephen Curry' },
      { team: 'HOU', salary: 35654150, name: 'Chris Paul' },
      { team: 'LAL', salary: 35654150, name: 'LeBron James' },
      { team: 'OKC', salary: 35654150, name: 'Russel Westbrook' },
      { team: 'DET', salary: 32088932, name: 'Blake Griffin' },
      // ...
    ]
    ```

## Item 28) 유효한 상태만 표현하는 타입을 지향하기

- 타입을 잘 설계하면 코드는 직관적으로 작성할 수 있으나 설계가 엉망이라면 어떠한 기억이나 문서도 도움되지 못함
- 효과적으로 타입을 설계하려면, 유효한 상태만 표현할 수 있는 타입을 만들어 내는 것이 가장 중요함

### 개선 전

```tsx
interface State {
  pageText: string
  isLoading: boolean
  error?: string
}
```

- 페이지를 그리는 `renderPage` 함수를 작성할 때는 상태 객체의 필드를 전부 고려해서 상태 표시를 분기해야 함

  ```tsx
  function renderPage(state: State) {
    if (state.error) {
      return `Error! Unable to load ${currentPage}: ${state.error}`
    } else if (state.isLoading) {
      return `Loading ${currentPage}...`
    }
    return `<h1>${currentPage}</h1>\n${state.pageText}`
  }
  ```

  - `isLoading`이 `true`이고 동시에 `error` 값이 존재하면 로딩 중인 상태인지 오류가 발생한 상태인지 명확히 구분할 수 없음

- 페이지를 전환하는 `changePage` 함수는 다음과 같음

  ```tsx
  async function chagePage(state: State, newPage: string) {
    state.isLoading = true
    try {
      const response = await fetch(getUrlForPage(newPage))
      if (!response.ok) {
        throw new Error(`Unable to load ${newPage}: ${response.statusText}`)
      }
      const text = await response.text()
      state.isLoading = false
      state.pageText = text
    } catch (e) {
      state.error = '' + e
    }
  }
  ```

  - 문제점 인지
    1. 오류가 발생했을 때 state.isLoading을 false로 설정하는 로직이 빠져 있음
    2. state.error를 초기화하지 않았기 떄문에, 페이지 전환 중에 로딩 메시지 대신 과거 오류 메시지를 보여 주게 됨
    3. 페이지 로딩 중에 사용자 페이지가 바뀌어 버리면 어떤 일이 벌어질지 예상하기 어려움
  - 상태 값의 두 가지 속성이 동시에 정보가 부족하거나, 충돌할 수 있는 위험이 있음

### 개선 후

```tsx
interface RequestPending {
  state: 'pending'
}
interface RequestError {
  state: 'error'
  error: string
}
interface RequestSuccess {
  state: 'ok'
  pageText: string
}
type RequestState = RequestPending | RequestError | RequestSuccess

interface State {
  currentPage: string
  requests: { [page: string]: RequestState }
}
```

- 네트워크 요청 과정 각각의 상태를 명시적으로 모델링하는 태그된 유니온이 사용됨으로써 상태를 나타내는 코드 길이가 길어졌으나 무효한 상태를 허용하지 않도록 크게 개선됨
- 그 결과로 개선된 renderPage와 changePage 함수는 쉽게 구현할 수 있음
- `renderPage`

  ```tsx
  function renderPage(state: State) {
    const { currentPage } = state
    const requestState = state.request[currentPage]
    switch (requestState.state) {
      case 'pending':
        return `Loading ${currentPage}...`
      case 'error':
        return `Error! Unable to load ${currentPage}: ${requestState.erorr}`
      case 'ok':
        return `<h1>${currentPage}</h1>\n${requestState.pageText}`
    }
  }
  ```

- `changePage`

  ```tsx
  async function changePage(state: State, newPage: string) {
    state.request[newPage] = { state: 'pending' }
    state.currentPage = newPage
    try {
      const response = await fetch(getUrlForPage(newPage))
      if (!response.ok) {
        throw new Error(`Unable to load ${newPage}: ${response.statusText}`)
      }
      const pageText = await response.text()
      state.requests[newPage] = { state: 'ok', pageText }
    } catch (e) {
      state.requests[newPage] = { state: 'error', error: '' + e }
    }
  }
  ```

### 정리

- 현재 페이지가 무엇인지 명확하며, 모든 요청은 정확히 하나의 상태로 맞아 떨어짐
- 무효가 된 요청이 실행되긴 하겠지만 UI에는 영향을 미치지 않음

## Item 29) 사용할 때는 너그럽게, 생성할 때는 엄격하게

- 함수의 매개변수는 타입의 범위가 넓어도 되지만, 결과를 반환할 때는 일반적으로 타입의 범위가 더 구체적이어야 함

### 예시 - 3D 매핑 API

```tsx
declare function setCamera(camera: CameraOptions): void
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions
```

- 카메라의 위치를 지정하고 경계 박스의 뷰포트를 계산하는 예시
- 카메라의 위치를 잡기 위해 `viewportForBounds`의 결과가 `setCamera`로 바로 전달될 수 있다면 편함

```tsx
interface CameraOptions {
  center?: LngLat
  zoom?: number
  bearing?: number
  pitch?: number
}
type LngLat = { lng: number; lat: number } | { lon: number; lat: number } | [number, number]
```

- 일부 값은 건드리지 않으면서 동시에 다른 값을 설정할 수 있어야 하므로 `CameraOptions`의 필드는 모두 선택적임
- 유사하게 `LngLat` 타입도 `setCamera`의 매개변수 범위를 넓혀줌

```tsx
type LngLatBounds =
  | { northeast: LngLat; southwest: LngLat }
  | [LngLat, LngLat]
  | [number, number, number, number]
```

- 이름이 주어진 모서리, 위도/경도 쌍, 또는 순서가 맞다면 4-튜플을 사용하여 경계를 지정할 수 있음
- `LngLat`는 세 가지 형태를 받을 수 있으므로, `LngLatBounds`의 가능한 형태는 19가지 이상임
- `GeoJSON` 기능을 지원하도록 뷰표트를 조절하고, 새 뷰포트를 `URL`에 저장하는 함수를 작성

### 문제 인식

```tsx
function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f)
  const camera = viewportForBounds(bounds)
  setCamera(camera)
  const {
    center: { lat, lng },
    zoom,
  } = camera
  // ... 형식에 'lat' 속성이 없습니다.
  // ... 형식에 'lng' 속성이 없습니다.
  zoom // 타입이 number | undefined
  window.location.search = `?v=@${lat},${lng}z${zoom}`
}
```

- 예제의 오류는 `lat`과 `lng` 속성이 없고 `zoom` 속성만 존재하기 때문에 발생했지만, 타입이 `number | undefined`로 추론되는 것 역시 문제임
- `camera` 값을 안전한 타입으로 사용하는 유일한 방법은 유니온 타입의 각 요소별로 코드를 분기해야 함

> 매개변수 타입의 범위가 넓으면 사용하기 편리하지만, 반환 타입의 범위가 넓으면 불편함

### 문제 개선

```tsx
interface LngLat {
  lng: number
  lat: number
}
type LngLatLike = LngLat | { lon: number; lat: number } | [number, number]

interface Camera {
  center: LngLat
  zoom: number
  bearing: number
  pitch: number
}
interface CameraOptions extends Omit<Partial<Camera>, 'center'> {
  center?: LngLatLike
}
type LngLatBounds =
  | { northeast: LngLatLike; southwest: LngLatLike }
  | [LngLatLike, LngLatLike]
  | [number, number, number, number]

declare function setCamera(camera: CameraOptions): void
declare function viewportForBounds(bounds: LngLatBounds): Camera
```

- `setCamera` 매개변수 타입의 `center` 속성에 `LngLatLike` 객체를 허용해야 하기 때문에 `Partial<Camera>`를 사용하면 코드가 동작하지 않음
- `LngLatLike`가 `LngLat`의 부분 집합이 아니라 상위집합이므로 `CameraOptions extend Partial<Camera>`를 사용할 수 없음

> 명시적으로 타입을 추출해 다음처럼 작성할 수 있음

```tsx
interface CameraOptions {
  center?: LngLatLike
  zoom?: number
  bearing?: number
  pitch?: number
}
```

- 앞에서 설명한 `CameraOptions`를 선언하는 두 가지 방식 모두 `focusOnFeature` 함수가 타입 체커를 통과할 수 있게 함

```tsx
function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f)
  const camera = viewportForBounds(bounds)
  setCamera(camera)
  const {
    center: { lat, lng },
    zoom,
  } = camera // 정상
  zoom // 타입이 number
  window.location.search = `?v=@${lat},${lng}z${zoom}`
}
```

- `zoom`의 타입이 `number|undefined`가 아니라 `number`이므로 `viewportForBounds` 함수를 사용하기 훨씬 쉬워짐

### 정리

- 앞에 예시처럼 경계 박스의 형태를 19가지나 허용하는 것은 좋은 설계가 아님

> **Referenced**

- 댄 밴더캄, 『이펙티브 타입스크립트』, 인사이트(2021.11.4), 147 ~ 166p
