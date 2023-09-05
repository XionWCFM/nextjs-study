# getServerSideProps

함수이름에서 쉽게 예상할 수 있듯이 페이지 라우터에서

서버측렌더링 ssr을 해야하는 컴포넌트라는 것을 알리는 함수입니다.

주로 데이터페칭 등의 작업을 위해 사용하게되며

getServerSideProps에서 반환한 값을 해당 페이지의 prop으로 전달받아 사용합니다.

<br/>

getServerSideProps는 각 요청에 대해 페이지를 미리 렌더링하며

자주 변경되는 데이터를 가져오고 페이지를 업데이트하고자 하며

항상 최신 데이터를 사용자에게 제시해야하는 경우 유용합니다.

# 예제를 보며 학습

```tsx

import type { InferGetServerSidePropsType, GetServerSideProps } from 'next'
 
type Repo = {
  name: string
  stargazers_count: number
}
 
export const getServerSideProps: GetServerSideProps<{
  repo: Repo
}> = async () => {
  const res = await fetch('https://api.github.com/repos/vercel/next.js')
  const repo = await res.json()
  return { props: { repo } }
}
 
export default function Page({
  repo,
}: InferGetServerSidePropsType<typeof getServerSideProps>) {
  return repo.stargazers_count
}
```

공식문서의 타입스크립트 예제를 통하여 많은 것을 배울 수 있습니다.

InferGetSErverSidePropsType 타입을 통하여 prop의 타입을 손쉽게 설정 가능합니다.

app 라우터의 서버컴포넌트에서 fetch 함수를 사용하는 것을 상상하면 이해가 쉽습니다만

페이지 라우터에서는 페이지 내부에서 직접 fetch를 사용하는 것이 아니라

getServerSideProps 내부에서 데이터페칭을 수행한다는 것에 유의하셔야합니다.

<br/>

반환 값은 `{props: {}}`의 형태로 반환하는 것이 강제됩니다.

또한 이 prop의 값은 직렬화 가능한 객체여야만 한다는 제약이 존재합니다.


# notFound 반환 가능

```tsx
export async function getServerSideProps(context) {
  const res = await fetch(`https://.../data`)
  const data = await res.json()
 
  if (!data) {
    return {
      notFound: true,
    }
  }
 
  return {
    props: { data }, // will be passed to the page component as props
  }
}
```

notFound : boolean 을 반환하는 것을 통해

이전에 성공적으로 생성한 페이지가 있더라도 404notFoundPage를 반환시킬 수 있습니다.

이것은 작성자가 사용자 생성 콘텐츠를 제거하는 등의 사용 사례를 지원하기 위한 옵션이에요

# redirect

```tsx
export async function getServerSideProps(context) {
  const res = await fetch(`https://.../data`)
  const data = await res.json()
 
  if (!data) {
    return {
      redirect: {
        destination: '/',
        permanent: false,
      },
    }
  }
 
  return {
    props: {}, // will be passed to the page component as props
  }
}
```

리다이렉트 객체를 이용하여 내부, 외부 리소스로 리디렉션 할 수 있습니다.

이 리다이렉트를 사용할 때에는 항상 위와 같은 구조의 객체를 반환해야만 합니다.

# 마치며

솔직히 데이터 페칭에 관한 부분은 서버사이드프랍보다 앱라우터가 더 간결하고 좋다는 생각이 듭니다.

하지만 서버사이드프랍 역시 러닝커브가 그리 험난하진 않으니 쉽게 익힐 수 있을 것입니다.