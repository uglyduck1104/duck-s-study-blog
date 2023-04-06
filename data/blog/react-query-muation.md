---
title: 변이(Mutation) - React Query로 서버 데이터 업데이트하기
date: '2023-04-06'
tags: ['react']
draft: false
summary: 변이(Mutation)와 변이 전역 설정, 커스텀 변이 훅, mutate 함수를 위한 Typescript, 변이 후의 쿼리 무효화하기, 쿼리 키 접두사, 변이로 예약 취소하기, 변이 응답으로 사용자와 쿼리 캐시 업데이트하기, 쿼리 취소 가능하게 만들기, 낙관적 업데이트 작성하기
layout: PostSimple
authors: ['default']
---

# 변이(Mutation) - React Query로 서버 데이터 업데이트하기

## 변이(Mutation)와 변이 전역 설정

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
    mutations: {
      onError: queryErrorHandler,
    },
  },
})
```

- 쿼리와 유사하게 오류의 경우 쿼리 클라이언트 `defaultOptions`의 `mutation` 속성에서 `onError` 콜백을 설정
  - `defaultOptions`는 지금까지 업데이트한 쿼리 속성과 변이 속성을 모두 가지고 있음

### `components/app/Loading.tsx`

```tsx
import { Spinner, Text } from '@chakra-ui/react'
import { ReactElement } from 'react'
import { useIsFetching, useIsMutating } from 'react-query'

export function Loading(): ReactElement {
  const isFetching = useIsFetching()
  const isMutating = useIsMutating()

  const display = isFetching || isMutating ? 'inherit' : 'none'

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

- `useIsFetching`과 유사하지만 변이 호출 중 현재 해결되지 않은 것이 있는지 알려줌
  - 따라서 `isMutation` 또는 `isFetching`에 표시되도록 `Loading` 컴포넌트를 업데이트

## 커스텀 변이 훅 - useReserveAppointments

- `useMutation`은 `useQuery`와 매우 유사하지만 다음과 같은 차이점이 있음
  - `useMutation`은 일회성이므로 캐시 데이터가 없음
  - 재시도는 구성할 수 있지만, 기본적으로는 없음
    - `useQuery`의 경우 기본적으로 세 번 재시도함
  - 관련된 데이터가 없으므로 리페치도 일어나지 않음
  - 캐시 데이터가 없으므로 `isLoading`과 `isFetching`이 구분되지 않음
    - `isLoading`은 데이터가 없을 때 이루어지는 페칭이므로 `isFetching`만 오로지 존재함
  - `onMutate` 콜백이 존재함
    - 낙관적(`Optimistic`) 쿼리에 사용하여 변이가 실패할 떄 롤백할 수 있도록 이전 상태를 저장하는 데 사용

### `components/appointments/hooks/useReserveAppointments.ts`

```tsx
// @ts-nocheck
import { useMutation } from 'react-query'

import { Appointment } from '../../../../../shared/types'
import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useCustomToast } from '../../app/hooks/useCustomToast'
import { useUser } from '../../user/hooks/useUser'

async function setAppointmentUser(
  appointment: Appointment,
  userId: number | undefined
): Promise<void> {
  if (!userId) return
  const patchOp = appointment.userId ? 'replace' : 'add'
  const patchData = [{ op: patchOp, path: '/userId', value: userId }]
  await axiosInstance.patch(`/appointment/${appointment.id}`, {
    data: patchData,
  })
}

type AppointmentMutationFunction = (appointment: Appointment) => void

export function useReserveAppointment(): AppointmentMutationFunction {
  const { user } = useUser()
  const toast = useCustomToast()

  const { mutate } = useMutation((appointment) => setAppointmentUser(appointment, user?.id))

  return mutate
}
```

- `useQuery`와 다르게 쿼리 키가 필요하지 않음(캐시와 관련 X)
- `setAppointmentUser`라는 변이 함수를 `useMutation hook`에 전달하고 객체 구조 할당을 통해 추출한 `mutate` 함수를 반환함
- `AppointmentMutationFunction`을 반환 타입으로 지정했으므로 다음 아이템에서 `mutate` 함수를 위한 `typescript`를 정의

## mutate 함수를 위한 Typescript

- 커스텀 훅에서 mutate 함수를 반환하는 유형

  ```
  UseMutationFunction<TData = unknown, TError = unknown, TVariable = void, TContext = unknown>
  ```

  - `TData`
    - 변이 함수 자체에서 반환된 데이터 유형으로써, 변이 함수는 데이터를 반환하지 않으므로 `void`로 설정
  - `TError`
    - 변이 함수에서 발생할 것으로 예상되는 오류(`Error`) 유형이므로 `Error`로 설정
  - `TVariable`
    - `mutate` 함수가 예상하는 변수 유형으로, 아래 예시의 경우 `Appointment`를 전달
  - `TContext`
    - 낙관적 업데이트 롤백을 위해 `onMutate`에서 설정하는 유형
    - 자세한 내용은 낙관적 업데이트 아이템에서 확인

### `components/appointments/hooks/useReserveAppointments.ts`

```tsx
import { UseMutateFunction, useMutation } from 'react-query'

import { Appointment } from '../../../../../shared/types'
import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useCustomToast } from '../../app/hooks/useCustomToast'
import { useUser } from '../../user/hooks/useUser'

async function setAppointmentUser(
  appointment: Appointment,
  userId: number | undefined
): Promise<void> {
  /* */
}

export function useReserveAppointment(): UseMutateFunction<void, unknown, Appointment, unknown> {
  const { user } = useUser()
  const toast = useCustomToast()

  const { mutate } = useMutation((appointment: Appointment) =>
    setAppointmentUser(appointment, user?.id)
  )

  return mutate
}
```

- `useMutationFunction`을 `react-query`에서 불러온 후 위의 형식에 맞게 인수를 전달
  - `Data`, `Error`, `Variables`, `Context` 순서로 전달

## 변이 후의 쿼리 무효화하기

- 통상적으로 변이 후에 데이터를 다시 가져옴으로써 관련 쿼리를 무효화한 후 데이터가 최신이 아님을 `React Query`에 알릴 수 있음

```tsx
import { UseMutateFunction, useMutation, useQueryClient } from 'react-query'

import { Appointment } from '../../../../../shared/types'
import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useCustomToast } from '../../app/hooks/useCustomToast'
import { useUser } from '../../user/hooks/useUser'

async function setAppointmentUser(
  appointment: Appointment,
  userId: number | undefined
): Promise<void> {
  if (!userId) return
  const patchOp = appointment.userId ? 'replace' : 'add'
  const patchData = [{ op: patchOp, path: '/userId', value: userId }]
  await axiosInstance.patch(`/appointment/${appointment.id}`, {
    data: patchData,
  })
}

export function useReserveAppointment(): UseMutateFunction<void, unknown, Appointment, unknown> {
  const { user } = useUser()
  const toast = useCustomToast()
  const queryClient = useQueryClient()

  const { mutate } = useMutation(
    (appointment: Appointment) => setAppointmentUser(appointment, user?.id),
    {
      onSuccess: () => {
        queryClient.invalidateQueries([queryKeys.appointments])
        toast({
          title: 'You have reserved the appointment!',
          status: 'success',
        })
      },
    }
  )

  return mutate
}
```

- `queryClient`에는 `invalicateQueries` 메서드가 있어서 특정한 트리거 후 `appointment`를 변경할 때 `appointment` 데이터에 대한 캐시를 무효화하는데 사용됨
  - 쿼리를 만료(`stale`)로 표시하고 쿼리가 현재 렌더링 중이면 리페치(`Refetch`)를 트리거함
- `onSuccess` 핸들러가 관련 쿼리를 무효화하고 이에 따라 데이터 리페치가 트리거 되는 형태
- 접두사로 `queryKeys` 상수와 `Appointments` 속성이 있는 쿼리를 무효화

## 쿼리 키 접두사

- 이전 아이템에서 `onSuccess` 핸들러가 관련 쿼리를 무효화했지만, 해당 쿼리키에 해당하는 데이터만 무효화 됨
- `Appointments`에서 변이를 실행할 때 연관된 모든 쿼리를 무효화하는 방식으로 접근
- `invalidateQueries`는 정확한 키가 아닌 접두사를 사용하므로 동일한 쿼리 키 접두사로 서로 관련된 쿼리를 설정하면 모든 쿼리를 한 번에 무효화할 수 있음
- 다른 `queryClient` 메서드도 `removeQueries`와 같은 쿼리 키 접두사를 사용함

### `user/hooks/useUserAppointments.ts`

```tsx
import dayjs from 'dayjs'
import { useQuery } from 'react-query'

import type { Appointment, User } from '../../../../../shared/types'
import { axiosInstance, getJWTHeader } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useUser } from './useUser'

async function getUserAppointments(user: User | null): Promise<Appointment[] | null> {
  if (!user) return null
  const { data } = await axiosInstance.get(`/user/${user.id}/appointments`, {
    headers: getJWTHeader(user),
  })
  return data.appointments
}

export function useUserAppointments(): Appointment[] {
  const { user } = useUser()

  const fallback: Appointment[] = []
  const { data: userAppointments = fallback } = useQuery(
    [queryKeys.appointments, queryKeys.user, user?.id],
    () => getUserAppointments(user),
    { enabled: !!user }
  )

  return userAppointments
}
```

- 이전에는 `useQuery`의 첫번째 인자로 `user-appointments`를 문자열로 하드코딩 했지만, 쿼리 접두사를 통해 `[queryKeys.appointments, queryKeys.user, user?.id]`를 전달하고 쿼리 키를 업데이트 했으므로 `useUser`에서도 제거할 때 업데이트 해야함

### `user/hooks/useUser.ts`

```tsx
import { AxiosResponse } from 'axios'
import { useQuery, useQueryClient } from 'react-query'

import type { User } from '../../../../../shared/types'
import { axiosInstance, getJWTHeader } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { clearStoredUser, getStoredUser, setStoredUser } from '../../../user-storage'

async function getUser(user: User | null): Promise<User | null> {
  if (!user) return null
  const { data }: AxiosResponse<{ user: User }> = await axiosInstance.get(`/user/${user.id}`, {
    headers: getJWTHeader(user),
  })
  return data.user
}

interface UseUser {
  user: User | null
  updateUser: (user: User) => void
  clearUser: () => void
}

export function useUser(): UseUser {
  const queryClient = useQueryClient()
  const { data: user } = useQuery(queryKeys.user, () => getUser(user), {
    initialData: getStoredUser,
    onSuccess: (received: User | null) => {
      if (!received) {
        clearStoredUser()
      } else {
        setStoredUser(received)
      }
    },
  })

  function updateUser(newUser: User): void {
    queryClient.setQueryData(queryKeys.user, newUser)
  }

  function clearUser() {
    queryClient.setQueryData(queryKeys.user, null)
    queryClient.removeQueries([queryKeys.appointments, queryKeys.user])
  }

  return { user, updateUser, clearUser }
}
```

- 하드 코딩된 `user-appointments` 문자열을 마찬가지로 `queryKeys.appointments`, `queryKeys.user`로 바꿈
- `removeQueries`도 쿼리 키 접두사를 사용하므로 쿼리 키에 두 가지가 첫 항목으로 포함되어 있는 한 사용자 `ID`는 따로 지정할 필요가 없음

## 변이로 예약 취소하기

### `components/user/hooks/useUserAppointments`

```tsx
import { UseMutateFunction, useMutation, useQueryClient } from 'react-query'

import { Appointment } from '../../../../../shared/types'
import { axiosInstance } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useCustomToast } from '../../app/hooks/useCustomToast'

async function removeAppointmentUser(appointment: Appointment): Promise<void> {
  const patchData = [{ op: 'remove', path: '/userId' }]
  await axiosInstance.patch(`/appointment/${appointment.id}`, {
    data: patchData,
  })
}

export function useCancelAppointment(): UseMutateFunction<void, unknown, Appointment, unknown> {
  const queryClient = useQueryClient()
  const toast = useCustomToast()

  const { mutate } = useMutation(removeAppointmentUser, {
    onSuccess: () => {
      queryClient.invalidateQueries([queryKeys.appointments])
      toast({
        title: 'You have canceled the appointment!',
        status: 'warning',
      })
    },
  })
  return mutate
}
```

- `useQueryClient` 훅에서 `queryClient`를 가져온 후 `useMutation`을 실행하여 반환 값에서 `mutate` 함수를 구조 분해
  - `useCancelAppointment` 훅으로 사용자에게 전달할 내용
- `mutate`를 실행하면 `appoinment` 인수를 전달하고, `useMutation`은 해당 `appoinment` 인수를 변이 함수(`removeAppointmentUser`)에 전달
- `onSuccess` 핸들러에 `queryKeys.Appoinments`로 시작하는 모든 쿼리를 무효화하고 `toast` 알림을 띄워줌
- `UseMutateFunction`의 타입
  - 변이 함수는 `void`를 반환하고 오류 유형은 `unknown`으로 지정
  - `mutate` 함수에 대한 인수는 `Appoinment` 의 유형이고, `onMutate`의 컨텍스트는 없으므로 `unknown`으로 지정

## 변이 응답으로 사용자와 쿼리 캐시 업데이트하기

### `components/user/hooks/usePatchUser.ts`

- 패치를 생성해서 서버에서 인증 보호된 라우터와 헤더를 보내는 커스텀 훅

```tsx
import jsonpatch from 'fast-json-patch'
import { UseMutateFunction, useMutation } from 'react-query'

import type { User } from '../../../../../shared/types'
import { axiosInstance, getJWTHeader } from '../../../axiosInstance'
import { useCustomToast } from '../../app/hooks/useCustomToast'
import { useUser } from './useUser'

async function patchUserOnServer(
  newData: User | null,
  originalData: User | null
): Promise<User | null> {
  if (!newData || !originalData) return null
  const patch = jsonpatch.compare(originalData, newData)

  const { data } = await axiosInstance.patch(
    `/user/${originalData.id}`,
    { patch },
    {
      headers: getJWTHeader(originalData),
    }
  )
  return data.user
}

export function usePatchUser(): UseMutateFunction<User, unknown, User, unknown> {
  const { user, updateUser } = useUser()
  const toast = useCustomToast()

  const { mutate: patchUser } = useMutation(
    (newUserData: User) => patchUserOnServer(newUserData, user),
    {
      onSuccess: (userData: User | null) => {
        if (user) {
          updateUser(userData)
          toast({
            title: 'User updated!',
            status: 'success',
          })
        }
      },
    }
  )

  return patchUser
}
```

- `useUser` 훅으로 쿼리 캐시를 업데이트하기 위해 `user`와 `updateUser`를 구조분해 할당
  - `updateUser`는 사용자 데이터를 가져와서 state를 설정하고 로컬 스토리지와 쿼리 캐시를 설정
- `mutate` 함수를 `useMutation`을 호출해서 구조 분해하고 `patchUser`로 이름을 변경
- `patchUserOnServer`에 `newData`와 `originalData`로 각각의 인수를 전달
  - `originalData`는 `useUser` 훅의 `state`에 있는 사용자의 데이터가 됨
  - `newData`는 `mutate`에 전달되는 것(사용자 타입)
- `onSuccess` 핸들러를 통해 서버에서 받은 응답으로 사용자를 업데이트
  - `onSuccess`는 변이 함수에서 반환된 모든 값을 인자로 받음
  - 취득한 사용자 데이터가 참인경우에만 변이 함수에서 얻은 응답을 가져와 `updateUser`에 전달함

## 쿼리 취소 가능하게 만들기

- 낙관적 업데이트의 중요한 부분은 서버로 요청이 전달되는 도중에 취소할 수 있다는 점으로 서버에서 오는 모든 데이터가 캐시의 낙관적 업데이트를 덮어쓰는 일이 없도록 해줌

### `components/user/hooks/useUser.ts`

```tsx
import { AxiosResponse } from 'axios'
import { useQuery, useQueryClient } from 'react-query'

import type { User } from '../../../../../shared/types'
import { axiosInstance, getJWTHeader } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { clearStoredUser, getStoredUser, setStoredUser } from '../../../user-storage'

async function getUser(user: User | null, signal: AbortSignal): Promise<User | null> {
  if (!user) return null
  const { data }: AxiosResponse<{ user: User }> = await axiosInstance.get(`/user/${user.id}`, {
    headers: getJWTHeader(user),
    signal,
  })
  return data.user
}

interface UseUser {
  user: User | null
  updateUser: (user: User) => void
  clearUser: () => void
}

export function useUser(): UseUser {
  const queryClient = useQueryClient()
  const { data: user } = useQuery(queryKeys.user, ({ signal }) => getUser(user, signal), {
    initialData: getStoredUser,
    onSuccess: (received: User | null) => {
      if (!received) {
        clearStoredUser()
      } else {
        setStoredUser(received)
      }
    },
  })

  function updateUser(newUser: User): void {
    /* ... */
  }

  function clearUser() {
    /* ... */
  }

  return { user, updateUser, clearUser }
}
```

- `useUser` 훅에 있는 `getUser` 호출은 낙관적 업데이트 대비 상대적으로 오래 되었을 수 있는 데이터를 서버로부터 가져오는 쿼리이므로 낙관적 업데이트 이후 수동으로 취소할 수 있도록 설정해야 하는 쿼리 함수
- `React Query`에서 쿼리를 수동으로 취소하기 위해 표준 자바스크립트 인터페이스인 `AbortController`를 사용
  - `AbortSignal` 객체를 `DOM` 요청에 보냄
- `React Query`의 일부 쿼리는 배후에서 자동으로 취소됨
  - ex) 어떤 쿼리가 실행 중에 기한이 만료(`stale`)되거나 비활성(`inactive`)되는 경우, 또는 쿼리 결과를 보여주는 컴포넌트가 해제되는 경우
- 결론적으로 `React Query`에서 `Axois` 쿼리를 수동으로 취소하려면 중단 신호를 전달해야 함
  - `useUser` 훅에서 `useQuery`를 통해 data를 구조분해 할때, 두 번째 인수에 `signal(AbortSignal)`을 `getUser`에 전달
  - `signal`을 `Axios` 인스턴스의 한 구성으로 전달

### 정리

1. `useQuery(queryKeys.user)`

- 사용자 쿼리 키를 지닌 useQuery가 AbortController를 관리

2. `AbortController`

- 쿼리 함수인 getUser에 전달되는 signal를 생성

3. `getUser`

- AbortController로 부터 받은 신호를 axios에 전달

4. `axios`

- 해당 signal에 연결된 상태로써, 취소 이벤트에 대하여 signal을 수신

5. `queryClient.cancelQuery(queryKeys.user)`

- cancelQuery 메서드가 실행되면 AbortController를 관리하는 동일한 키에 실행하는 경우 AbortController에 취소 이벤트를 전달

> 즉, Axios 호출 등 signal을 listen하는 모든 객체는 해당 취소 이벤트를 수신하고 중단함

## 낙관적 업데이트 작성하기

### `components/user/hooks/usePatchUser.ts`

```tsx
import jsonpatch from 'fast-json-patch'
import { UseMutateFunction, useMutation, useQueryClient } from 'react-query'

import type { User } from '../../../../../shared/types'
import { axiosInstance, getJWTHeader } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { useCustomToast } from '../../app/hooks/useCustomToast'
import { useUser } from './useUser'

async function patchUserOnServer(
  newData: User | null,
  originalData: User | null
): Promise<User | null> {
  if (!newData || !originalData) return null
  const patch = jsonpatch.compare(originalData, newData)

  const { data } = await axiosInstance.patch(
    `/user/${originalData.id}`,
    { patch },
    {
      headers: getJWTHeader(originalData),
    }
  )
  return data.user
}

export function usePatchUser(): UseMutateFunction<User, unknown, User, unknown> {
  const { user, updateUser } = useUser()
  const toast = useCustomToast()
  const queryClient = useQueryClient()

  const { mutate: patchUser } = useMutation(
    (newUserData: User) => patchUserOnServer(newUserData, user),
    {
      // onMutate는 onError에 전달되는 컨텍스트를 반환합니다.
      onMutate: async (newData: User | null) => {
        // 사용자 데이터에 대한 모든 요청을 취소하여 이전 서버 데이터가
        // optimistic update를 덮어쓰지 않도록합니다.
        queryClient.cancelQueries(queryKeys.user)

        // 이전 사용자 값의 스냅샷을 가져오고,
        const previousUserData: User = queryClient.getQueryData(queryKeys.user)

        // 새 사용자 값으로 optimistic update를 캐시에 업데이트합니다.
        updateUser(newData)

        // 스냅샷 된 값을 포함하는 컨텍스트 객체를 반환합니다.
        return { previousUserData }
      },
      onError: (error, newData, context) => {
        // 캐시를 저장된 값으로 롤백합니다
        if (context.previousUserData) {
          updateUser(context.previousUserData)
          toast({
            title: 'Update failed; restoring previous values',
            status: 'warning',
          })
        }
      },
      onSuccess: (userData: User | null) => {
        if (user) {
          updateUser(userData)
          toast({
            title: 'User updated!',
            status: 'success',
          })
        }
      },
      onSettled: () => {
        // 사용자 쿼리를 무효화하여 서버 데이터와 동기화되는지 확인합니다
        queryClient.invalidateQueries(queryKeys.user)
      },
    }
  )

  return patchUser
}
```

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
