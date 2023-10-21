```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

export function wrapper() {
  const testQueryClient = new QueryClient();
  // eslint-disable-next-line react/display-name
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={testQueryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

리액트 쿼리 테스트를 위해서는 우선 프로바이더를 모킹해줘야한다.

방법은 어렵지 않은데 이런식으로 작성해주면 되겠다.

이렇게 잘 작성했다면 가장 간단한 동작을 하는 쿼리를 테스트해보자

```tsx
import { useQuery } from '@tanstack/react-query';

export function useCustomHook() {
  return useQuery({ queryKey: ['customHook'], queryFn: () => 'Hello' });
}
```

아주 심플하게 hello를 반환하는 함수가 qeuryFn으로 들어간 useQuery 훅이다

실전에서는 당연하게도 써먹지 못할 코드이지만

첫 시작을 해보는것이 중요하다.

잘 작성했다면 이제 테스트코드를 작성해보자

```tsx
import { render, renderHook, waitFor } from '@testing-library/react';
import { useCustomHook } from './use-fetch-data';
import { wrapper } from '@/__mocks__/react-query';
test('react-query test', async () => {
  const { result } = renderHook(() => useCustomHook(), {
    wrapper: wrapper(),
  });

  await waitFor(() => expect(result.current.isSuccess).toBe(true));
});
```

위에서 만들어뒀던 wrapper 함수를 wrapper 옵션에 전달하고

테스트를 진행해본다.
