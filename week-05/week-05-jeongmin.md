### 11/27 수 - 11/28 목

> 회사에서 난생 처음 next를 쓰게 되어서 공부중임다... ㅠ

#### [Next.js의 App router]
Next.js 13부터 도입된 새로운 라우팅 시스템으로, 기존의 pages 디렉토리 기반 라우팅에서 app 디렉토리 기반으로 라우팅 방식이 변경되었다.  
기존의 Pages router과 비교했을 때, 파일 시스템 기반 라우팅과 동적 라우팅 면에서는 유사하지만 아래의 점들에서 큰 차이를 보인다.
- Layout
  - 기존 Pages router의 경우 공통 레이아웃을 여러 페이지에서 재사용하려면 Layout 컴포넌트를 명시적으로 각 페이지에서 포함해야 했지만, App router의 경우 app/layout.js 파일에서 레이아웃을 정의하고, 이를 각 페이지에서 공유할 수 있다. (자동으로 레이아웃을 상속받음)
- Server Component
  - App Router에서는 컴포넌트가 기본적으로 서버 컴포넌트로 동작한다. 클라이언트 컴포넌트로 변환하려면 use client 선언이 필요하다.
  - 서버 컴포넌트를 사용하여 서버에서만 렌더링할 부분과 클라이언트에서만 렌더링할 부분을 명확히 구분할 수 있다. (pages router의 경우 SSR/SSG 지원으로 성능 최적화를 했다면 app router의 경우 서버 컴포넌트로 성능 최적화 가능)
  - 데이터 패칭을 서버 컴포넌트 내에서 직접 처리할 수 있기 때문에, 데이터 로딩 상태를 클라이언트에서 관리하지 않아도 된다. getStaticProps, getServerSideProps를 사용하지 않고도 서버에서 데이터를 받아 렌더링할 수 있다.
- Streaming
  - 기존 서버 렌더링 전략으로는 Static, Dynamic이 있었다. 그 과정에서 TTI가 일어나기 전까지 꽤나 시간이 걸리는데, HTML을 작게 나누어 부분적으로 미리 렌더링할 수 있게 하는 개념이다.
  - loading.js, 리액트 Suspense를 사용하여 구현 가능하다.

#### [Next.js의 Server component]
Next.js 13에서 도입된 기능으로, React 18의 서버 컴포넌트를 기반으로 한 기능이다. 서버 컴포넌트는 클라이언트에서 실행되지 않고 서버에서만 렌더링되는 React 컴포넌트다.   use client를 명시하지 않는 한 모든 컴포넌트는 서버 컴포넌트다.  
서버에서 렌더링 작업을 수행하면 데이터 페칭, 보안, 캐싱, 성능, 초기 페이지 로드, SEO, 스트리밍 면에서 장점을 가져갈 수 있다.  
- 서버 렌더링
  - 리액트는 서버 컴포넌트를 RSC Payload 형식으로 렌더링한다.
  - Next.js는 RSC Payload를 사용하여 서버에서 HTML을 렌더링한다. (TTV)
- 서버 캐싱 (Full Route Cache)
  - 경로의 렌더링 결과(RSC Payload + HTML)를 서버에 캐싱
- Hydration & Reconciliation
  - RSC Payload를 사용하여 클라이언트 및 서버 컴포넌트 트리를 조정하고 돔을 업데이트한다.
  - 클라이언트 컴포넌트를 Hydration한다. (TTI)
- 클라이언트 캐싱 (Router Cache)
  - 이전 경로를 저장하고 미래 경로를 프리패치한다.
  - 이후 프리패치 시 Router Cache 확인 → 없으면 서버에서 RSC Payload 가져와서 캐시에 채움
