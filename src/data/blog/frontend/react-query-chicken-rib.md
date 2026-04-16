---
author: "JeongHyeon"
pubDatetime: 2026-04-16T00:00:00Z
title: "계륵같은 존재 react-query"
featured: false
draft: false
tags: ["react-query", "React", "상태관리"]
description: "react-query, 쓰자니 애매하고 안 쓰자니 아쉬운 계륵 같은 존재에 대한 이야기"
---

## 시작하기 전에

오늘도 고민이 되는 친구 React Query ...

계속 API Call 하기에는 너무 비싼 것 같고 ... 그렇다고 캐싱하자니 애매하고 ...

뭔가 있으면 좋을 것 같은데 ... 넣자니 애매한 .. useCallback 같은 녀석 ...

나도 우리 백엔드 개발자가 useCallback 난사하는 것처럼 아무 생각 없이 쓰고 싶다 ...

## React Query

Google에 react-query 검색하면 가장 먼저 눈에 보이는 [React Query가 꼭 필요한 거야?](https://www.reddit.com/r/nextjs/comments/1hwpydb/is_react_query_needed/?tl=ko)

[카카오페이 프론트엔드 개발자들이 React Query를 선택한 이유](https://tech.kakaopay.com/post/react-query-1/)

너무 자극적인 문장이다.

기본적으로 React Query는 React Application에서 서버 상태를 불러오고, 캐싱하는 것을 도와주는 라이브러리다.

Redux를 사용한 복잡한 설계 없이, 네트워크 요청의 캐싱을 간단하게 처리할 수 있다.

[공식문서](https://tanstack.com/query/latest/docs/framework/react/overview)에 있는 코드를 먼저 살펴보자.

```tsx
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
} from '@tanstack/react-query'

const queryClient = new QueryClient()

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}

function Example() {
  const { isPending, error, data } = useQuery({
    queryKey: ['repoData'],
    queryFn: () =>
      fetch('https://api.github.com/repos/TanStack/query').then((res) =>
        res.json(),
      ),
  })

  if (isPending) return 'Loading...'

  if (error) return 'An error has occurred: ' + error.message

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>👀 {data.subscribers_count}</strong>{' '}
      <strong>✨ {data.stargazers_count}</strong>{' '}
      <strong>🍴 {data.forks_count}</strong>
    </div>
  )
}
```

| 이름 | 설명 |
|------|------|
| `QueryClient` | React Query의 캐시와 설정을 관리하는 핵심 인스턴스. 모든 쿼리의 캐시 데이터가 여기에 저장된다. |
| `QueryClientProvider` | `QueryClient`를 하위 컴포넌트에 주입하는 Context Provider. 앱 최상단에 한 번 감싸주면 된다. |
| `useQuery` | 데이터를 조회(GET)할 때 사용하는 훅. 로딩 상태, 에러, 데이터를 한 번에 반환해준다. |
| `queryKey` | 쿼리를 식별하는 고유 키. 이 키를 기준으로 캐싱, 리패칭, 무효화가 이루어진다. |
| `queryFn` | 실제 데이터를 가져오는 비동기 함수. `fetch`, `axios` 등 원하는 방식으로 작성하면 된다. |
| `isPending` | 쿼리가 아직 데이터를 불러오는 중인지 나타내는 boolean 값. |
| `error` | 쿼리 실행 중 에러가 발생하면 해당 에러 객체가 담긴다. |
| `data` | `queryFn`이 성공적으로 반환한 데이터가 담긴다. |
| `staleTime` | 데이터가 "신선하다"고 간주되는 시간(ms). 이 시간 동안은 리패칭이 발생하지 않는다. 기본값은 `0`. |
| `gcTime` | 비활성 쿼리가 캐시에 남아있는 시간(ms). 이 시간이 지나면 가비지 컬렉션된다. 기본값은 `5분`. |

보면 사용 방법도 너무 간단하다.

1. 전역에 Provider 최상단에 넣어주고
2. `useQuery`에서 `queryKey`에 키값과 `queryFn` 함수 넣어주고
3. `staleTime`, `gcTime` 등으로 캐싱 시간을 설정해주면 끝이다.

빈번하게 호출하는 API 통신을 캐싱하는 것은 자연스럽게도 고민해야 하는 일이다.

게다가 React Query는 React 생태계를 경험한 개발자에게 익숙한 Hook을 이용하여 사용할 수 있는 방법을 제안한다.

코드에서 볼 수 있듯이, `queryKey`와 일치하는 요청에 대한 응답 데이터를 캐싱한다.

- **어떤 데이터를?** `queryFn`에서 반환하는 `data`를
- **얼마나?** `staleTime`과 `gcTime`에서 설정한 시간만큼

### Boilerplate 감소

React Query를 사용하면 Boilerplate, 즉 재사용되는 표준화된 코드량이 감소한다.

기존의 Redux + redux-saga가 가장 유명한 조합일 테고, 이후 RTK Query, Zustand 등 다양한 방법이 있겠지만, React Query 도입으로 지옥 같은 Redux 상태 관리를 위해 작성하던 코드를 작성할 필요가 줄어든다.

이는 코드의 복잡도를 감소시키고 유지보수성을 높여준다. 이를 통해 개발자가 복잡하게 작성하는 일도 줄어들고, 크게 보면 휴먼 에러로 발생하는 일도 줄어들게 될 거고, 잘못된 설계로 힘들어할 일도, 관련된 여러 이슈들을 감소시킬 수 있다.

## 근데 왜 계륵일까

내 주관적인 생각이다.

### 1. 사용하기가 애매하다

캐싱을 통해서 사용자에게 서버 <-> 화면 통신하는 대신 메모리 캐시를 하여 보여주면 UX는 최고인 것 같다. 예를 들어 앱을 백그라운드로 내렸다가 다시 올리게 됐을 때 캐시된 데이터를 보여주면 깜빡임이 최소화될 것이다. 근데 이때의 데이터는 **"과거의 데이터"** 가 될 것이다.

내가 만들었던 여러 커뮤니티들이나, 주문 시스템에서도, 하드웨어 제어 설비 시스템에서도 캐싱을 통해 정보를 보여줄 경우 치명적이라고 생각했다.

내가 예상하지 못했던 캐시 타이밍, 그를 위한 수동 동기화 타이밍을 계산하는 불편함이 React Query가 주는 편안함보다 더 큰 것 같다.

데이터 정합성이 깨지는 순간 그때부터는 겉잡을 수 없이 문제가 커진다. 그렇기 때문에 데이터의 최신화, 서버와의 동기화된 데이터 상태가 더 가치가 크다고 생각한다.

### 2. 캐시 키 관리와 무효화, 결국 고민은 비슷하다

Redux나 상태 관리를 내가 직접 하지 않지만, 캐시 키 관리와 캐시 무효화 관련된 상황을 고려하는 건 힘든 건 비슷비슷하다.

1과 이어지는 이야기인데, 초기 설계 코드가 줄어드는 건 정말 큰 장점이라고 생각한다. 근데 "해당 데이터가 어느 시점까지 가비지 데이터가 아닌가?"에 대해서 시간을 설정해주고, 컴포넌트들과 캐싱 데이터들의 관계, 캐싱한 데이터들과 또 다른 데이터들의 관계, 타이밍, 무효화를 잘못하면 내가 생각하지 못했던 옛날 데이터로 인해 발생하는 정합성이 깨지는 끔찍한 상황들을 고민하는 거랑, 어떤 걸 고민하느냐의 차이지 힘든 건 동일한 것 같다.

## 마치며

React Query가 나쁘다고는 **"절대"** 생각하지 않는다. 내가 아직 필요한 상황을 겪어보지 못했다고 생각한다.

그렇기 때문에 나에게는 계륵 같은 존재이다. 이론적으로는 정말 좋은 라이브러리고, 많은 사랑을 받는 데는 이유가 있다고 분명 생각한다.

하지만 분명 무분별하게 사용할 순 없고, 오히려 성능을 악화시키게 될 것이다. 잘 알고 사용하는 게 중요하다고 생각한다.

우리 회사에는 "모든 함수를 useCallback"으로 감싸서 캐싱하려는 나쁜 개발자가 있다.

삭제 함수 같은 건 왜 거는지 모르겠지만 이런 게 오버엔지니어링이 될 거고, 성능이 최적화되지 않는다고 생각한다.

좀 더 고민해보고, 사용성을 높이기 위해선 캐싱 정책은 잘 수립하고 가야 하기 때문에 React Query를 도입하는 날, 더 좋은 글과 함께 한층 더 성장한 모습으로 돌아오고 싶다.
