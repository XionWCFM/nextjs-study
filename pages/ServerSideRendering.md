# SSR

넥스트에서 서버사이드렌더링은 Dynamic Render라고도 부릅니다.

페이지에서 서버측 렌더링을 사용하는 경우에는 각 요청마다 페이지의 HTML이 생성되게 됩니다.

페이지에서 서버측 렌더링을 사용하기 위해서는 이러한 패턴으로 작성할 필요가 있는데

그게 무엇이냐면 getServerSideProps함수를 이용하는 것입니다.

예컨대 페이지에서 자주 업데이트되는 데이터를 사전 렌더링해야한다고 가정해보겠습니다.

아래는 넥스트의 예제입니다.

```tsx
export default function Page({ data }) {
  // Render data...
}
 
// This gets called on every request
export async function getServerSideProps() {
  // Fetch data from external API
  const res = await fetch(`https://.../data`)
  const data = await res.json()
 
  // Pass data to the page via props
  return { props: { data } }
}

```

getServerSideProps 라는 예약된 이름의 async 함수를 생성하고 export 해주면 됩니다.

# SSG

페이지가 SSG를 사용하는 경우에는 빌드타임에 HTML이 생성됩니다.

그리고 이는 프로덕션에서 빌드를 실행할 때 HTML이 생성된다는 것을 의미해요

이 HTML 은 각 요청에 재사용되며 CDN을 통해 캐싱할 수 있습니다.

넥스트에서는 데이터가 있거나 없는 페이지를 정적으로 생성할 수 있는데요

각각의 예시를 보면 이해가 쉽습니다.


1. 데이터가 없는 SSG

```tsx

function About() {
  return <div>About</div>
}
 
export default About

```

이 페이지는 SSG를 위해 외부 데이터를 가져올 필요가 없습니다.

따라서 넥스트는 빌드 시간동안 HTML 파일을 단일 생성합니다.


2. 데이터가 있는 SSG

```tsx
// TODO: Need to fetch `posts` (by calling some API endpoint)
//       before this page can be pre-rendered.
export default function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  )
}
```

이러한 페이지의 경우에는 사전 렌더링을 위해 외부 데이터가 필요합니다.

두가지 시나리오가 존재할 수 있는데요 하나 혹은 둘 다가 적용될 수 있습니다.

페이지 콘텐츠가 외부 데이터에 따라 다른 경우에는 getStaticProps 함수를 사용합니다.

페이지 경로가 외부데이터에 따라 다른 경우에는 getStaticPaths를 추가로 사용합니다.


그러나 중요한것은 사용자의 요청에 앞서 미리 페이지를 렌더할 수 없다면

SSG는 의미 없는 선택이 될 가능성이 높다는 것입니다.

이러한 경우에는 클라이언트측 데이터 페칭을 사용하거나 SSR 기능을 사용하는게 좋습니다.


# ISR

ISR은 증분형 정적 생성이라고 표현할 수 있습니다.

증분 정적 생성을 이용하면 사이트를 배포한 이후에도 정적 페이지가 업데이트될 수 있습니다.

또한 이것을 이용하면 전체 사이트를 다시 빌드할 필요 없이

페이지 별로 정적 생성을 사용할 수 있다는 자 ㅇ점이 있습니다.

ISR을 사용하기 위해서는 getStaticProps의 반환값에

Revalidate 프로퍼티를 추가하면 되는데요

```tsx

function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
 
// This function gets called at build time on server-side.
// It may be called again, on a serverless function, if
// revalidation is enabled and a new request comes in
export async function getStaticProps() {
  const res = await fetch('https://.../posts')
  const posts = await res.json()
 
  return {
    props: {
      posts,
    },
    // Next.js will attempt to re-generate the page:
    // - When a request comes in
    // - At most once every 10 seconds
    revalidate: 10, // In seconds
  }
}
 
// This function gets called at build time on server-side.
// It may be called again, on a serverless function, if
// the path has not been generated.
export async function getStaticPaths() {
  const res = await fetch('https://.../posts')
  const posts = await res.json()
 
  // Get the paths we want to pre-render based on posts
  const paths = posts.map((post) => ({
    params: { id: post.id },
  }))
 
  // We'll pre-render only these paths at build time.
  // { fallback: 'blocking' } will server-render pages
  // on-demand if the path doesn't exist.
  return { paths, fallback: 'blocking' }
}
 
export default Blog

```

이 예제에서 getStaticProps의 반환값에 집중해보면

revalidate 프로퍼티에 10이라는 값을 준 것을 확인할 수 있습니다.

이는 10초로 취급되게 됩니다.

