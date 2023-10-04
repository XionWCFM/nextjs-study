# useRouter

useRouter훅은 클라이언트 컴포넌트에서만 사용할 수 있습니다.

특정한 요구사항이 없는 경우에는 Link 컴포넌트를 사용하는 것이 권장됩니다.

```tsx
'use client';

import { useRouter } from 'next/navigation';

export default function Page() {
  const router = useRouter();

  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  );
}
```

일반적인 사용례는 다음과 같습니다.

useRouter훅을 호출한뒤 메서드를 사용합니다.

여러가지 기능이 있는데 주목할만한 것이 하나 있습니다.

바로 app router에서는 똑같은 이름의 useRouter이지만 가져오는 경로가 다르단 것인데요

app router에서는 next/navigation 경로에서 가져와야합니다.

이전 페이지 라우터 시절에는 next/router 경로에서 가져왔었지요
