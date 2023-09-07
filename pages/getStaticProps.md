# getStaticProps

getServersideProps와 비슷한 일을 수행하지만 성격은 매우 다른 함수입니다.

getStaticProps의 특징은 다음과 같습니다.

- 사용자 요청에 앞서 빌드 타임에 모든 데이터가 제공됩니다.

- 데이터는 헤드리스 CMS에서 제공됩니다.

- 페이지는 SEO를 위해 사전렌더링 되며 속도가 매우 빨라야합니다.

- 성능을 위해 HTML, JSON 파일이 캐시될 수 있습니다.

- 데이터는 공개적으로 캐시될 수 있으며 특정한 경우 우회할 수 있습니다.

# 실행 시점

getStaticProps는 "항상" 서버에서 실행되며 클라이언트에서는 실행되지 않습니다.

# 알아두어야 할 것

getStaticPropsfㅡㄹ 빌드 할 때 사전렌더링 되면 HTML 파일 외에도

next.js는 실행결과를 보유하는 JSON 파일을 생성합니다.

# 그 외에도 사용 방법

getStaticProps는 서버에서 실행되기 때문에 node api를 사용할수도 있다.

path, fs와 같은 node.js에서만 제공되는 api의 사용이 필요할 때에도 사용가능하다.


# 마치며

getStaticProps와 getServersideProps를 통해

서버렌더링과 SSG를 구분하는 방식은 앱라우터에 비해 좀 더 명확하다고 느껴집니다.

