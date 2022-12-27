---
title: 연습 프로젝트 - 장소 선택 및 공유 앱
date: '2022-12-27'
tags: ['typescript']
draft: false
summary: 프로젝트 설정, 구글 API 키 설정, 지도 렌더링
layout: PostSimple
authors: ['default']
---

# 연습 프로젝트 - 장소 선택 및 공유 앱

## 프로젝트 설정

- 주소를 입력하고, 주소를 검색한 후 한 쌍의 좌표로 변환
- 구글 API 중 하나인 Google Map API를 사용해 지도를 렌더링
- 구글맵 자바스크립트 SDK 사용

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Understanding TypeScript</title>
    <script src="dist/bundle.js" defer></script>
    <link rel="stylesheet" href="app.css" />
  </head>
  <body>
    <div id="map">
      <p>Please enter an address!</p>
    </div>
    <form>
      <input type="text" id="address" />
      <button type="submit">SEARCH ADDRESS</button>
    </form>
  </body>
</html>
```

- 지도를 렌더링할 영역을 `div` 태그에 `id`값이 `map`인 요소를 지정
- `form` 태그 안에 `input type`이 `text`인 필드를 만들어 `id`가 `address`인 요소를 지정하고 버튼을 누르면 양식이 제출되면서 한 쌍의 좌표값을 가진 지도를 렌더링하는 것이 목표

### `app.css`

```css
html {
  font-family: sans-serif;
}

#map {
  width: 90%;
  height: 20rem;
  border: 1px solid #ccc;
  margin: 2rem auto;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

form {
  text-align: center;
  margin: 2rem auto;
}
```

## 사용자 입력 받기

### `app.ts`

```tsx
const form = document.querySelector('form')!
const addressInput = document.getElementById('address')! as HTMLInputElement

function searchAddressHandler(event: Event) {
  event.preventDefault()
  const enteredAddress = addressInput.value

  // send this to Google's API!
}

form.addEventListener('submit', searchAddressHandler)
```

- `form` 태그를 `querySelector`를 통해 취득
- `input` 태그의 `id` 값이 `address`인 요소를 취득
  - 상위의 요소(`form`, `input`)가 항상 존재할 것을 타입스크립트에게 `!` 마크를 통해 널이 아님을 알려줌
  - 취득한 `input` 태그의 값을 가져오기 위해 단언문을 사용해 `HTMLInptElement` 타입 캐스팅 처리(`value` 속성을 사용하기 위함)
- 양식을 제출하면 `trigger`될 이벤트 리스너 함수를 구현
  - 매개변수의 `event`가 `Event` 타입임을 타입스크립트에게 알려줌
  - `event.preventDefault()` 명령어를 사용해 양식 제출을 방지하고 해당 요청을 전송하지 않게 일시적으로 막아둠
- 구글 API로 보내기 위한 주석 작성

## Google API 키 설정하기

1. 구글 계정 만들기
2. 구글 결제 계정 활성화(`신용 카드` 필수)
3. Google API Key 취득

[Get Started | Geocoding API | Google Developers](https://developers.google.com/maps/documentation/geocoding/start)

## Axios를 사용하여 입력된 주소의 좌표 가져오기

- 기존의 내장된 `fetch API`를 사용할 수 있지만, 에러 대응을 하기 더 편한 3rd 라이브러리(`Axios`)를 사용
  - [참고 링크](https://github.com/axios/axios)

### Axios 설치

```bash
$ npm install --save axios
```

### Axios 적용 - `app.ts`

```tsx
import axios from 'axios'

const form = document.querySelector('form')!
const addressInput = document.getElementById('address')! as HTMLInputElement

const GOOGLE_API_KEY = 'YOUR_API_KEY'
type CoogleGeocodingResponse = {
  results: { geometry: { location: { lat: number; lng: number } } }[]
  status: 'OK' | 'ZERO_RESULTS'
}
function searchAddressHandler(event: Event) {
  event.preventDefault()
  const enteredAddress = addressInput.value

  axios
    .get<CoogleGeocodingResponse>(
      `https://maps.googleapis.com/maps/api/geocode/json?address=${encodeURI(
        enteredAddress
      )}&key=${GOOGLE_API_KEY}`
    )
    .then((response) => {
      if (response.data.status !== 'OK') {
        throw new Error('Could not fetch locations')
      }
      const coordinates = response.data.results[0].geometry.location
    })
    .catch((err) => {
      alert(err.message)
      console.log(err)
    })
}

form.addEventListener('submit', searchAddressHandler)
```

- `import` 구문을 사용해 `axios` 패키지를 불러옴
- 내장된 타입스크립트 지원을 통해 컴파일 에러가 나는지 확인
- `https://maps.googleapis.com/maps/api/geocode/json?address=1600+Amphitheatre+Parkway,+Mountain+View,+CA&key=YOUR_API_KEY`
  - 필요한 쿼리스트링의 키값만 추출
  - Backtick(\`\`)으로 템플릿 리터럴을 생성하고,`address`값과, 취득한`Google API key`를 할당
  - `address`의 경우 `URI`에서 특정한 문자를 사용가능한 문자열(`UTF-8`)만 포함할 수 있도록 `encodeURI` 인자로 넘겨줌
- `axios`의 `then()`, `catch()` 구문을 통해 에러와 응답을 구분
  - `then`
    - 응답받은 객체의 (`response.data.results`) 배열의 첫번째 값을 찾아서 `coordinates` 변수에 할당
    - 응답 객체를 사용자 정의 타입(`CoogleGeocodingResponse`)에 필요한 키값으로 타입 선언
    - `status`가 성공 값이 아니면, Error 객체에 `message`를 전달해 `catch` 구문으로 넘어감

## Google 지도로 지도 렌더링

### CDN 적용 - `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Understanding TypeScript</title>
    <script src="dist/bundle.js" defer></script>
    <link rel="stylesheet" href="app.css" />
    <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_KEY" async defer></script>
  </head>
  <body>
    <div id="map">
      <p>Please enter an address!</p>
    </div>
    <form>
      <input type="text" id="address" />
      <button type="submit">SEARCH ADDRESS</button>
    </form>
  </body>
</html>
```

- 구글맵 SDK를 어플리케이션에 추가하는 스크립트를 추가
- CDN 방식으로 `script` 태그안에 `src`를 입력([공식 문서](https://developers.google.com/maps/documentation/javascript/overview)에서 제공)

### `app.ts`

```tsx
import axios from 'axios'

const form = document.querySelector('form')!
const addressInput = document.getElementById('address')! as HTMLInputElement

const GOOGLE_API_KEY = 'YOUR_API_KEY'
type CoogleGeocodingResponse = {
  results: { geometry: { location: { lat: number; lng: number } } }[]
  status: 'OK' | 'ZERO_RESULTS'
}
function searchAddressHandler(event: Event) {
  event.preventDefault()
  const enteredAddress = addressInput.value

  axios
    .get<CoogleGeocodingResponse>(
      `https://maps.googleapis.com/maps/api/geocode/json?address=${encodeURI(
        enteredAddress
      )}&key=${GOOGLE_API_KEY}`
    )
    .then((response) => {
      if (response.data.status !== 'OK') {
        throw new Error('Could not fetch locations')
      }
      const coordinates = response.data.results[0].geometry.location

      const map = new google.maps.Map(document.getElementById('map') as HTMLElement, {
        zoom: 16,
        center: coordinates,
      })

      new google.maps.Marker({
        position: coordinates,
        map: map,
      })
    })
    .catch((err) => {
      alert(err.message)
      console.log(err)
    })
}

form.addEventListener('submit', searchAddressHandler)
```

- `google.map` 생성자 함수를 인스턴스화해서 한 쌍의 좌표를 `center`에 전달
  - 초기에 해당 타입을 찾을 수 없기 때문에, `var google`을 `declare` 키워드로 전역 선언을 하거나 3rd 라이브러리를 사용해 구글 맵 관련 타입을 추가하는 패키지를 설치
    - 전역 설정하는 법
      - `declare var gogole:any;`
    - 패키지 모듈 설치하는 법
      - `npm install —save-dev @types/googlemaps`
  - `zoom`에 특정 값(ex. 16)을 할당하여 초기 확대 수치 값을 전달할 수 있음
- `google.maps.Marker`
  - 렌더링 된 맵에 마커를 추가하기 위해 추출한 좌표값(`coordinates`)과 요소(`map`)를 전달하여 지도가 정확한 장소에 렌더링 되도록해야 함

> **Referenced**

- [Understanding TypeScript - 2022 Edition](https://www.udemy.com/course/understanding-typescript/)
