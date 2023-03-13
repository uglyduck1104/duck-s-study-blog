---
title: 쿼리 특성 1. 프리페칭(Pre-fetching)과 페이지 매김
date: '2023-03-13'
tags: ['react']
draft: false
summary: 캐시에 데이터 추가하기, 프리페칭(Pre-fetching) 처리 - 개념, 프리페칭(Pre-fetching) 처리 - 구문, useAppointments를 위한 useQuery - 실습 코드, 의존성 배열로서의 쿼리 키  
layout: PostSimple
authors: ['default']
---

# 쿼리 특성 1. 프리페칭(Pre-fetching)과 페이지 매김

## 캐시에 데이터 추가하기

|                 | 사용되는 곳                | 데이터 출처 | 캐시 추가 유무 |
|-----------------|-----------------------|--------|----------|
| prefetchQuery   | method to queryClient | server | Y        |
| setQueryData    | method to queryClient | client | Y        |
| placeholderData | option to useQuery    | client | N        |
| initialData     | option to useQuery    | client | Y        |
- 일반적으로 사용자에게 보여주고 싶은 정보가 있을 때 캐시에 데이터가 없는 경우 미리 채워 넣을 수 있음

### prefetchQuery

- `queryClient`에 대한 메서드
- 데이터는 서버에서 오기 때문에 데이터를 가져오기 위해 서버로 이동하고 데이터는 캐시에 추가됨

### setQueryData

- `useQuery`를 실행하지 않고 쿼리 데이터를 캐시에 추가하는 방법
- `queryClient`에 대한 메서드
- 클라이언트에서 데이터를 가져오므로 서버에서 변이(`mutation`)에 대한 응답으로 나온 데이터
- `queryClient`에서 `setQueryData` 메소드를 사용하여 캐시에 데이터를 추가하면 `useQuery`가 데이터를 요청할 때 캐시가 해당 데이터를 제공하도록 할 수 있음

### placeholderData

- `useQuery`를 실행할 때 데이터를 제공하기 때문에 클라이언트에서 데이터를 가져오고 캐시에는 추가되지 않음
- 고정값 또는 함수를 사용하고, 자리 표시자 데이터값을 동적으로 결정하는 함수를 사용하는 경우 사용하는 것을 권장

### initialData

- `useQuery`에 대한 옵션으로써 클라이언트에서 제공됨
- `placeholderData`와 달리 캐시에 추가해야 하는 데이터로써, 쿼리에 대한 유효한 데이터임을 선언해 둘 필요가 있음

## 프리페칭(Pre-fetching) 처리(개념)

- 사용자가 전체 페이지를 로드할 때, 특정 액션을 할 때 기다릴 필요가 없이 특정 데이터를 미리 가져옴
- 캐시 시간 내에 `useQuery`로 데이터를 호출하지 않으면 가비지 컬렉션으로 수집됨
- `prefetchQuery`는 `queryClient`의 메서드이므로 `useQuery`와 달리 클라이언트 캐시에 추가됨
- `useQuery`는 페칭과 리페칭 등의 작업이 필요한 쿼리를 생성하지만 `prefethQuery`는 일회성임

## 프리페칭(Pre-fetching) 처리(구문)

### `components/treatements/hooks/useTreatements.ts`

```tsx
import { useQuery, useQueryClient } from 'react-query';

import type { Treatment } from '../../../../../shared/types';
import { axiosInstance } from '../../../axiosInstance';
import { queryKeys } from '../../../react-query/constants';

async function getTreatments(): Promise<Treatment[]> {
  const { data } = await axiosInstance.get('/treatments');
  return data;
}

export function useTreatments(): Treatment[] {
  const fallback = [];
  const { data = fallback } = useQuery(queryKeys.treatments, getTreatments);
  return data;
}

export function usePrefetchTreatments(): void {
  const queryClient = useQueryClient();
  queryClient.prefetchQuery(queryKeys.treatments, getTreatments);
}
```

- 캐시를 채우는 것이 목적이므로 아무것도 반환하지 않아도 됨(`void`)
- React Query에서 useQueryClient 훅을 호출해 쿼리 공급자에서 프로퍼티로 사용한 queryClient가 반환됨
  - queryClient를 가져온 후 prefetchQuery를 실행
  - 쿼리 키는 캐싱에서 어느 useQuery가 해당 데이터를 찾아야 하는지 알려주므로 매우 중요함

### `components/app/Home.tsx`

```tsx
import { Icon, Stack, Text } from '@chakra-ui/react';
import { ReactElement } from 'react';
import { GiFlowerPot } from 'react-icons/gi';

import { BackgroundImage } from '../common/BackgroundImage';
import { usePrefetchTreatments } from '../treatments/hooks/useTreatments';

export function Home(): ReactElement {
  usePrefetchTreatments();

  return (
    <Stack textAlign="center" justify="center" height="84vh">
      <BackgroundImage />
      <Text textAlign="center" fontFamily="Forum, sans-serif" fontSize="6em">
        <Icon m={4} verticalAlign="top" as={GiFlowerPot} />
        Lazy Days Spa
      </Text>
      <Text>Hours: limited</Text>
      <Text>Address: nearby</Text>
    </Stack>
  );
}
```

- 방금 작성한 훅을 실행하기 위해 `import` 구문을 통해 실행
- `Home` 컴포넌트의 모든 렌더링에서 실행하는 이유는 `Home` 컴포넌트는 동적이지 않으므로 리렌더가 많이 발생하지 않음
  - `staleTime`과 `cacheTime`을 관리해 줄 몇 가지 옵션을 통해 여러번 실행될 경우를 방지할 수 잇음
- 개발자 도구는 로딩 속도와 상관없이 쿼리 캐시에서 어떤 작업이 진행 중인지 알 수 있게 함

## useAppointments를 위한 useQuery(실습 코드)

### `appointments/hooks/useAppoinments.ts`

```tsx
// @ts-nocheck
import dayjs from 'dayjs';
import { Dispatch, SetStateAction, useState } from 'react';
import { useQuery } from 'react-query';

import { axiosInstance } from '../../../axiosInstance';
import { queryKeys } from '../../../react-query/constants';
import { useUser } from '../../user/hooks/useUser';
import { AppointmentDateMap } from '../types';
import { getAvailableAppointments } from '../utils';
import { getMonthYearDetails, getNewMonthYear, MonthYear } from './monthYear';

// useQuery 호출
async function getAppointments(
  year: string,
  month: string,
): Promise<AppointmentDateMap> {
  const { data } = await axiosInstance.get(`/appointments/${year}/${month}`);
  return data;
}

interface UseAppointments {
  appointments: AppointmentDateMap;
  monthYear: MonthYear;
  updateMonthYear: (monthIncrement: number) => void;
  showAll: boolean;
  setShowAll: Dispatch<SetStateAction<boolean>>;
}

export function useAppointments(): UseAppointments {
  const currentMonthYear = getMonthYearDetails(dayjs());

  const [monthYear, setMonthYear] = useState(currentMonthYear);

  function updateMonthYear(monthIncrement: number): void {
    setMonthYear((prevData) => getNewMonthYear(prevData, monthIncrement));
  }

  const [showAll, setShowAll] = useState(false);

  const { user } = useUser();

  const fallback = {};
  const { data: appointments = fallback } = useQuery(
    queryKeys.appointments,
    () => getAppointments(monthYear.year, monthYear.month),
  );

  return { appointments, monthYear, updateMonthYear, showAll, setShowAll };
}
```

- `useQuery`를 실행하고 반환되는 데이터 구조를 분해
- 데이터의 이름을 `appointments`로 변경해서 반환 값을 가져올 수 있게 한 후 기본 값을 `fallback`으로 설정

```tsx
 const { data: appointments = fallback } = useQuery(
    queryKeys.appointments,
    () => getAppointments(monthYear.year, monthYear.month),
  );
```

- `useQuery`는 쿼리 키를 사용하므로 `queryKeys.Appointments`를 사용한 후 쿼리 함수를 사용해 `year`와 `month`를 취함
  - 현재 상태인 `monthYear`를 넣어주고 `year` 프로퍼티를 입력한 다음 `monthYear`와 `month`를 입력

## 의존성 배열로서의 쿼리 키

### 문제점 파악

- 위의 useQuery를 구현한 방식에서 monthYear가 변경될 때 데이터가 변경되지 않음

### 이유

- 모든 쿼리는 동일한 키를 사용함
  - `queryKeys.Appointments`를 사용하므로 이전 달이나 다음 달로 이동하기 위해 버튼을 누르면 쿼리 데이터는 만료(stale) 상태이지만 리페치를 트리거할 대상이 존재하지 않음
  - 리페치를 트리거하면 컴포넌트를 다시 마운트하거나 창을 다시 포커할 수도있고 리페치 함수를 수동으로 실행할 수 있음
- useQuery가 반환하는 객체에 refetch 함수가 포함되어 있고, 자동 리페치도 수행할 수 있음

> 항상 키는 의존성 배열로 취급해야 하고, 데이터가 변경될 경우 키도 변경되도록해야 새 쿼리를 만들고 새 데이터를 가져올 수 있음
>

```tsx
const { data: appointments = fallback } = useQuery(
  [queryKeys.appointments, monthYear.year, monthYear.month],
  () => getAppointments(monthYear.year, monthYear.month),
);
```

- 우선 `Appointments` 배열의 첫 번째 항목으로 유지해야 함
  - `queryKeys.appointments`
  - 데이터를 무효화하려면 모든 배열에 공통점이 있어야 하는데, 특히 공통 접두사가 있으면 한 번에 모두 무효화할 수 있음
- 나머지 `monthYear` 별로 고유한 배열을 넣음

> **Referenced**

- [React Query: React Server State Management](https://www.udemy.com/course/learn-react-query/)
