# reactquery의 useQuery 동작 원리

우리는 흔하게 react-query를 사용하여 비동기 상태를 관리합니다.

react-query는 로딩, 성공, 실패 등의 상태를 개발자가 직접 관리하는 것에서 벗어나

선언적으로 상태를 관리할 수 있도록 도와주며

stale, cache 등의 개념을 도입하는 것을 통해 불필요한 api 요청을 최소화시키는 일을 수행합니다.

그러나 모든 라이브러리가 그렇듯 매직 코드는 존재하지 않습니다.

react-query 역시도 자바스크립트로 쓰여진 코드의 뭉치일 뿐이라고 볼 수 있습니다.

이번에는 react-query의 useQuery에 대해 알아봅니다.

# 4개의 주요한 객체

useQuery를 분석하기에 앞서 긴밀하게 상호작용하는 4개의 객체가 존재합니다.

순서대로 나열하면 다음과 같습니다.

1. QueryClient

2. QueryCache

3. Query

4. QueryObserver

## QueryClient

QueryClient는 context api를 통하여 전역적으로 사용할 수 있는 객체입니다.

우리는 흔히

```tsx
const queryClient = new QueryClient();
```

와 같은 클래스 생성자를 이용하여 쿼리클라이언트 인스턴스를 생성하고

이를 QueryClientProvider에게 전달합니다.

```tsx
export class QueryClient {
  private queryCache: QueryCache
  ...

  constructor(config: QueryClientConfig = {}) {
    this.queryCache = config.queryCache || new QueryCache()
    ...
  }
}
```

그런데 이 QueryClient는 생성되는 시점에 QueryCache 객체를 가지게 되는 것을 볼 수 있습니다.

## QueryCache

```tsx
export class QueryCache extends Subscribable<QueryCacheListener> {
  config: QueryCacheConfig

  private queries: Query<any, any, any, any>[]
  private queriesMap: QueryHashMap

  constructor(config?: QueryCacheConfig) {
    super()
    this.config = config || {}
    this.queries = []
    this.queriesMap = {}
  }
  ...
  add(query: Query<any, any, any, any>): void {
    if (!this.queriesMap[query.queryHash]) {
      this.queriesMap[query.queryHash] = query
      this.queries.push(query)
      this.notify({
        type: 'added',
        query,
      })
    }
  }
  ...
}

```

queryCache 객체는 쿼리 객체들을 저장하는 역할을 수행합니다.

이때 위 코드에서는 생략되었지만 useQuery()의 인자로 넘겨준 queryKey를 이용하여

queryCache 객체는 queryHash를 만들며 이 queryHash 를

queriesMap의 key로 , Query 객체를 value로 하여 저장합니다.

바로 이 작업을 통하여 리액트 쿼리는 쿼리키값을 기반으로 요청이 동일한 것인지 / 다른 요청인지를 판단합니다.

## Query

```tsx
export class Query<...> extends Removable {
  ...

  constructor(config: QueryConfig<TQueryFnData, TError, TData, TQueryKey>) {
    super()
    ...
    this.observers = []
    this.cache = config.cache
    this.state = this.initialState
    ...
  }
  ...
}

```

이 Query 객체는 내부적으로 자신의 상태가 변경되었을 때 호출할 옵저버들을 가지게 되는데요

구독 요청이 들어오면 this.observers에 옵저버가 추가되고

상태가 변경되었을때 등록된 옵저버들을 호출하는 형태로 전역 상태의 변경을 "발행"합니다.

이러한 패턴을 pub-sub 패턴이라고도 불러요

## QueryObserver

```tsx
export class QueryObserver<...> extends Subscribable<QueryObserverListener<TData, TError>> {
  ...
  constructor(
    client: QueryClient,
    options: QueryObserverOptions<...>
  ) {
    super()

    this.client = client
    ...
    this.trackedProps = new Set()
    ...
  }
  ...
  protected onSubscribe(): void {
    if (this.listeners.length === 1) {
      this.currentQuery.addObserver(this)

      if (shouldFetchOnMount(this.currentQuery, this.options)) {
        this.executeFetch()
      }
      ...
    }
  }
  ...
  private updateQuery(): void {
    const query = this.client.getQueryCache().build(this.client, this.options)

    if (query === this.currentQuery) {
      return
    }

    this.currentQuery = query
    ...
  }
}

```

마지막 QueryObserver 객체는 QueryClient와 Query 객체를 가집니다.

