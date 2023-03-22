---
title: React Query와 인증
date: '2023-03-21'
tags: ['react']
draft: false
summary: useAuth, useUser와 useQuery, React Query와 Auth 통합하기, localStorage에서 사용자 데이터 유지하기, 의존적 쿼리 - userAppointments, 쿼리 클라이언트 removeQueries 메서드
layout: PostSimple
authors: ['default']
---

# React Query와 인증

## useAuth, useUser와 useQuery

### useAuth Hook

- 로그인, 가입, 로그아웃 기능을 제공하는 훅
- `components/user/hooks/useUser.ts`

  ```tsx
  import { AxiosResponse } from 'axios'
  import { useQuery } from 'react-query'

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
    const { data: user } = useQuery(queryKeys.user, () => getUser(user))

    function updateUser(newUser: User): void {
      // TODO: update the user in the query cache
    }

    function clearUser() {
      // TODO: reset user to null in query cache
    }

    return { user, updateUser, clearUser }
  }
  ```

  - `updateUser`
    - 사용자 로그인이나 사용자 정보 업데이트 처리
  - `clearUser`
    - 로그아웃 처리
  - `useUser`의 책임은 `localStorage`와 `Query Cache`에서 사용자의 상태를 유지
  - `getUser`
    - 사용자 로그인 여부에 따라 사용자 객체나 `null`이 될 수 있음
    - 로그인한 사용자가 없으면 서버에 가지 않고 `null`을 반환함
    - 로그인한 사용자가 있으면 서버로 이동하여 로그인한 사용자의 `user.id` 데이터를 취득
      - 서버에서 확인하려면 `JWTHeader`를 포함해야 함
  - `const { data: user } = useQuery(queryKeys.user, () => getUser(user));`
    - 첫 번째 인수로 쿼리 키를 가져옴
    - 두 번째 인수로는 쿼리 함수를 실행하여 getUser 함수를 호출하고, 기존 user의 값을 업데이트하는 데 사용
      - 기존 `user`의 값 취득 → `setQueryData` 함수를 사용해 `useAuth` 함수를 쿼리 캐시와 연결하고 로컬 스토리지에 데이터를 보존하는 것이 목표

## React Query와 Auth 통합하기

- 위의 코드에서는 애초에 `user`가 정의되지 않았기때문에 항상 거짓의 값이 나오고 사용자는 `null`을 반환함
- `updateUser`의 함수와 `clearUser` 함수를 필요로 함
- `useAuth` 훅으로 쿼리 캐시에 값을 설정하기 위해 `queryClient.setQueryData`를 사용
  - 쿼리 키와 값을 가져와 쿼리 캐시에 해당키에 대한 값을 설정할 수 있음
- `useUser` 훅에 있는 `updateUser`와 `clearUser`에 `setQueryData` 호출을 추가해야 함

### `auth/useAuth.tsx`

```tsx
import axios, { AxiosResponse } from 'axios'

import { User } from '../../../shared/types'
import { axiosInstance } from '../axiosInstance'
import { useCustomToast } from '../components/app/hooks/useCustomToast'
import { useUser } from '../components/user/hooks/useUser'

interface UseAuth {
  signin: (email: string, password: string) => Promise<void>
  signup: (email: string, password: string) => Promise<void>
  signout: () => void
}

type UserResponse = { user: User }
type ErrorResponse = { message: string }
type AuthResponseType = UserResponse | ErrorResponse

export function useAuth(): UseAuth {
  const SERVER_ERROR = 'There was an error contacting the server.'
  const toast = useCustomToast()
  const { clearUser, updateUser } = useUser()

  async function authServerCall(
    urlEndpoint: string,
    email: string,
    password: string
  ): Promise<void> {
    try {
      const { data, status }: AxiosResponse<AuthResponseType> = await axiosInstance({
        url: urlEndpoint,
        method: 'POST',
        data: { email, password },
        headers: { 'Content-Type': 'application/json' },
      })

      if (status === 400) {
        const title = 'message' in data ? data.message : 'Unauthorized'
        toast({ title, status: 'warning' })
        return
      }

      if ('user' in data && 'token' in data.user) {
        toast({
          title: `Logged in as ${data.user.email}`,
          status: 'info',
        })

        // update stored user data
        updateUser(data.user)
      }
    } catch (errorResponse) {
      const title =
        axios.isAxiosError(errorResponse) && errorResponse?.response?.data?.message
          ? errorResponse?.response?.data?.message
          : SERVER_ERROR
      toast({
        title,
        status: 'error',
      })
    }
  }

  async function signin(email: string, password: string): Promise<void> {
    authServerCall('/signin', email, password)
  }
  async function signup(email: string, password: string): Promise<void> {
    authServerCall('/user', email, password)
  }

  function signout(): void {
    // clear user from stored user data
    clearUser()
    toast({
      title: 'Logged out!',
      status: 'info',
    })
  }

  return {
    signin,
    signup,
    signout,
  }
}
```

- 로그인하거나 가입할 때 실행되는 `authServerCall`에서 `updateUser(data.user)`를 호출
  - 쿼리 캐시 값이 설정된 `user`를 인수로 가짐
- `clearUser()`는 사용자 데이터에 대한 쿼리 캐시를 `null`로 설정하므로 별도의 인수를 취하지 않음

### `user/hooks/useUser.ts`

```tsx
import { AxiosResponse } from 'axios'
import { useQuery, useQueryClient } from 'react-query'

import type { User } from '../../../../../shared/types'
import { axiosInstance, getJWTHeader } from '../../../axiosInstance'
import { queryKeys } from '../../../react-query/constants'
import { clearStoredUser, getStoredUser, setStoredUser } from '../../../user-storage'

async function getUser(user: User | null): Promise<User | null> {
  //...
}

interface UseUser {
  user: User | null
  updateUser: (user: User) => void
  clearUser: () => void
}

export function useUser(): UseUser {
  const queryClient = useQueryClient()
  const { data: user } = useQuery(queryKeys.user, () => getUser(user))

  function updateUser(newUser: User): void {
    queryClient.setQueryData(queryKeys.user, newUser)
  }

  function clearUser() {
    queryClient.setQueryData(queryKeys.user, null)
  }

  return { user, updateUser, clearUser }
}
```

- updateUser, clearUser 모두 QueryClient가 필요함
- `updateUser`

  1. useQueryClient 훅을 사용해 QueryClient 취득
  2. setQueryData 메서드 실행
  3. 설정하려는 쿼리 데이터에 대한 키를 입력(user)

  > 사용자가 성공적으로 인증되었을 경우 캐시에 사용자 정보를 업데이트

- `clearUserr`
  - 사용자가 로그아웃을 했을 경우 실행
  - `updateUser`와 마찬가지로 `queryClient`를 실행하고 `setQueryData`를 사용하지만, 값은 `null`로 설정함
- 새로고침 했을 경우 로그인 상태값이 유지되는가?
  - `localStorage`를 통해 로그인 정보 저장

## localStorage에서 사용자 데이터 유지하기

- 로그인과 로그아웃은 가능하지만 로그인한 다음 페이지를 리로드할 경우 로그인이 유지되지 않음
  - 사용자 데이터를 로컬 스토리지에 저장하고 useUser의 useQuery가 초기 실행될 때 해당 로컬 스토리지를 초기 데이터로 사용

### `components/user/hooks/useUser.ts`

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
    onSuccess: (received: User | null) => {
      if (!received) {
        clearStoredUser()
      } else {
        setStoredUser(received)
      }
    },
  })

  function updateUser(newUser: User): void {
    //...
  }

  function clearUser() {
    //...
  }

  return { user, updateUser, clearUser }
}
```

- `useUser` 함수에서 `useQuery`의 인수에 `onSuccess` 콜백으로 로컬 스토리지를 업데이트

  - `onSuccess`는 쿼리 함수나 `setQueryData`에서 데이터를 가져오는 함수
  - received(받은 데이터)가 존재 하지 않은 경우

    - `clearStoredUser` 함수를 실행해서 사용자의 데이터를 삭제

      ```tsx
      export function clearStoredUser(): void {
        localStorage.removeItem(USER_LOCALSTORAGE_KEY)
      }
      ```

  - received(받은 데이터)가 존재하는 경우

    - `setStoredUser` 함수를 실행해서 사용자의 데이터를 저장

      ```tsx
      export function setStoredUser(user: User): void {
        localStorage.setItem(USER_LOCALSTORAGE_KEY, JSON.stringify(user))
      }
      ```

### 문제점 인지

- 새로 고침시 발생하는 문제 발생 → 초기 캐시 값을 채우는 데 로컬 스토리지 값을 사용하지 않음

### 문제점 해결

- 페이지를 새로 고침할 때와 같이 useQuery가 초기화를 실행할 때 로컬 스토리지에 정의되어 있는지 확인해야함
- useQuery에서 initialData 옵션을 사용해 초기 데이터를 캐시에 추가
- `components/user/hooks/useUser.ts`

  ```tsx
  //...
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
  //...
  ```

- `getStoredUser` 함수를 실행하고 로컬 스토리지에서 JSON 형식의 데이터를 가져와 객체로 구문 분석

  ```tsx
  export function getStoredUser(): User | null {
    const storedUser = localStorage.getItem(USER_LOCALSTORAGE_KEY)
    return storedUser ? JSON.parse(storedUser) : null
  }
  ```

## 의존적 쿼리: userAppointments

### `components/user/hooks/useUserAppointments.ts`

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
    'user-appointments',
    () => getUserAppointments(user),
    { enabled: !!user }
  )

  return userAppointments
}
```

- `useUser`에서 반환되는 결과에 따라 `user`를 `User` 혹은 `null`로 판별
- `user`값이 `falsy`한 값일 경우 `null` 값을 반환하도록 하는 이유는 `user.id`가 없다면 애초에 서버에 연결을 시도하지 않도록 하기 위함임
- 쿼리 함수를 처음 실행하기 전에는 데이터가 `undefined`이므로 `fallback`을 추가한 다음 `Appointments[]`를 입력하고 빈배열로 설정
- `useUser`와 마찬가지로 `queryKey`와 `getUserAppointments` 함수를 실행하고, `user` 값이 참인지 거짓인지에 따라 `enabled`에 `boolean`값을 할당

## 쿼리 클라이언트 removeQueries 메서드

- 사용자가 로그아웃한 후에는 예약 데이터가 지워져야 함
  - 따라서 `queryClient`에 `removeQueries` 메서드를 통해 특정 쿼리에 대한 데이터를 제거해야 함
- `setQueryData`에 `null`을 사용하는 이유는 사용자 데이터를 변경해서 `onSuccess` 콜백을 발생시킬 때 `onSuccess` 콜백이 로컬 스토리지에 데이터를 유지하며 setQueryData가 `onSuccess`를 발생시키기 때문임

### `components/user/hooks/useUser.ts`

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
    queryClient.removeQueries('user-appointments')
  }

  return { user, updateUser, clearUser }
}
```

- 사용자가 로그아웃했을 때, `clearUser`가 호출되고 쿼리 데이터를 `null`로 설정해서 `onSuccess`를 트리거할 뿐만 아니라 `clearStoredUser`를 통해 로컬 스토리지로부터 사용자를 지움
- `removeQueries`를 추가로 실행해 인수에 특정 쿼리 키(`useUserAppointments` 훅에서 사용)를 전달
  - 하나 이상의 쿼리 키에 `removeQueries`를 여러 번 동일하게 실행하면 됨

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
