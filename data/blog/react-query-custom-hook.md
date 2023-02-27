---
title: 더 큰 앱에서의 React Query - 설정, 집중화, 커스텀 후크
date: '2023-02-27'
tags: ['react']
draft: false
summary: 커스텀 쿼리 훅 - useTreatments, 폴백 데이터, useIsFetching을 사용하는 중앙 집중식 페칭 표시기, useQuery에 대한 onError 핸들러, 쿼리 클라이언트에 대한 onError 기본 값
layout: PostSimple
authors: ['default']
---

# 더 큰 앱에서의 React Query - 설정, 집중화, 커스텀 후크

## 커스텀 쿼리 훅 - useTreatments

### 커스텀 훅을 작성하는 이유

- 더 큰 앱들에서는 각 데이터 유형에 커스텀 훅을 만드는 데 큰 장점이 있음
  - 다수의 컴포넌트에서 데이터를 접근해야 하는 경우, useQuery 호출을 재작성할 필요가 없음
  - 다수의 useQuery 호출을 사용했다면 사용 중인 키의 종류가 헷갈릴 수 있는데 커스텀 훅을 사용하면 키를 헷갈릴 위험이 줄어 듦
    - 또한 사용하길 원하는 쿼리 함수를 혼동할 일이 없음
  - 커스텀 훅에 넣어주면 다수의 컴포넌트에서 굳이 불러올 필요 없음
  - 일반적으로 디스플레이 레이어에서 데이터를 어떻게 가져오는가에 대한 구현을 추상화하여 변경된 구현 발생 시, 컴포넌트를 업데이트 하지 않고 훅만 업데이트하면 됨

### `components/treatments/hooks/useTreatments.ts`

```tsx
import { useQuery } from 'react-query'

import type { Treatment } from '../../../../../shared/types'
import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useCustomToast } from '../../app/hooks/useCustomToast'

async function getTreatments(): Promise<Treatment[]> {
  const { data } = await axiosInstance.get('/treatments')
  return data
}

export function useTreatments(): Treatment[] {
  const { data } = useQuery(queryKeys.treatments, getTreatments)
  return data
}
```

- `axiosInstance`와 엔드포인트 `treatments`를 사용해 데이터를 가져옴

  ```tsx
  import axios, { AxiosRequestConfig } from 'axios'

  import { User } from '../../../shared/types'
  import { baseUrl } from './constants'

  export function getJWTHeader(user: User): Record<string, string> {
    return { Authorization: `Bearer ${user.token}` }
  }

  const config: AxiosRequestConfig = { baseURL: baseUrl }
  export const axiosInstance = axios.create(config)
  ```

- `useQuery`를 통해 `data`를 구조분해 할당하여 `Treatment` 배열을 반환
- `useQuery`에 쿼리 키와 인수로 전달하기 위한 익명함수(`getTreatments`)를 추가함
  - 쿼리키는 휴먼 에러를 방지하기 위해 상수로 불러옴
  - 쿼리 키의 일관성을 통해 캐시된 데이터를 캐시가 제공할 수 있도록함

## 폴백 데이터

### 원인

- `Treatments`를 매핑하려고 시도하다가 정의되지 않은 프로퍼티 맵을 읽을 수 없다는 오류 발생

### 해결

- 데이터가 정의되어 있지 않거나 아직 로딩 중인 경우 각 컴포넌트마다 오류 로딩의 처리를 진행하지 않고 데이터에 대한 폴백 값을 생성하여 확보할 수 있음(공백 배열)

  ```tsx
  export function useTreatments(): Treatment[] {
    const fallback = []
    const { data = fallback } = useQuery(queryKeys.treatments, getTreatments)
    return data
  }
  ```

## useIsFetching을 사용하는 중앙 집중식 페칭 표시기

- 각 컴포넌트마다 개별 로딩 인디케이터를 사용하는 대신 한 곳에서 집중된 로딩 인디케이터를 사용하도록 업데이트
- `React Query` 훅인 `useIsFetching`을 사용하여 로딩 스피너를 표시
  - 쿼리를 가져오는 중일 때는 로딩 스피너를 표시하고 가져오는 중인 쿼리가 없는 경우 끔

### `components/app/Loading.tsx`

```tsx
import { Spinner, Text } from '@chakra-ui/react'
import { ReactElement } from 'react'
import { useIsFetching } from 'react-query'

export function Loading(): ReactElement {
  const isFetching = useIsFetching()

  const display = isFetching ? 'inherit' : 'none'

  return (
    <Spinner
      thickness="4px"
      speed="0.65s"
      emptyColor="olive.200"
      color="olive.800"
      role="status"
      position="fixed"
      zIndex="9999"
      top="50%"
      left="50%"
      transform="translate(-50%, -50%)"
      display={display}
    >
      <Text display="none">Loading...</Text>
    </Spinner>
  )
}
```

- 현재 가져오기 상태인 쿼리 호출의 수(정수값 반환)를 나타내는 `useIsFetcing`의 값을 통해 0보다 크면 `display`는 `inherit`로 설정되어 참으로 평가되어 노출되고 현재 가져오는 값이 없으면 0(`false`)으로 반환되어 `display`값을 `none`으로 설정할 수 있음

## useQuery에 대한 onError 핸들러

- Chakra UI 라이브러리의 Toast 컴포넌트를 통해 에러를 전달하는 메시지를 노출
- 모든 useQuery 호출에 대해 onError 콜백을 구성
  - isError와 error를 분해하지 않고 onError 콜백을 구성해 다양한 방식으로 에러 처리를 할 수 있음

### `components/treatments/hooks/useTreatments.ts`

```tsx
import { useQuery } from 'react-query'

import type { Treatment } from '../../../../../shared/types'
import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useCustomToast } from '../../app/hooks/useCustomToast'

// for when we need a query function for useQuery
async function getTreatments(): Promise<Treatment[]> {
  const { data } = await axiosInstance.get('/treatments')
  return data
}

export function useTreatments(): Treatment[] {
  const toast = useCustomToast()

  const fallback = []
  const { data = fallback } = useQuery(queryKeys.treatments, getTreatments, {
    onError: (error) => {
      const title = error instanceof Error ? error.message : 'error connection to the server'
      toast({ title, status: 'error' })
    },
  })
  return data
}
```

- `useQuery`의 세번째 인자에 `onError` 핸들러를 추가하고 인수에 `error`를 넘겨줌
- 에러 타입은 `unknown`이므로 `error`가 JavaScript `Error` 클래스의 인스턴스라면 `error`의 `message` 프로퍼티에 이름을 설정
  - `error`가 `Error` 클래스의 인스턴스일 때만 해당 코드로 진입하므로 `message` 프로퍼티가 있음을 확신할 수 있음
- `toast` 컴포넌트에 삼항연산자를 거친 값을 할당한 title과 상태값을 지정해줌
- `React Query`는 작업을 중단하고 에러를 표시하기까지 기본값으로 세 번 작업을 시도함

## 쿼리 클라이언트에 대한 onError 기본 값

### `components/react-query/queryClient.ts`

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

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: queryErrorHandler,
    },
  },
})
```

- 오류 핸들링을 집중화하기 위해, `queryClient.ts` 파일에 `defaultOptions`로 `onError` 핸들링을 추가하여 관리
  - `useError` 훅이 존재할 수 없는 이유는 정수 이상의 값이 반환되야하는데, 각 에러마다 다른 문자열을 가진 오류가 각자 팝업 하도록 구현하기란 쉽지 않으므로 `queryClient`에 `onError` 핸들러를 추가하여 관리할 수 밖에 없음
  - 일반적으로 `queryClient`는 쿼리 혹은 변이(`Mutation`)에 대해 기본값을 가질 수 있으므로 `options` 객체에 `queries` 키를 가진 핸들러를 구성할 수 있음

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
