---
title: React Query 테스트
date: '2023-04-12'
tags: ['react']
draft: false
summary: Mock Service Worker를 포함한 테스트 설정, 테스트에서의 쿼리 클라이언트와 제공자, 렌더링된 쿼리 데이터 테스트하기, 쿼리 에러 테스트하기, 변이(Mutation) 테스트하기, 예약 필터 테스트
layout: PostSimple
authors: ['default']
---

# React Query 테스트

## Mock Service Worker를 포함한 테스트 설정

- REST API 방식으로 통신하는 MSW(Mock Service Worker)를 구성하여 React Query를 사용하는 서버와 클라이언트 간의 커뮤니케이션인 네트워크 호출을 테스트
  - 해당 도구를 사용해서 네트워크 호출을 차단한 후 정의한 핸들러를 토대로 응답을 반환
  - 테스트 중 네트워크 호출이 실제로 발생하지 않도록 방지하며 테스트 조건을 설정하는 데에도 도움을 제공함

### Installation

```bash
npm install msw --save-dev
# or
yarn add msw --dev
```

### Define Mock(`src/mock/server.js`)

```jsx
import { setupServer } from 'msw/node'

import { handlers } from './handlers'

// 주어진 요청 핸들러(request handlers)로 요청 모의(Mock) 서버를 설정 합니다.
export const server = setupServer(...handlers)
```

- 노드 서버를 구성한 `MSW`에서 가져오는데, 이는 공식문서에서 제공하는 보일러플레이트 코드로 적용

### `setupTest.js`

```jsx
import '@testing-library/jest-dom'

import { server } from './mocks/server'

// 모든 테스트 전에 API 모의를 설정합니다.
beforeAll(() => server.listen())

// 테스트 중에 추가된 모든 요청 핸들러를 재설정하여
// 다른 테스트에 영향을 미치지 않도록 합니다.
afterEach(() => server.resetHandlers())

// 테스트가 끝나면 정리(clean up)합니다.
afterAll(() => server.close())
```

- `beforeAll(() => server.listen());`
  - 해당 코드는 테스트 코드에서 API를 모의(Mock)하는 작업을 모든 테스트가 실행되기 전에 먼저 수행하도록 지시
  - 이를 통해 테스트 실행 시 실제 API에 의존하지 않고도 테스트를 수행할 수 있음
- `afterEach(() => server.resetHandlers());`
  - 해당 코드는 이전 테스트에서 추가된 요청 핸들러가 다음 테스트에 영향을 미치지 않도록 재설정하는 작업을 수행
  - 이를 통해 테스트 간 서로 독립적으로 실행될 수 있도록 함
- `afterAll(() => server.close());`
  - 해당 코드는 테스트가 완료된 후에 필요한 모든 정리(clean up) 작업을 수행하는 것을 의미함
  - 이를 통해 테스트 실행 중 생성된 데이터나 리소스들을 정리하고, 테스트 환경을 초기 상태로 되돌리는 등의 작업을 수행할 수 있음

### `src/mock/handlers.js`

```
import { rest } from 'msw';

import {
  mockAppointments,
  mockStaff,
  mockTreatments,
  mockUserAppointments,
} from './mockData';

export const handlers = [
  rest.get('http://localhost:3030/treatments', (req, res, ctx) => {
    return res(ctx.json(mockTreatments));
  }),
  rest.get('http://localhost:3030/staff', (req, res, ctx) => {
    return res(ctx.json(mockStaff));
  }),
  rest.get(
    'http://localhost:3030/appointments/:year/:month',
    (req, res, ctx) => {
      return res(ctx.json(mockAppointments));
    },
  ),
  rest.get('http://localhost:3030/user/:id/appointments', (req, res, ctx) => {
    return res(ctx.json({ appointments: mockUserAppointments }));
  }),
  rest.patch('http://localhost:3030/appointment/:id', (req, res, ctx) => {
    return res(ctx.status(200));
  }),
];
```

- 핸들러를 정의해 각 요청에 JSON 형식으로 모의 처리를 반환

### `src/mock/mockData.js`

```jsx
import dayjs from 'dayjs'

/** STAFF ************************************************************************* */
export const mockStaff = [
  {
    id: 1,
    name: 'Divya',
    treatmentNames: ['facial', 'scrub'],
    image: {
      fileName: 'divya.jpg',
      authorName: 'Pradeep Ranjan',
      authorLink:
        'https://unsplash.com/@tinywor1d?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText',
      platformName: 'Unsplash',
      platformLink: 'https://unsplash.com/',
    },
  },
  {
    id: 2,
    name: 'Sandra',
    treatmentNames: ['facial', 'massage'],
    image: {
      fileName: 'sandra.jpg',
      authorName: 'Pj Go',
      authorLink:
        'https://unsplash.com/@phizzahot?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText',
      platformName: 'Unsplash',
      platformLink: 'https://unsplash.com/',
    },
  },
  {
    id: 3,
    name: 'Michael',
    treatmentNames: ['facial', 'scrub', 'massage'],
    image: {
      fileName: 'michael.jpg',
      authorName: 'Fortune Vieyra',
      authorLink:
        'https://unsplash.com/@fortunevieyra?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText',
      platformName: 'Unsplash',
      platformLink: 'https://unsplash.com/',
    },
  },
  {
    id: 4,
    name: 'Mateo',
    treatmentNames: ['massage'],
    image: {
      fileName: 'mateo.jpg',
      authorName: 'Luis Quintero',
      authorLink:
        'https://unsplash.com/@jibarofoto?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText',
      platformName: 'Unsplash',
      platformLink: 'https://unsplash.com/',
    },
  },
]

/** APPOINTMENTS ************************************************************************* */
export const mockAppointments = {
  1: [
    {
      id: 10,
      treatmentName: 'Massage',
      userId: 1,
      dateTime: dayjs().toDate(),
    },
    {
      id: 11,
      treatmentName: 'Massage',
      dateTime: dayjs().add(-1, 'hours').toDate(),
    },
  ],
  12: [
    {
      id: 12,
      treatmentName: 'Scrub',
      dateTime: dayjs().add(2, 'hours').subtract(4, 'days').toDate(),
    },
    {
      id: 13,
      treatmentName: 'Facial',
      dateTime: dayjs().toDate(),
    },
  ],
  28: [
    {
      id: 16,
      treatmentName: 'Massage',
      dateTime: dayjs().add(3, 'hours').toDate(),
    },
  ],
  30: [
    {
      id: 17,
      treatmentName: 'Scrub',
      dateTime: dayjs().add(2, 'hours').toDate(),
    },
    {
      id: 18,
      treatmentName: 'Scrub',
      dateTime: dayjs().add(-2, 'hours').toDate(),
    },
  ],
  31: [
    {
      id: 19,
      treatmentName: 'Massage',
      dateTime: dayjs().add(3, 'hours').toDate(),
    },
    {
      id: 20,
      treatmentName: 'Facial',
      dateTime: dayjs().toDate(),
    },
  ],
}

/** TREATMENTS ************************************************************************* */
export const mockTreatments = [
  {
    id: 1,
    name: 'Massage',
    durationInMinutes: 60,
    image: {
      fileName: 'massage.jpg',
      authorName: 'Mariolh',
      authorLink:
        'https://pixabay.com/users/mariolh-62451/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=567021',
      platformName: 'Pixabay',
      platformLink:
        'https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=567021',
    },
    description: 'Restore your sense of peace and ease with a relaxing massage.',
  },
  {
    id: 2,
    name: 'Facial',
    durationInMinutes: 30,
    image: {
      fileName: 'facial.jpg',
      authorName: 'engin akyurt',
      authorLink:
        'https://unsplash.com/@enginakyurt?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText',
      platformName: 'Unsplash',
      platformLink: 'https://unsplash.com/',
    },
    description: 'Give your face a healthy glow with this cleansing treatment.',
  },
  {
    id: 3,
    name: 'Scrub',
    durationInMinutes: 15,
    image: {
      fileName: 'scrub.jpg',
      authorName: 'Monfocus',
      authorLink:
        'https://pixabay.com/users/monfocus-2516394/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5475880',
      platformName: 'Pixabay',
      platformLink:
        'https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5475880',
    },
    description: 'Invigorate your body and spirit with a scented Himalayan salt scrub.',
  },
]

/** USER APPOINTMENTS ************************************************************************* */
export const mockUserAppointments = [
  {
    id: 10,
    treatmentName: 'Massage',
    userId: 1,
    dateTime: dayjs().toDate(),
  },
  {
    id: 13,
    treatmentName: 'Facial',
    dateTime: dayjs().add(3, 'days').toDate(),
  },
]

/** USER  ************************************************************************* */
export const mockUser = {
  id: 1,
  email: 'test@test.com',
  name: 'Test Q. User',
  address: '123 Elm Street',
}
```

- 각 핸들러 요청에 사용(반환)될 모의 데이터 구성
  - 서버에서 반환되는 데이터와 매우 유사함

> Referenced. MSW(Getting Started)

[Mocking REST API - Getting Started](https://mswjs.io/docs/getting-started/mocks/rest-api)

## 테스트에서의 쿼리 클라이언트와 제공자

### 테스트 실행

- `npm test` 를 입력해 테스트 진행
- `renders response from query` 와 같은 에러가 출력되고, QueryClient가 올바르게 설정되어 있지 않기 때문에 오류가 발생됨

### 테스트 설정 시나리오

- 렌더링에 앞서 쿼리 제공자 내부에 렌더링하려는 요소를 포함시킬 함수를 생성해야 함
  - 캐시와 기본값 처리를 담당하기 위해 쿼리 제공자에는 쿼리 클라이언트가 필요함
- 각 테스트에 새로운 쿼리 클라이언트를 제공해 테스트를 격리해서 진행
- 쿼리 클라이언트에 함수를 포함시켜 생성해 테스팅에 대한 기본값을 설정할 때 도움을 받도록 구성
- 특정 테스트를 진행할 때 제공자 내부에 포함되도록 함수를 설정해 주는 것 외의 다른 클라이언트를 사용하려는 경우 함수 내부에 클라이언트를 명시할 수 있도록 허용해야 함

### 테스트 구성

`test-utils/index.tsx`

```tsx
import { render, RenderResult } from '@testing-library/react'
import { ReactElement } from 'react'
import { QueryClient, QueryClientProvider } from 'react-query'

// // 각 테스트마다 고유한 쿼리 클라이언트를 생성하는 함수 생성
const generateQueryClient = () => {
  return new QueryClient()
}

export function renderWithQueryClient(ui: ReactElement, client?: QueryClient): RenderResult {
  const queryClient = client ?? generateQueryClient()
  return render(<QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>)
}
```

- `renderWithQueryClient`
  - 컴포넌트를 렌더링하기에 앞서 래핑해 줄 함수 생성
  - React 요소에 해당하는 UI를 받고, 부가적으로 쿼리 클라이언트를 받음
  - 특정 테스트에서 오버라이드 하려면 특정 옵션이 필요한 경우에 렌더링 결과, 테스팅 라이브러리 렌더링 결과를 반환
  - `queryClient`를 알 수 없는 경우 `client`와 동일하게 설정하거나, `client`를 알 수 있는 경우 `generateQueryClient`를 실행해 새로 생성
- `<QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>`
  - `render`는 제공자의 역할로 `queryClient`를 제공하고 `render`에 전달한 `UI`를 `queryClient`와 래핑

`components/treatments/tests/Treatments.test.tsx`

```tsx
import { screen } from '@testing-library/react'

import { renderWithQueryClient } from '../../../test-utils'
import { Treatments } from '../Treatments'

test('renders response from query', () => {
  renderWithQueryClient(<Treatments />)
})
```

- 클라이언트를 제공하는 제공자에 래핑된 `treatment`를 렌더링

## 렌더링된 쿼리 데이터 테스트하기

### `components/treatments/tests/Treatments.test.tsx`

```tsx
import { screen } from '@testing-library/react'

import { renderWithQueryClient } from '../../../test-utils'
import { Treatments } from '../Treatments'

test('renders response from query', async () => {
  renderWithQueryClient(<Treatments />)

  const treatmentTitles = await screen.findAllByRole('heading', {
    name: /massage|facial|scrub/i,
  })

  expect(treatmentTitles).toHaveLength(3)
})
```

- 쿼리로부터 응답을 비동기식으로 렌더링하기 위해 비동기식 함수로 변환(`async/await`)
- `screen` → 렌더링 결과에 접근하는 방법
- `findBy` → 비동기 식으로 처리하는 접두사
  - `heading` 역할을 하는 속성이 `name`이고 `message`, `facial`, `scrub`을 대소문자 구분없이 할당하는 요소를 찾음
  - `fintAllByRole` → 세 단어 중 하나가 있고 해당 역할을 지닌 요소들을 모두 찾을 것을 요청

## 쿼리 에러 테스트하기

### `appointments/tests/Calendar.test.tsx`

```tsx
import { screen } from '@testing-library/react'
import { rest } from 'msw'

import { server } from '../../../mocks/server'
import { renderWithQueryClient } from '../../../test-utils'
import { Calendar } from '../Calendar'

test('Reserve appointment error', async () => {
  // (re)set handler to return a 500 error for appointments
  server.resetHandlers(
    rest.get('http://localhost:3030/appointments/:month/:year', (req, res, ctx) => {
      return res(ctx.status(500))
    })
  )

  renderWithQueryClient(<Calendar />)

  const alertToast = await screen.findByRole('alert')
  expect(alertToast).toHaveTextContent('Request failed with status code 500')
})
```

- 서버에서 `msw`에서 정의한 `api`를 가져오는 로직
- 쿼리 클라이언트로 달력 컴포넌트를 렌더링하지 않게 격리시킨 후, `Toast alert`을 확인
- 서버에서 동시에 응답을 주지 않기 위해서 `await/async`로 비동기 함수 처리
- `Toast`에서 원하는 경고가 나오는지 확인하기 위해서 어떤 내용이 반환되는지 확인해야 함
  - `findByRole`로 경고를 확인할 때 내용을 직접 넣을 수 없으므로, `toHaveTextContent`로 해당 문자열을 전달

### 에러 해결

- 위의 순서대로 코드를 작성한 후 `npm test`를 실행할 때, 무수히 많은 에러를 볼 수 있음
  - 에러가 발생하면 콘솔에 출력하므로 Logger를 설정해야 함
- `test-utils/index.tsx`

  ```tsx
  import { render, RenderResult } from '@testing-library/react'
  import { ReactElement } from 'react'
  import { QueryClient, QueryClientProvider, setLogger } from 'react-query'

  import { generateQueryClient } from '../react-query/queryClient'

  setLogger({
    log: console.log,
    warn: console.warn,
    error: () => {
      // swallow error without printing out
    },
  })

  const generateTestQueryClient = () => {
    /* ... */
  }

  export function renderWithQueryClient() {
    /* ... */
  }
  ```

  - `setLogger`를 설정해 `log`, `warn`, `error` 케이스 분리

- `react-query/queryClient.ts`

  ```tsx
  import { createStandaloneToast } from '@chakra-ui/react'
  import { QueryClient } from 'react-query'

  import { theme } from '../theme'

  const toast = createStandaloneToast({ theme })

  function queryErrorHandler(error: unknown): void {
    const id = 'react-query-error'
    const title = error instanceof Error ? error.message : 'error connecting to server'

    // prevent duplicate toasts
    toast.closeAll()
    toast({ id, title, status: 'error', variant: 'subtle', isClosable: true })
  }

  export function generateQueryClient(): QueryClient {
    return new QueryClient({
      defaultOptions: {
        queries: {
          onError: queryErrorHandler,
          staleTime: 600000, // 10 minutes
          cacheTime: 900000, // 15 minutes
          refetchOnMount: false,
          refetchOnReconnect: false,
          refetchOnWindowFocus: false,
        },
        mutations: {
          onError: queryErrorHandler,
        },
      },
    })
  }

  export const queryClient = generateQueryClient()
  ```

  - 쿼리 클라이언트에 에러 핸들러가 없으므로 클라이언트에 똑같은 기본 설정을 적용해야 함
  - `generateQueryClient` 함수를 만들고 `queryClient`를 그대로 반환하게 구현

- `test-utils/index.tsx`

  ```tsx
  import { render, RenderResult } from '@testing-library/react'
  import { ReactElement } from 'react'
  import { QueryClient, QueryClientProvider, setLogger } from 'react-query'

  import { generateQueryClient } from '../react-query/queryClient'

  setLogger({
    log: console.log,
    warn: console.warn,
    error: () => {
      // swallow error without printing out
    },
  })

  const generateTestQueryClient = () => {
    const client = generateQueryClient()
    const options = client.getDefaultOptions()
    options.queries = { ...options.queries, retry: false }
    return client
  }

  export function renderWithQueryClient(ui: ReactElement, client?: QueryClient): RenderResult {
    const queryClient = client ?? generateTestQueryClient()
    return render(<QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>)
  }
  ```

  - `generateQueryClient`를 가져와 `queryClient`를 만드는 대신 `generateQueryClient`를 반환
  - 쿼리 클라이언트를 업데이트해서 세 번의 재시도 후에 타임아웃이 되므로 클라이언트가 더 이상 재시도하지 않게끔 해야함
    - `getDefaultOptions` 메서드를 사용해 기본 옵션값을 불러온 후, `retry`를 `false`로 지정

## 변이(Mutation) 테스트하기

### `components/appointments/tests/appointmentMutations.test.tsx`

```tsx
import { fireEvent, screen, waitForElementToBeRemoved } from '@testing-library/react'
import { MemoryRouter } from 'react-router-dom'

import { mockUser } from '../../../mocks/mockData'
import { renderWithQueryClient } from '../../../test-utils'
import { Calendar } from '../Calendar'

// useUser를 모방하여 로그인된 사용자를 모방하는 것
jest.mock('../../user/hooks/useUser', () => ({
  __esModule: true,
  useUser: () => ({ user: mockUser }),
}))

test('Reserve appointment', async () => {
  renderWithQueryClient(
    <MemoryRouter>
      <Calendar />
    </MemoryRouter>
  )

  // 예약된 모든 일정 찾기
  const appointments = await screen.findAllByRole('button', {
    name: /\d\d? [ap]m\s+(scrub|facial|massage)/i,
  })

  // 예약을 위해 첫 번째 일정을 클릭하는 것을 의미
  fireEvent.click(appointments[0])

  // toast alert 체크
  const alertToast = await screen.findByRole('alert')
  expect(alertToast).toHaveTextContent('reserve')

  // 상태를 유지(clean)하고 알림이 사라질 때까지 기다리기 위해 알림을 닫습니다.
  const alertCloseButton = screen.getByRole('button', { name: 'Close' })
  alertCloseButton.click()
  await waitForElementToBeRemoved(alertToast)
})

test('Cancel appointment', async () => {
  /* ... */
})
```

- `jest.mock('../../user/hooks/useUser', () => ({…})`
  - useUser 모듈을 모킹하여 하나의 프로퍼티를 가진 객체(user)를 반환해 로그인한 사용자를 시뮬레이션
- `renderWithQueryClient();`
  - React Query에서 제공하는 모든 훅을 사용할 수 있게 해주는 클라이언트가 있는 공급자로 감싸서 렌더링
  - MemoryRouter로 래핑해서 하나의 컴포넌트로 사용할 수 있게 함
- `const appointments = await screen.findAllByRole('button', {});`
  - button 요소이고 해당 정규식을 통과하는 name 속성을 가진 예약 버튼을 모두 찾음
- `fireEvent.click(appointments[0]);`
  - 취득한 요소 중 첫번째 요소를 `fireEvent` 메서드로 `click` 이벤트 처리
- `reserve`라는 텍스트 콘텐츠가 포함된 경고창이 뜨는 지, 체크

  ```tsx
  const alertToast = await screen.findByRole('alert')
  expect(alertToast).toHaveTextContent('reserve')
  ```

- 알럿창이 열린 상태에서 다른 테스트에 영향을 미치지 않도록 경고창을 닫아야 하므로, 비동기로 처리

  ```tsx
  const alertCloseButton = screen.getByRole('button', { name: 'Close' })
  alertCloseButton.click()
  await waitForElementToBeRemoved(alertToast)
  ```

  - 테스트 함수를 닫기 전에 경고창이 사라졌는지 확인하기 위해 `Close`라는 이름의 버튼 요소를 찾아 `click` 이벤트를 발생시키고, 테스팅 라이브러리에서 제공하는 `waitForElementToBeRemoved` 메서드에 해당 `alert toast`를 전달

## 예약 필터 테스트

### `test-utils/index.tsx`

```tsx
import { render, RenderResult } from '@testing-library/react'
import { ReactElement } from 'react'
import { QueryClient, QueryClientProvider, setLogger } from 'react-query'

import { generateQueryClient } from '../react-query/queryClient'

setLogger({
  /* ... */
})

const generateTestQueryClient = () => {
  const client = generateQueryClient()
  const options = client.getDefaultOptions()
  options.queries = { ...options.queries, retry: false }
  return client
}

export function renderWithQueryClient(ui: ReactElement, client?: QueryClient): RenderResult {
  const queryClient = client ?? generateTestQueryClient()
  return render(<QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>)
}

export const createQueryClientWrapper = (): React.FC => {
  const queryClient = generateTestQueryClient()
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  )
}
```

- 객체 인수에서 구조 분해된 자식(children) 주위에 클라이언트와 함께 제공자를 래핑하는 함수를 반환

  - 래퍼 옵션을 사용할 수 있도록 함수를 만듦
  - 함수를 만들면 매번 새로운 queryClient를 생성하므로 테스트와 쿼리 클라이언트 간에 크로스오버가 발생하지 않도록 해줌

  > 각 테스트마다 queryClient가 존재해야 테스트를 깔끔하고 확실하게 만들 수 있음

### `components/appointments/tests/useAppointments.test.tsx`

```tsx
import { act, renderHook } from '@testing-library/react-hooks'

import { createQueryClientWrapper } from '../../../test-utils'
import { useAppointments } from '../hooks/useAppointments'
import { AppointmentDateMap } from '../types'

// AppointmentDateMap 객체에서 예약된 총 일정 수를 가져오는 데 도움이 되는 함수
const getAppointmentCount = (appointments: AppointmentDateMap) =>
  Object.values(appointments).reduce(
    (runningCount, appointmentsOnDate) => runningCount + appointmentsOnDate.length,
    0
  )

test('filter appointments by availalibility', async () => {
  const { result, waitFor } = renderHook(() => useAppointments(), {
    wrapper: createQueryClientWrapper(),
  })

  // 일정이 채워질 때까지 기다립니다.
  await waitFor(() => getAppointmentCount(result.current.appointments) > 0)

  const filteredAppointmentLength = getAppointmentCount(result.current.appointments)

  // 모든 예약을 필터링하도록 설정합니다.
  act(() => result.current.setShowAll(true))

  // 예약이 필터링될 때보다 더 많이 표시될 때까지 기다립니다.
  await waitFor(() => {
    return getAppointmentCount(result.current.appointments) > filteredAppointmentLength
  })
})
```

- `getAppointmentCount` 헬퍼 함수
  - `AppointmentDateMap` 객체를 입력값으로 받아서 해당 객체에 저장된 예약 일정의 총 개수를 반환
  - `Object.values(appointments)`를 통해 `appointments` 객체의 값들(즉, 각 날짜별 예약 정보)이 배열 형태로 반환하며 `reduce` 함수를 사용하여 모두 합산하여 총 예약 개수를 계산
    - `reduce` 함수의 콜백 함수는 누적값(`runningCount`)과 현재 배열 요소(`appointmentsOnDate`)를 인자로 받아, 콜백 함수에서는 누적값과 현재 배열 요소의 **`length`** 값을 더하여 `runningCount`에 할당하고, 최종적으로 총 예약 개수를 반환
- `renderHook(() ⇒ {});`
  - 훅의 결과를 포함하는 객체로써 함수를 취하고, `useAppointments` 훅을 전달
  - `wrapper` 옵션을 사용해 `createQueryClientWrapper()`를 호출해 해당 함수에서 생성하는 새로운 `queryClient`를 취할 수 있도록 함
    - 반환되는 함수는 훅을 래핑하는 데 사용
- `waitFor`
  - 훅의 비동기 액션을 기다릴 수 있게함
- `getAppointmentCount` 헬퍼함수에서 `useAppointments`에서 반환되는 `result.current.appointments`를 매개변수로 전달해 길이를 반환 받고, 예약이 필터링될 때보다 더 많이 표시될 때까지 기다림

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
