---
title: 쿼리 특성 2. 프리페칭(Pre-fetching)과 페이지 매김
date: '2023-03-16'
tags: ['react']
draft: false
summary: useQuery의 select 옵션으로 데이터 필터링하기, 리페치(Re-fetch)의 정의 및 코드 구현, 전역 리페치 옵션, 리페치 기본값 오버라이드와 폴링, 폴링 - 간격에 따른 자동 리페칭
layout: PostSimple
authors: ['default']
---

# 쿼리 특성 2. 프리페칭(Pre-fetching)과 페이지 매김

## useQuery의 select 옵션으로 데이터 필터링하기

- useQuery의 select 옵션을 사용하면 쿼리 함수가 반환하는 데이터를 변환할 수 있음
- select 옵션을 사용하는 이유는 불필요한 연산을 줄이고 최적화(Memoization)를 하기위해 사용
  - select 함수를 삼중 등호로 비교하며 데이터와 함수가 모두 변경되었을 경우에만 실행됨
  - 변동이 없을 경우에는 select 함수를 재실행하지 않는 것이 최적화와 안정화에 좋음
  - 안정화를 위해 익명함수의 경우 React의 useCallback 훅을 사용

> Referenced. tkdodo blog

[React Query Data Transformations](https://tkdodo.eu/blog/react-query-data-transformations)

### `components/appointments/hook/useAppointments.ts`

```tsx
// @ts-nocheck
import dayjs from 'dayjs'
import { Dispatch, SetStateAction, useCallback, useState } from 'react'
import { useQuery } from 'react-query'

import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useUser } from '../../user/hooks/useUser'
import { AppointmentDateMap } from '../types'
import { getAvailableAppointments } from '../utils'
import { getMonthYearDetails, getNewMonthYear, MonthYear } from './monthYear'

async function getAppointments(year: string, month: string): Promise<AppointmentDateMap> {
  const { data } = await axiosInstance.get(`/appointments/${year}/${month}`)
  return data
}

interface UseAppointments {
  appointments: AppointmentDateMap
  monthYear: MonthYear
  updateMonthYear: (monthIncrement: number) => void
  showAll: boolean
  setShowAll: Dispatch<SetStateAction<boolean>>
}

export function useAppointments(): UseAppointments {
  const currentMonthYear = getMonthYearDetails(dayjs())

  const [monthYear, setMonthYear] = useState(currentMonthYear)

  function updateMonthYear(monthIncrement: number): void {
    setMonthYear((prevData) => getNewMonthYear(prevData, monthIncrement))
  }

  const [showAll, setShowAll] = useState(false)
  const { user } = useUser()

  const selectFn = useCallback((data) => getAvailableAppointments(data, user), [user])

  const fallback = {}

  const { data: appointments = fallback } = useQuery(
    [queryKeys.appointments, monthYear.year, monthYear.month],
    () => getAppointments(monthYear.year, monthYear.month),
    {
      select: showAll ? undefined : selectFn,
    }
  )

  return { appointments, monthYear, updateMonthYear, showAll, setShowAll }
}
```

- `useQuery`의 인수중에 객체 타입의 `select` 옵션 중, `showAll` 상태값에 따라서 참일 경우에는 모든 데이터를 반환하도록 `undefined` 값을 할당해 존재하지 않는 값이며 옵션으로 포함하지 않음
  - 반대의 경우 `selectFn` 함수를 실행하고 `useCallback`을 사용해 안정적인 함수로 만듦
    - `select` 함수는 `getAvailableAppintments`를 실행해 `appointments` 데이터와 `user` 데이터를 인수로 취함
  - 종속성 배열에는 `user`를 넣어 로그인하는 사용자가 바뀌거나 사용자가 로그아웃할 때마다 함수를 변경함

## 리페치(Re-fetch)의 정의 및 코드 구현

### 리페치의 정의

- 기본적으로 리페칭을 위해서 염두에 둬야 할 사항은 서버가 만료된 데이터를 업데이트하는 것
  - 의지와 상관없이 일정 시간이 지나면 서버가 만료된 데이터를 삭제함
- `stale` 쿼리는 다음과 같은 조건 하에 자동적으로 다시 가져옴
  - 새로운 쿼리 인스턴스가 많아지는 경우
  - 쿼리 키가 처음 호출되는 경우
  - 쿼리를 호출하는 컴포넌트를 증가시키는 경우
  - 창을 재포커스 하는 경우
  - 만료된 데이터의 업데이트 여부를 확인할 수 있는 네트워크가 다시 연결된 경우
  - 리페칭 간격이 지난 경우
- 제공하는 리페칭 옵션
  - refetchOnMount(`boolean`)
  - refetchOnWindowFocus(`boolean`)
  - refetchOnReconnect(`boolean`)
  - refetchInterval(`millisecond`)
- 주의 사항
  - 리페칭을 제한할 때는 변동이 잦지 않은 데이터에 적용해야 함
  - 미세한 수정에도 큰 변화를 불러오는 데이터에 적용하지 말아야 함

### 리페치 구현

`components/appointments/hook/useAppointments.ts`

```tsx
import { useQuery, useQueryClient } from 'react-query'

import type { Treatment } from '../../../../../shared/types'
import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'

// for when we need a query function for useQuery
async function getTreatments(): Promise<Treatment[]> {
  const { data } = await axiosInstance.get('/treatments')
  return data
}

export function useTreatments(): Treatment[] {
  const fallback = []
  const { data = fallback } = useQuery(queryKeys.treatments, getTreatments, {
    staleTime: 600000, // 10 minutes
    cacheTime: 900000, // 15 minutes (doesn't make sense for staleTime to exceed cacheTime)
    refetchOnMount: false,
    refetchOnWindowFocus: false,
    refetchOnReconnect: false,
  })
  return data
}

export function usePrefetchTreatments(): void {
  const queryClient = useQueryClient()
  queryClient.prefetchQuery(queryKeys.treatments, getTreatments, {
    staleTime: 600000, // 10 minutes
    cacheTime: 900000, // 15 minutes (doesn't make sense for staleTime to exceed cacheTime)
  })
}
```

- `useQuery`의 세번째 인자에 객체 옵션 `staleTime`, `cacheTime`, `refetchOnMount`, `refetchOnWindowFocus`, `refetchOnReconnect`를 키로 가지는 값을 설정
  - `staleTime`
    - 60만 밀리초(10분)를 설정해 웹사이트의 서비스 목록을 최대 10분으로 설정
  - `cacheTime`
    - 기본값은 5분이므로 만료 타임이 캐싱 타임을 초과하면 안되는 조건을 기준으로 만료된 데이터를 불러오는 동안 캐싱에 백업된 내용을 보여줌
    - 만료된 데이터를 불러오는 동안 캐싱에 백업된 내용이 보여지므로, 만료된 데이터보다 캐싱이 먼저 만료된다는 것은 리페칭을 실행시키는 동안 보여줄 화면이 없다는 것임
  - `refetchOnMount`, `refetchOnWindowFocus`, `refetchOnReconnect`
    - 리페칭의 대상이 되는 조건(기본값: true)을 `false`값으로 할당해서 제한 사항을 걸어줌
- `prefetch` 역시, 동일하게 `staleTime`과 `cacheTime`을 설정

> 만약에, 무수히 많은 컴포넌트에 위와 같은 리페칭 옵션을 모두 적용하고 싶을 때에는 어떻게 할 것인가?

## 전역 리페치 옵션

- 각각의 useQuery마다 쿼리 옵션을 각각 설정해서 관리하는 방법 보다 전역적으로 쿼리 옵션을 적용해 한 곳에서 관리할 수 있게할 수 있음

### `src/react-query/queryClient.ts`

```tsx
import { createStandaloneToast } from '@chakra-ui/react'
import { QueryClient } from 'react-query'

import { theme } from '../theme'

const toast = createStandaloneToast({ theme })

function queryErrorHandler(error: unknown): void {
  const id = 'react-query-error'
  const title = error instanceof Error ? error.message : 'error connecting to server'

  toast.closeAll()
  toast({ id, title, status: 'error', variant: 'subtle', isClosable: true })
}

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: queryErrorHandler,
      staleTime: 600000, // 10 minutes
      cacheTime: 900000, // 15 minutes
      refetchOnMount: false,
      refetchOnReconnect: false,
      refetchOnWindowFocus: false,
    },
  },
})
```

- `queryClient` 인스턴스에 앞서 설정했던 각각의 옵션을 객체 형태로 전달
- `onError`을 기본 옵션으로 설정해 둔 덕에 `useQuery`를 일일이 에러 핸들러로 등록하지 않아도 됨
- 해당 옵션들은 단지 전역 설정을 위한 옵션이므로, 데이터를 무용지물로 만들거나 사용자들은 원하는 정보를 얻지 못하게 될 우려가 있으므로 예시로만 참고

## 리페치 기본값 오버라이드와 폴링

- 전역으로 기본 설정한 옵션들을 반대로 오버라이딩해서, 필요한 부분에 적용할 수 있음

### `components/appointments/hook/useAppointments.ts`

```tsx
// @ts-nocheck
import dayjs from 'dayjs'
import { Dispatch, SetStateAction, useCallback, useEffect, useState } from 'react'
import { useQuery } from 'react-query'

import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useUser } from '../../user/hooks/useUser'
import { AppointmentDateMap } from '../types'
import { getAvailableAppointments } from '../utils'
import { getMonthYearDetails, getNewMonthYear, MonthYear } from './monthYear'

// common options for both useQuery and prefetchQuery
const commonOptions = { staleTime: 0, cacheTime: 300000 }

async function getAppointments(year: string, month: string): Promise<AppointmentDateMap> {
  const { data } = await axiosInstance.get(`/appointments/${year}/${month}`)
  return data
}

interface UseAppointments {
  appointments: AppointmentDateMap
  monthYear: MonthYear
  updateMonthYear: (monthIncrement: number) => void
  showAll: boolean
  setShowAll: Dispatch<SetStateAction<boolean>>
}

export function useAppointments(): UseAppointments {
  const currentMonthYear = getMonthYearDetails(dayjs())

  const [monthYear, setMonthYear] = useState(currentMonthYear)

  function updateMonthYear(monthIncrement: number): void {
    setMonthYear((prevData) => getNewMonthYear(prevData, monthIncrement))
  }
  const [showAll, setShowAll] = useState(false)

  const { user } = useUser()

  const selectFn = useCallback((data) => getAvailableAppointments(data, user), [user])

  const queryClient = useQueryClient()

  useEffect(() => {
    const nextMonthYear = getNewMonthYear(monthYear, 1)
    queryClient.prefetchQuery(
      [queryKeys.appointments, nextMonthYear.year, nextMonthYear.month],
      () => getAppointments(nextMonthYear.year, nextMonthYear.month),
      commonOptions
    )
  }, [queryClient, monthYear])

  const fallback = {}

  const { data: appointments = fallback } = useQuery(
    [queryKeys.appointments, monthYear.year, monthYear.month],
    () => getAppointments(monthYear.year, monthYear.month),
    {
      select: showAll ? undefined : selectFn,
      ...commonOptions,
      refetchOnMount: true,
      refetchOnReconnect: true,
      refetchOnWindowFocus: true,
    }
  )

  return { appointments, monthYear, updateMonthYear, showAll, setShowAll }
}
```

- 전역으로 설정한 옵션에서 필요한 부분을 오버라이딩하기 위해 `useQuery` 옵션의 세번째 인자에 `refetch` 옵션들을 `true`로 정의하고, 폴링 옵션(`staleTime`, `cacheTime`)을 `commonOptions` 객체로 관리
  - `staleTime`은 0으로 설정해, 즉시 만료되도록 하고, `cacheTime`은 5분으로 설정

## 폴링 - 간격에 따른 자동 리페칭

- 주기적으로 데이터를 자동으로 리페칭하기 위해서는 만료 시간, 캐시 타임처럼 리페칭 간격 옵션을 사용해 해당 간격 시간마다 데이터를 리페칭할 수 있음
  - `refetchInterval`
    - `cacheTime`과 `staleTime` 설정과 동일하게 millisecond(밀리세컨드) 단위로 설정해 줄 수 있음

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
