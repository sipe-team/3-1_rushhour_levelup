### 6주차 전체

App Router는 Next.js 13부터 도입된 새로운 라우팅 시스템으로, 기존의 pages 디렉토리 기반 라우팅에서 app 디렉토리 기반으로 라우팅 방식이 변경되었습니다.  
기존의 Pages router와 비교했을 때, 아래의 점들에서 큰 차이점을 보입니다.

- Layout
  - 기존 Pages router의 경우 공통 레이아웃을 여러 페이지에서 재사용하려면 Layout 컴포넌트를 명시적으로 각 페이지에서 포함해야 했지만, App router의 경우 app/layout.js 파일에서 레이아웃을 정의하고, 이를 각 페이지에서 공유할 수 있습니다. (자동으로 레이아웃을 상속받습니다.)
- Server Components
- Streaming
- Server Actions

#### 1. Server Rendering & Prerender
서버에서 렌더링 작업을 수행하면 데이터 페칭, 보안, 캐싱, 성능, 초기 페이지 로드, SEO, 스트리밍 면에서 장점을 가져갈 수 있습니다.  
서버에서 HTML을 미리 렌더링하여 준비하는 기술인 Prerendering 개념으로 볼 수 있습니다. 
- HTML을 빌드 타임에 생성하는 SSG
- 주기적으로 생성하는 ISR
- 요청마다 매번 생성하는 SSR
> SSG와 SSR을 결합한 Partial Prerendering (PPR)이 실헙적 기능으로 도입되어있습니다.

Next.js에서는 세 가지의 서버 렌더링 전략을 가져가고 있습니다.
- Static Rendering (Default) - SSG
  - 빌드 또는 revalidation 시 백그라운드에서 렌더링됩니다. 결과는 캐싱되어 CDN에 푸시됩니다.
- Dynamic Rendering - SSR
  - 요청마다 각 사용자에 대해 렌더링됩니다.
  - 개인화된 페이지 영역에 대해 유용합니다.
  - 렌더링 중에 Dynamic Functions 또는 캐시되지 않은 데이터 요청이 발견되면 전체 경로가 Dynamic으로 전환됩니다.
  - searchParams 사용에 유의해야합니다. ([2] 상품 목록 페이지 튜닝)
- Streaming (+ Suspense)
  - App Router에서 도입된 전략입니다.
  - 서버에서 UI를 점진적으로 렌더링하여 초기 페이지 로딩 성능을 향상시킵니다.
  - loading.js, 리액트 Suspense를 사용하여 구현 가능합니다.

#### Server Components
Server Components는 React 18의 서버 컴포넌트를 기반으로 한 기능으로, 서버에서 렌더링되는 컴포넌트입니다. use client를 명시하지 않는 한 모든 컴포넌트는 Server Components입니다.
- App Router의 컴포넌트는 기본적으로 서버 컴포넌트로 동작합니다. 클라이언트 컴포넌트로 변환하려면 use client 선언이 필요합니다.
- Pages Router의 경우 SSR/SSG 지원으로 성능 최적화를 했다면 App Router의 경우 서버 컴포넌트로 성능 최적화가 가능합니다. (getStaticProps, getServerSideProps를 사용하지 않아도 됩니다.)

#### Hydration 
서버에서 렌더링된 HTML을 리액트 컴포넌트로 변환하는 과정입니다. 리액트는 서버에서 렌더링된 HTML을 하이드레이션하여, 클라이언트 측에서 리액트가 제어할 수 있는 상태로 만듭니다.  
이때, 서버에서 렌더링된 HTML과 클라이언트에서 React가 렌더링한 결과가 일치하지 않는 경우에 Hydration 에러가 발생합니다.
- 상태 불일치
  - 클라이언트와 서버에서 사용하는 상태가 다른 경우에 에러가 발생합니다.
  - 브라우저에서 localStorage나 window 객체에 접근하는 코드가 서버에서 실행되면 서버와 클라이언트에서의 렌더링이 다를 수 있습니다.
  - 서버에서 클라이언트 전용 API를 호출하지 않도록 주의가 필요합니다.
- 비동기 데이터 불일치
  - 서버에서 데이터를 가져와 렌더링할 때와 클라이언트에서 데이터를 가져오는 시점에 차이가 있을 경우에 에러가 발생합니다.
  - 서버 사이드에서의 데이터를 클라이언트에서 동일하게 가져와서 사용하는 방식으로 해결할 수 있습니다.
    - Tanstack Query의 경우 initialData, Hydration API를 활용할 수 있다.
      - HydrationBoundary, dehydrate

 

#### 1. Caching
> 대부분의 Next.js 캐싱 휴리스틱은 API 사용에 의해 결정되며,  
> 최소한의 구성으로도 최상의 성능을 발휘하도록 기본값이 설정되어 있습니다.

SSG와 ISR은 CDN에서의 정적 페이지 캐싱 활용, SSR은 캐싱 서버 활용 및 HTTP 캐싱 활용과 같이 기본 캐싱 전략은 동일하지만, Next.js 13과 React 18 이후 등장한 Sever component, Suspense, Concurrent Rendering 등으로 캐싱 전략이 더욱 세분화되었고, App Router에서 지원하는 캐싱 동작 방식도 다양해졌습니다.

#### 캐싱 동작 방식
- Server Caching
  - Request Memoization (함수의 반환 값 캐싱)
    - next.js 기능은 아니고 react 자체 기능입니다.
    - 동일한 url/옵션을 가진 api 요청을 자동으로 메모이제이션한 후, 여러 리액트 트리 내에서 동일한 요청 시 한번만 호출합니다. (서버 컴포넌트에서 props로 넘길 필요가 없습니다.)
    - 메모이제이션은 요청의 수명동안만 지속됩니다. 서버 요청 간에 공유되지 않고 렌더링 중에만 적용되므로 revalidation이 필요하지 않습니다.
    - 캐시는 메모리에 저장됩니다.
  - Data Cache (데이터 캐싱)
    - 서버 요청과 배포 간에 데이터 페칭을 지속적으로 유지합니다.
    - 원본 데이터 소스로의 요청 수를 줄일 수 있습니다.
    - nextConfig의 revalidate 옵션으로 설정합니다.
      - revalidate 옵션은 Data Cache와 Full Route Cache 둘 다에 영향을 줍니다. (Router Cache에는 영향 x)
      - no-cache : max-age=0과 동일합니다. 캐싱은 하지만 사용할 때마다 서버에게 재검증 요청을 보냅니다.
      - no-store : 캐싱 자체를 하지 않습니다. 브라우저는 어떤 경우에도 해당 리소스를 저장하지 않습니다.
    - revalidation
      - time-based
        - 지정된 시간 내에 호출되는 요청은 캐시된 데이터를 반환합니다.
        - 시간 프레임이 지난 후, 다음 요청은 여전히 캐시된(오래된) 데이터를 반환합니다.
          - Next.js는 백그라운드에서 데이터 재검증을 트리거합니다.
          - 데이터가 성공적으로 가져와지면 Next.js는 최신 데이터로 Data Cache를 업데이트합니다.
          - 백그라운드 재검증이 실패하면 이전 데이터가 변경되지 않고 유지됩니다.
      - on-demand
        - 첫 번째 요청이 호출되면 외부 데이터 소스에서 가져와져 Data Cache에 저장됩니다.
        - revalidation이 트리거되면 적절한 캐시 항목이 캐시에서 제거됩니다.
          - 반면 time-based는 재검증 기간 동안 최신 데이터가 가져와질 때까지 캐시에 오래된 데이터를 유지합니다.
        - 다음 요청이 있을 때 다시 캐시 MISS가 되며 데이터가 외부 데이터 소스에서 가져와져 Data Cache에 저장됩니다.
  - Full Route Cache (HTML & RSC 페이로드 캐싱)
    - 빌드 시 자동으로 경로를 렌더링하고 그 결과(RSC Payload + HTML)를 캐싱합니다.
    - 캐시는 Next.js 서버의 파일 시스템(디스크)에 저장됩니다.
    - x-nextjs-cache
      - HIT: 캐시된 페이지 결과를 가져온 것입니다. 이전에 렌더링된 캐시 페이지를 반환합니다.
      - MISS: 캐시된 페이지가 없어서 새로운 페이지 렌더링이 필요함을 의미합니다. 새로운 페이지를 서버에서 렌더링하고 캐시에 저장한 후 반환합니다.
      - STALE: 캐시된 페이지를 반환하면서 동시에 백그라운드에서 페이지의 유효성을 검사합니다. 캐시 갱신이 필요한 경우 새로운 페이지로 대체합니다.
- Client Caching
  - Router Cache (RSC 페이로드 캐싱)
    - 네비게이션(prefetching) 개선에 용이합니다.
    - 캐시는 브라우저의 임시 메모리에 저장됩니다.

#### ETag
ETag HTTP 응답 헤더는 특정 버전의 리소스를 식별하는 식별자입니다. Next.js는 기본적으로 모든 페이지에 대해 ETag를 생성합니다. (generateEtags 옵션)
- If-Match: ETag 값 포함하여 post 요청 → 해시가 일치하지 않으면 412 에러 (문서가 변경됨)
- If-None-Match: ETag 전송 → 해시 일치하면 304 반환 (캐시 유효함)

304 응답은 HTTP 본문을 포함하지 않기 때문에 매우 빠릅니다.  
유효기간 내에 요청이 들어온다면, 브라우저는 서버 요청 없이 디스크 또는 메모리에서 캐시를 읽어와 사용합니다. (디스크 캐시인지 메모리 캐시인지는 브라우저가 결정합니다.)  
유효기간이 지났다면, 브라우저는 ETag를 포함해 조건부 요청을 보냅니다.

#### Server Components의 렌더링 및 캐싱
Server Components의 렌더링 및 캐싱은 다음과 같은 프로세스로 동작합니다.

1. 서버 렌더링
- 리액트는 서버 컴포넌트를 RSC Payload 형식으로 렌더링합니다.
- Next.js는 RSC Payload를 활용하여 서버에서 HTML을 렌더링합니다. (TTV)

2. 서버 캐싱 (Full Route Cache)
- 경로의 렌더링 결과(RSC Payload + HTML)를 서버에 캐싱합니다.
- Next.js의 서버 디스크에 저장됩니다.

3. Hydration & Reconciliation
- RSC Payload를 사용하여 클라이언트 및 서버 컴포넌트 트리를 조정하고 돔을 업데이트합니다.
- 클라이언트 컴포넌트를 Hydration합니다. (TTI)

4. 클라이언트 캐싱 (Router Cache)
- 이전 경로를 저장하고 미래 경로를 프리패치합니다.
- 이후 프리패치 시 Router Cache를 확인하고, 데이터가 없으면 서버에서 RSC Payload 가져와서 캐시에 채웁니다.
  - 브라우저의 임시 메모리에 저장됩니다.

#### 3. Data Fetching
대부분의 일반적인 페칭은 Server Components를 활용할 수 있습니다.
- fetch API 확장
  - Reqeust Memoization 및 Data Cache가 적용됩니다.
- ORM, DB 클라이언트 호출

잦은 리렌더링이 예상된다면 클라이언트 측에서 페칭하는 게 더 적합할 수 있습니다.
- SWR, Tanstack Query
  - Hydration에 유의해야 합니다.
- Route Handlers를 활용할 수도 있습니다.

#### Server Actions
클라이언트 컴포넌트에서 서버의 비동기 함수를 호출할 수 있는 리액트 기능입니다. 서버 API를 따로 만들지 않고도 서버 코드를 작성할 수 있습니다. use server를 명시하여 정의할 수 있습니다.  
폼 제출 및 데이터 mutation에 용이합니다.

#### 4. Optimizing

#### 이미지 최적화
- Next.js Image는 레이아웃 이동을 자동으로 방지합니다.
  - 이미지가 로드되면서 페이지의 다른 요소를 밀어내는 현상으로, 이미지가 성능에 가장 큰 영향을 미치는 것 중 하나입니다.
- next.config에 URL 패턴 목록을 정의함으로써 이미지 최적화를 안전하게 허용할 수 있습니다.
  - 현재 팬페이지에서는 remotePatterns 말고 domains를 사용하고 있습니다.
  - Next.js에서는 도메인에서 제공되는 모든 콘텐츠를 소유하는 경우에만 domains를 사용할 것을 권장하고 있습니다.
- 반응형 이미지
  - 현재 팬페이지에서는 deviceSizes을 사용하여 너비 중단점을 지정하고 있습니다.
  - 이미지의 크기를 알 수 없는 경우, fill 속성과 함께 미디어 쿼리 분기점에 맞추어 sizes 속성을 사용하고 있습니다.
    - px로 정의할 경우, devicePixelRatio 값에 따라 크기가 다르게 계산되는 문제가 있어 vw로 정의하는 것이 좋아보입니다.

#### Lazy Loading
> Lazy Loading은 Client Components에 적용되는 기능입니다.  
> Server Components의 경우에는 자동으로 code splitting되며, 서버에서 클라이언트로 점진적으로 전송하는 streaming을 사용하면 됩니다. Server Components에 dynamic을 적용하면 Server Components의 자식 Client Components가 lazy loaded됩니다.

Next.js의 dynamic은 React.lazy()와 Suspense의 복합체입니다. dynamic을 사용하면 Client Components는 기본적으로 SSR됩니다. (false 옵션을 적용할 수 있습니다.)
