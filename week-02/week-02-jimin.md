### 11/4 CSS content-visibility를 이용해 렌더링 성능 향상 시키기
[아티클 링크](https://kofearticle.substack.com/p/korean-fe-article-css-content-visibility?utm_source=post-email-title&publication_id=695413&post_id=151113223&utm_campaign=email-post-title&isFreemail=true&r=1ehppm&triedRedirect=true&utm_medium=email)     
- 이모지 선택기의 성능문제, 개선 방법   
  - 이모지 선택기 열람 중 4만 개의 DOM 요소 생성 -> 빈번한 레이아웃 및 높은 페인트 비용   
  1. 가상화 -> 구현 복잡, 접근성에 영향을 미침   
  2. **`content-visibility` 속성을 통해 DOM 특정 부분을 숨겨 레이아웃, 페인트 비용 절감 시도**   
    - 접근성 영향 X, 화면 밖 요소 크기를 브라우저가 미리 계산해 성능 개선 O   
    - `<img loading="lazy">` 가 해결하지 못하는 이슈 발견 -> `<img>` 태그 -> CSS를 통한 배경 이미지 설정으로 성능 개선   
    - `IntersectionObserver` 사용으로 성능 개선도 향상   
      - 카테고리가 화면에 나타날 때만 이미지 로딩   
> 성능 개선의 이슈를 마주할 때, lazy loading을 적용해서 개선했던 적이 많았었다!
> IntersectionObserver web API를 사용해 loading='lazy'로 해결하지 못하는 이슈도 해결이 가능하다는 사실을 이제야 알게 된...

### 11/5 지연 시간이 긴 환경 최적화하기
[아티클 링크](https://emewjin.github.io/optimising-for-high-latency-environments/?utm_source=substack&utm_medium=email)       
- **RTT**: 클라이언트와 서버 간에 요청이 갔다가 돌아오는 데 걸리는 시간       
  - ~~대역폭~~(X) 지연 시간(O)이 웹 성능에 주된 영향을 미치는 것 보여주는 지표       
- **지연 시간 줄이기!**       
  1. 전송에 필요한 패킷 수를 줄여 RTT의 영향 완화       
  2. CDN으로 전세계 서버에서 데이터 제공       
  3. 성능 좋은 DNS 사용(ex. cloudflare)       
  4. HTTP/2, HTTP/3으로 업그레이드 -> 멀티플렉싱으로 여러 리소스를 동시에 전송       
    - HTTP/3: UDP 기반으로 TLS를 프로토콜에 포함해 지연 시간 줄임       
  5. TLS 1.3 + 0-RTT 사용       
- **불필요 요청 최소화**       
  - 불필요한 리다이렉트 -> 순수 지연 시간으로, CRP에서 리다이렉트 줄이기       
  - 프리플라이트 요청 피하기 -> CORS로 인해 발생하는 프리플라이트 요청 피하거나 캐싱 -> 대기 시간 최소화       
- **선대처, 캐싱 활용**       
  - Preconnect와 Speculation Rules API 사용       
  - HSTS 설정 -> 사용자가 사이트를 재방문할 때 https 리다이렉트를 피하기 위해 HSTS로 보안 연결 유지       
- **캐싱을 통한 지연 최소화**       
  - 브라우저 및 CDN 캐싱       
  - `Access-Control-Max-Age` 헤더로 프리플라이트 요청의 재검증 주기 늘리기

### 11/7 shadcn/ui 해부
[아티클 링크](https://siosio3103.medium.com/shadcn-ui-%EC%9D%98-%ED%95%B4%EB%B6%80-ebd469c34614)        
- **Headless UI 구조**        
  - `shadcn/ui` - UI 컴포넌트를 `구조와 동작` 계층과 `스타일` 계층으로 분리        
    - 구조 계층: 컴포넌트의 핵심 동작(예: 키보드 내비게이션, 접근성) 캡슐화        
    - 스타일 계층: Tailwind CSS를 사용해 다양한 스타일 변형 관리        
-> 이러한 분리로 인해 디자인 시스템의 편리한 유지보수, 일관성 유지 가능        
- **주요 의존성**        
  - **Radix UI**: Accordion, Popover, Tabs 같은 headless 컴포넌트의 동작을 위한 라이브러리        
  - **React Hook Form**: 폼 상태 관리 및 유효성 검증을 위한 라이브러리        
  - **Tanstack React Table**: Table과 DataTable 같은 고급 테이블 컴포넌트를 headless 형태로 구현        
  - **React Day Picker**: 캘린더 및 날짜 선택 컴포넌트를 위한 headless 라이브러리        
- **스타일 관리**        
  - Tailwind CSS를 기반으로 모든 스타일 관리, `Class Variance Authority(CVA)`를 통해 변형(variants) 스타일 구성        
  - `twMerge`, `clsx` 유틸리티 -> Tailwind 클래스 간의 충돌을 방지하고, 조건부 스타일 적용 용이        
- **컴포넌트 예시**        
  - **Badge 컴포넌트**: class-variance-authority의 cva 함수로 스타일 변형 정의, cn 유틸리티로 Tailwind CSS 클래스 문자열 관리        
  - **Switch 컴포넌트**: Radix UI의 SwitchPrimitives로 키보드, 스크린 리더 지원 등 접근성 충족        
    - 스타일: Tailwind CSS로 커스터마이징 유연성 확보 가능        
