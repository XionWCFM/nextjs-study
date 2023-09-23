# api route

api route는 nextjs를 사용하여 공개 api를 구축하기 위한 솔루션을 제공한다고 합니다.

폴더 내의 모든 파일을 pages/api 로 매핑되며 /api/*로 처리됩니다.

즉 api route를 사용하기 위해서는 api 폴더 내에 경로를 만들어주어야합니다.

다음은 api route를 이용하는 간단한 예시 코드입니다.

```tsx
import type { NextApiRequest, NextApiResponse } from 'next'
 
type ResponseData = {
  message: string
}
 
export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<ResponseData>
) {
  res.status(200).json({ message: 'Hello from Next.js!' })
}
```

이 코드는 상태코드가 포함된 JSON Response 200을 반환합니다.

# api route의 장점


api route를 통하게 되면 cors 에러에 자유로워집니다.

이는 기본적으로 api route와 페이지들이 동일 출처라는 것을 의미하게 됩니다.

이를 통하여 cors error를 우회할 수 있습니다.


# Typescript

```tsx
export default function handler(req: NextApiRequest, res: NextApiResponse) {
  // ...
}
```

next 에서 제공해주는 타입을 사용해주면 되겠습니다.

```tsx
import type { NextApiRequest, NextApiResponse } from 'next'
 
export default function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'POST') {
    // Process a POST request
  } else {
    // Handle any other HTTP method
  }
}
```

req.method를 통해 분기를 쳐주면 리스폰스를 다르게 넣어줄 수 있습니다.

개인적으로 이건 좀 ㅇㅅㅇ...

주목할 점은 res 객체에 대해서는 <> 제네릭 문법을 통해 반환 타입을 지정해줄 수 있다는 것입니다.

```tsx
res: NextApiResponse<ResponseData>
```


# 동적 api 경로 지원

pages/api/post/[pid].ts

```tsx
import type { NextApiRequest, NextApiResponse } from 'next'
 
export default function handler(req: NextApiRequest, res: NextApiResponse) {
  const { pid } = req.query
  res.end(`Post: ${pid}`)
}
```

req.query 를 통해 쿼리의 값을 받아올 수 있습니다.

모든 경로를 포착하고 싶다면 ... 스프레드 문법을 적용하면 됩니다.

pages/api/post/[...slug].ts

```tsx
import type { NextApiRequest, NextApiResponse } from 'next'
 
export default function handler(req: NextApiRequest, res: NextApiResponse) {
  const { slug } = req.query
  res.end(`Post: ${slug.join(', ')}`)
}
```

스프레드 문법을 통해 모든 경로를 포착하는 경우

slug라는 프로퍼티에 경로들이 다음과 같은 배열 형태로 담겨 옵니다.

```tsx
{ "slug": ["a", "b"] }
```

