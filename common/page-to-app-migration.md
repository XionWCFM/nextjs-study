# 점진적 채택을 위해선 어떻게 해야할까

우선 공식문서를 참고해봤을 때 버전을 업데이트하는 방법은 다음과 같다.

```
npm install next@latest react@latest react-dom@latest
npm install -D eslint-config-next@latest
```

# 크게 차이가 나는 부분

## _app.js , _document.js

이 둘은 앱라우터에서 단일 파일로 대체되며 layout.js로 대체된다.

## _error.js

이녀석은 error.js로 대체되었다.

## 404.js

이녀석은 not-found.js로 대체되었다.

그 외에 다행인점은 api 를 이용한 서버기능들은 아직 pages 디렉토리를 쓴다는것이다.


# 마이그레이션 가이드

우선 앱폴더와 루트레이아웃을 정의해주는 것 부터 시작한다.

app/layout.tsx를 생성하고

```tsx
export default function RootLayout({
    chidren: {
        children:React.ReactNode
    }
}) {
    return (
        <html lang="en">
            <body>{children}</body>
        </html>
    )
}
```

이렇게 생성해준 루트레이아웃은 _app.tsx와 _document.tsx를 대체한다.

그런데 저 두 파일은 무슨일을 하는 것일까?

_app은 서버로 요청이 들어왔을때 가장 먼저 실행되며

페이지에 적용할 공통 레이아웃의 역할을 수행한다.

_document는 _app 다음에 실행되며

메타태그를 커스텀하거나 body 태그안의 내용들을 커스텀할 때 사용한다.

특이한점은 이 두파일이 server only 파일이라는 것인데

따라서 당연하게도 window, dom 은 사용할 수 없다.


_app.js는 공통으로 수행되어야할 애플리케이션 로직을 수행하고

_document는 html 마크업 같은 부분을 수행한다.

확실히 합쳐줘도 무방해보이는 로직들이긴하다.


# 마치며

그 외에는 이미 app router에 어느정도 익숙한 나로서는 그리 어렵지 않은 내용들이었다.

다만 실제로는 로직들이 잔뜩 있는 코드들을 전환하려고하면 꽤 진통이 있을 것 같다.

# 레퍼런스

https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration

https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration#migrating-from-pages-to-app

https://merrily-code.tistory.com/154

https://velog.io/@cyranocoding/Next-js-%EA%B5%AC%EB%8F%99%EB%B0%A9%EC%8B%9D-%EA%B3%BC-getInitialProps