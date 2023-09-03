# nextjs의 렌더링

nextjs를 사용하면서 가장 많이 만나게 되는 에러는

서버사이드에서 발생하는 에러입니다.

예컨대 기존 리액트만을 이용하여 개발을 진행할 때에는

window 객체에 접근하기 , web api 사용하기 등의 동작을

별도의 분기처리없이도 자연스럽게 사용할 수 있지만

nextjs를 사용할 때에는 문제가 발생하곤 합니다.

예컨대 `window is not defined`와 같은 에러가 발생할 수 있습니다.

이는 nextjs의 일부 동작이 서버사이드에서 진행되는 것에 기인하는 문제입니다.


# 토스 기술블로그의 예제

```tsx
function App() {
    const name = new URL(location.href).searchParams.get(name)
    return <div>{name}</div>

}

```
이 코드는 지금까지 리액트에 익숙한 개발자의 관점에서 문제의 여지가 없는 코드입니다.

그러나 SSR 환경에서는 이 코드를 실행시키면 에러가 발생합니다.

`location is not defined`

브라우저 객체 location이 존재하지 않는다는 에러입니다.

그 아래에는 This error happend while generating the page

라는 에러를 찾아볼 수 있습니다.

위 두 에러메시지를 기반으로 이 에러는

브라우저가 아닌 환경에서 HTML 생성을 하는 도중 발생한 에러라는 것을 유추할 수 있고

따라서 서버에서 발생한 에러라는 추론이 가능합니다.

# 위 문제의 근본적인 원인

이는 클라이언트 사이드에만 존재하는 코드가 있음을 개발자가 알고있어야 한다.

라는 사실을 시사합니다.

따라서 서버환경에서는 location에 접근할 수 없도록 코드를 수정해야하며

이는 다른 web api나 window 객체를 사용할 때에도 동일하게 적용됩니다.

# 해결 방법

그러나 이를 해결하기 위해 분기처리를 두는것만으로는 문제가 해결되지 않습니다.

또 다른 에러로 하이드레이션 에러가 발생하는데요

이 에러는 서버사이드와 클라이언트사이드의 렌더링 결과가가 다르기 때문에

발생하는 에러입니다.


<br/>

서버에서 생성한 HTML은 JS가 없는 단순 마크업이기 때문에 인터랙션이 불가합니다.

따라서 리액트는 이벤트 리스너, 상태와 같은 클라이언트 로직과

HTML을 통합하는 것을 통해 애플리케이션으로 동작할 수 있도록 합니다.

<br/>

그런데 주의할 점은 로직의 연결 과정입니다.

리액트는 요소와 로직 정보가 담긴 가상 돔을 생성한 후

이것을 전달받은 HTML과 비교하며 둘의 결과물이 같은 경우에만

하이드레이션을 수행할 수 있습니다.

그렇기 때문에 서버와 클라이언트의 렌더링 결과를 동일하게 보장해야만 합니다.


<br/>

그렇게하기 위해 Isomorphic이라는 개념이 등장합니다.

하이드레이션 미스매치 에러를 방지하기 위한 방법인것이지요

next.js의 경우에는 이런 쿼리와 관련된 경우의 불편을 해소하기 위해

useRouter 함수를 제공하고 있으며 이를 통헤 항상 동일한 결과를 보장합니다.

# next의 pre-rendering

리액트는 CSR을 기반으로 빈 index.html을 전달받은 뒤 JavaScript 번들 다운로드가 완료되면

자바스크립트를 이용하여 렌더링을 진행한 후 한번에 화면을 보여줍니다.

이는 사용자에게 의미있는 첫화면을 보여주는 시간을 길게 늘린다는 문제가 있습니다.

반면 next.js는 모든 페이지를 pre-rendering 해둡니다.

이것은 각 페이지의 HTML을 미리 생성해두는 것을 의미하며

이 HTML은 해당 페이지에 필요한 최소한의 자바스크립트 코드와 연결됩니다.

그 이후 페이지가 브라우저에 의해 로드되면 자바스크립트 코드가 실행되고

유저가 페이지에서 상호작용을 할 수 있어지는 것입니다.

# SSG와 SSR

Static site generation은 pre-render라고 생각해도 될 것 같습니다.

미리 정적인 페이지를 생성해두는 개념이며 바뀌지 않는 페이지를 SSG로 작성합니다.

반면 SSR은 사용자마다 다른 화면이 보여야하는 장바구니와 같은 경우 사용합니다.

# 렌더링 순서

우선 렌더링은 일반적으로 서버 -> 클라이언트 순으로 진행된다.

서버의 렌더링 흐름은 다음과 같다.


## 서버 렌더링 흐름

server/render.tsx 

```tsx
export async function renderToHTML(
  req: IncomingMessage,
  res: ServerResponse,
  pathname: string,
  query: NextParsedUrlQuery,
  renderOpts: RenderOpts
): Promise<RenderResult | null> {
  // ...

  const renderDocument = async () => {
    // ...
    async function loadDocumentInitialProps(
      renderShell?: (_App: AppType, _Component: NextComponentType) => Promise<ReactReadableStream>
    ) {
      // ...
      const renderPage: RenderPage = (
        options: ComponentsEnhancer = {}
      ): RenderPageResult | Promise<RenderPageResult> => {
        // ...
        const html = ReactDOMServer.renderToString(
          <Body>
            <AppContainerWithIsomorphicFiberStructure>
              {renderPageTree(EnhancedApp, EnhancedComponent, {
                ...props,
                router,
              })}
            </AppContainerWithIsomorphicFiberStructure>
          </Body>
        );
        return { html, head };
      };
    }
    // ...
    return {
      bodyResult,
      documentElement,
      head,
      headTags: [],
      styles,
    };
  };
}

```

이 코드의 renderPage 함수 부분에서 HTML을 생성한다

renderPage 함수의 반환값을 보면

`Promise<RenderPageResult> | RenderPageResult` 이다.

또한 자체적으로 정의한 renderPage 함수는

내부적으로 ReactDOMServer.renderToString()메서드를 이용하고 있다.

이를 통하여 next.js는 ReactNode를 HTML 문자열로 변환한다.


## 클라이언트

client/next.js
```tsx
initialize({})
.then(() => hydrate())
.catch(console.error)
```

먼저 초기화를 진행 한 후 하이드레이션을 실행하는 것으로 이해할 수 있다.

client/index.tsx
```tsx
export async function initialize(opts: { webpackHMR?: any } = {}): Promise<{
  assetPrefix: string;
}> {
  initialData = JSON.parse(document.getElementById('__NEXT_DATA__')!.textContent!);
  window.__NEXT_DATA__ = initialData;

  const prefix: string = initialData.assetPrefix || '';

  appElement = document.getElementById('__next');
  return { assetPrefix: prefix };
}

```
초기화를 담당하던 initialize 코드의 내부구현은 index.tsx에서 찾을 수 있다.

이는 전역객체 window의 프로퍼티에 값을 할당해두고

운영환경에 따라 assetPrefix를 반환한다는 것을 알 수 있다.

그럼 initialize 이후 실행되는 hydrate는 어떻게 구현되어 있을까?

```tsx
export async function hydrate(opts?: { beforeRender?: () => Promise<void> }) {
  // ...
  const renderCtx: RenderRouteInfo = {
    App: CachedApp,
    initial: true,
    Component: CachedComponent,
    props: initialData.props,
    err: initialErr,
  };

  render(renderCtx);
}

```
hydrate는 유효성 검증을 한 뒤 렌더링에 필요한 컨텍스트를

render 함수에 넘겨주고 render 함수를 호출한다.

실질적인 렌더링은 render 함수에서 진행될것을 예상할 수 있을 듯하다.

```tsx
async function render(renderingProps: RenderRouteInfo): Promise<void> {
  // ...
  await doRender(renderingProps);
}

function doRender(input: RenderRouteInfo): Promise<any> {
  // ...
  renderReactElement(appElement!, (callback) => (
    <Root callbacks={[callback, onRootCommit]}>
      {process.env.__NEXT_STRICT_MODE ? <React.StrictMode>{elem}</React.StrictMode> : elem}
    </Root>
  ));
}

```
render 함수는 renderReactElement 함수를 실행하고 있다.

그리고 strictmode 체크여부에 따라 간단한 분기처리도 실행하고있는 모습을

확인할 수 있다.

```tsx
let shouldHydrate: boolean = true; // 첫 렌더에서는 항상 true이다

function renderReactElement(domEl: HTMLElement, fn: (cb: () => void) => JSX.Element): void {
  //...
  const reactEl = fn(shouldHydrate ? markHydrateComplete : markRenderComplete);

  // ...
  if (shouldHydrate) {
    ReactDOM.hydrate(reactEl, domEl);
    shouldHydrate = false;
  } else {
    ReactDOM.render(reactEl, domEl);
  }
}

```

드디어 우리에게 익숙한 ReactDOM 코드가 등장했다.

shouldHydrate 값의 상태에 따라 하이드레이션을할지 렌더를 할지 결정하는 모습이다.

# 마치며


렌더링 과정에 대한 내용은 2022년에 작성된 블로그글을 토대로 작성한 것이다.

따라서 현재와는 내부구현이 조금 다를 여지가 존재한다.

대표적으로 ReactDOM.hydrate의 경우에도 현재는 deprecated 된 상태이다.

하지만 어느정도의 흐륾을 이해하는데에는 유용하다고 판단하였다.



# 레퍼런스

https://toss.tech/article/22443

https://www.howdy-mj.me/next/hydrate