# page 라우터에서의 라우팅

사실 앱라우터에 절여져있어서 page 라우터는 대충 이렇게 동작한다라고만 알고있었다.

pages/index.js -> /

pages/blog/index.js -> blog

여기까지는 뭐 app이랑 매우 유사하다.

특이한것은 페이지라우터에서는 경로 nesting을 지원한다는 것인데

예제로보면 다음과 같다.

pages/blog/first-post.js -> blog/first-post

즉 index.js가 아닌 다른 파일명으로 pages 라우터내에 파일을 만들게되면

그 경로로 라우팅이 생성되어 버린다.

# 동적 경로 작성하기

동적 경로는 app라우터와 마찬가지로 [] 키워드를 통해 사용할 수 있다.

pages/posts/[id].js 는 posts/1 posts/2 등이 될 수 있다.

# layout 패턴

components/layout.js

```tsx
import Navbar from './navbar';
import Footer from './footer';

export default function Layout({ children }) {
  return (
    <>
      <Navbar />
      <main>{children}</main>
      <Footer />
    </>
  );
}
```

이런식으로 app router에서의 레이아웃 패턴을 사용하는 것도 가능하다.

다만 레이아웃역시 파일로 관리하는 앱라우터와는 다르게

페이지별로 레이아웃을 적용시키기 위해서는 getLayout이라는 프로퍼티가 필요하다.

pages/index.js

```tsx

import Layout from '../components/layout'
import NestedLayout from '../components/nested-layout'

export default function Page() {
  return (
    /** Your content */
  )
}

Page.getLayout = function getLayout(page) {
  return (
    <Layout>
      <NestedLayout>{page}</NestedLayout>
    </Layout>
  )
}
```

page/\_app.js

```tsx
export default function MyApp({ Component, pageProps }) {
  // Use the layout defined at the page level, if available
  const getLayout = Component.getLayout || ((page) => page);

  return getLayout(<Component {...pageProps} />);
}
```

개인적인 감상으로는 굉장히 직관적이지 않다.라는 느낌이다.

# 타입스크립트

특히 타입스크립트를 사용하는 것은 더 끔찍해보이게 만드는데

pages/index.tsx

```tsx
import type { ReactElement } from 'react';
import Layout from '../components/layout';
import NestedLayout from '../components/nested-layout';
import type { NextPageWithLayout } from './_app';

const Page: NextPageWithLayout = () => {
  return <p>hello world</p>;
};

Page.getLayout = function getLayout(page: ReactElement) {
  return (
    <Layout>
      <NestedLayout>{page}</NestedLayout>
    </Layout>
  );
};

export default Page;
```

pages/\_app.tsx

```tsx
import type { ReactElement, ReactNode } from 'react';
import type { NextPage } from 'next';
import type { AppProps } from 'next/app';

export type NextPageWithLayout<P = {}, IP = P> = NextPage<P, IP> & {
  getLayout?: (page: ReactElement) => ReactNode;
};

type AppPropsWithLayout = AppProps & {
  Component: NextPageWithLayout;
};

export default function MyApp({ Component, pageProps }: AppPropsWithLayout) {
  // Use the layout defined at the page level, if available
  const getLayout = Component.getLayout ?? ((page) => page);

  return getLayout(<Component {...pageProps} />);
}
```

그만 정신을 잃고말았다

# 마치며

예제를 보면서 느끼는 것이지만 이거 직접 해보면 에러 투성이일것 같다.

직접 예제를 만들어서 실습해봐야겠다.