# 1주차: 11/04 ~ 11/10

&nbsp;

## 김수빈

### 11/04 월

- :cloud: AWS와 서버리스 이야기
  - AWS에는 다양한 서버리스 서비스들이 있는데, 대표적으로는 Lambda와 Fargate
  - 하지만 App Runner, Aurora Serverless는 서버리스일까?
  - 엄밀하게는 사용하지 않을 때 비용이 청구되지 않아야 서버리스
    - AWS re:Invent에서 AWS CTO가 발표함
  - App Runner는 서버리스로 소개되기도 하지만, idle time에도 유휴 비용이 있으므로 서버리스는 아닐 수 있음
  - 그래도 24시간 사용하지 않는 기준 EC2보다는 비용이 적고, 배포와 운영이 매우매우 간편하다
- :ocean: Istio xDS가 DB Connection Pool을 밀어버릴 수 있다는 이야기
  - Istio에서 Telemetry API 이용해서 Label 설정을 하나 새로 내려줬는데, HTTP나 TCP는 괜찮았는데 DB Connection Pool이 싹 날아가서 서비스 중단된 이야기
  - LDS 전파 과정, MySQL은 서버에서 클라이언트로 TCP handshake 맺기 때문에 Connection manager filter가 동작하지 않는 known-issue 있었음 ;;
  - 결국 3306 포트 등을 예외처리로 해결하였다는 이야기 ㅠ

### 11/05 화

https://newsletter.pragmaticengineer.com/p/bluesky

- 분산형 소셜 네트워크 Bluesky를 만든 이야기
  - 다른 SNS와는 달리 분산형 소셜 네트워크였고, 누구나 자신의 서버를 운영할 수 있었음 (데이터도 서버도 직접)
  - 첫 해에는 엔지니어 3명, 출시 때에는 엔지니어 12명
- 어떻게 가능했을까?
  - 2~3명의 엔지니어가 9개월 동안 프로토타입 개발, 기초 개발
  - v1: AWS 위에서 PostgreSQL, Pulumi 이용
  - v2: federation 지원을 통해 분산형 지원
  - PostgreSQL에서 ScyllaDB와 SQLite로 마이그레이션 (확장)
  - AWS에서 IDC로 마이그레이션 (비용)
- 타임 라인
  - 엘론 머스크의 트위터 인수 발표 후 Bluesky가 갑자기 출시하게 됨
  - 사용자가 늘어나면서 사이버 불링이나 괴롭힘이 급증했고, 차단 기능 하루만에 구현
  - 10만 명, 100만 명이 넘어가면서 공개 출시를 준비
- 분산 네트워크
  - 피드에 각각 분산된 Bluesky 호스팅 서버들을 모두 등록하고 보여주기 위한 고민 (=federation)
  - 내부에 PDS 서버를 먼저 20개 정도 띄워서 운영하고, 현재는 300개의 외부 PDS 서버 호스팅 되어 연결되어 있음
- AWS에서 IDC로 마이그레이션
  - Jake라는 사람이 입사하고 6개월 후 IDC로 마이그레이션 완료
  - 지금도 IDC가 모자르거나 해서 필요하면 AWS를 이용해서 확장이 가능한 구조임

https://newsletter.pragmaticengineer.com/p/bluesky-engineering-culture

시리즈로 Bluesky의 개발 문화에 대한 글도 있습니다.

- TypeScript를 사용 (모바일 앱은 RN, RN for Web까지 썼다고 하는데 대박 사건임)
- Relay 등 성능이 필요하거나 Scylla와의 인터페이스를 위해 Go를 부분적으로 도입.

경험과 역량이 충분하면 소규모 팀으로도 가능하구나를 느낀 글들이었습니다. :+1:

### 11/06 수

예민한 Python 이벤트 루프와 서버 워커에 대한 고민

- Python FastAPI에서 BackgroundTask를 통해 event loop 내에서 비동기로 태스크를 실행할 수 있음
- 이 때 이 비동기 태스크 내에서 동기 함수를 하나라도 사용하거나 해서 event loop을 30초 이상 차단하면 gunicorn/uvicorn worker가 timeout으로 내려감
  - worker에서는 비동기로 0.1초에 한 번 tmp 경로의 pid 경로에 있는 파일의 마지막 수정 시간을 갱신하고, 이를 체크하면서 worker가 살아있는지 확인함
  - 이게 동기 함수로 인해 차단되어서 문제가 발생
- 해결하는 방법들로는
  - threadpool에서 돌려서 별도의 event loop에서 실행하거나
  - 동기 함수들을 모두 찾아내서 비동기로 고치거나

여러분들은 Python에서 비동기를 쓰지 마세요.

### 11/07 목

Netflix 웹 성능 사례 스터디

- 기존에 React로 작성한 웹 페이지를 Vanilla JavaScript로 전환하여 번들 크기 감소 (-200kb)
  - 데스크톱 랜딩 페이지 최적화
- 다음 페이지는 XHR울 이용해서 css/js만 prefetch
  - html은 no-cache
  - `<link rel=prefetch>`는 브라우저에 따라 동작하지 않을 수 있음
- 참고 사항이 꽤 재미있음..
  - Preact도 고려했지만 상호작용이 적은 페이지의 경우에는 Vanilla JavaScript가 더 간단
  - Service Worker를 이용한 정적 리소스 캐싱 다시 고민 중
  - 랜딩 페이지에 A/B 테스트도 고려해야 함

https://medium.com/dev-channel/a-netflix-web-performance-case-study-c0bcde26a9d9

### 11/08 금

Slack에는 여러 개의 Workspace를 하나로 묶는 Enterprise Grid라는 플랜이 있는데, 이걸 구현한 내용을 소개하는 글입니다.

- Slack은 기존에는 Workspace 단위로 아키텍처, 데이터 샤드, API가 설계되어 있었지만, 큰 기업들에서 여러 Workspace를 함께 사용하는 케이스가 생김
- Workspace의 부모 개념을 하는 org를 만들고 조직 수준의 데이터는 여기의 샤드에 넣음. 여러 Workspace에 거친 채널을 조회할 때 Workspace 용 샤드에 정보가 없으면 org용 샤드로 재요청하도록 구현함.
- 이후에 여러 개선을 했지만 Workspace 중심 아키텍처에 한계를 느끼고 개선하기로 함. 어떻게?
  - 일단 Slack 클라이언트가 시작될 때 org 수준이 아닌 Workspace 수준에서 시작되도록 함
  - Workspace API를 호출하면 속한 org와 채널 목록을 내려주도록 구현하고, 아직 org에서 Workspace로 데이터가 다 채워지지 않은 상태나 장애 상황 등을 고려해서 최대 50개까지의 Workspace를 순회하며 성공할 때까지 API 호출
  - 기존에 org에 저장되던 데이터를 Workspace에 넣도록 변경된 점이 가장 큰 변경
- 대부분의 제품 엔지니어링 팀의 노력이 필요했고, 모든 API와 권한 목록을 정리한 스프레드시트로 작업 분배
  - API 내부 테스트와 통합 테스트 (권한 포함) 진행

샤드 설계의 중요성... Enterprise Grid는 돈이 되니 이렇게 해주었지만 커뮤니티를 위해 Discord처럼 1 user multi workspace(server) 구조로 바뀌는 것은 불가능할 것 같네요 :pensive:

https://slack.engineering/unified-grid-how-we-re-architected-slack-for-our-largest-customers/

## 사승민

### 11/04 월
- [CSS content-visibility를 이용해 렌더링 성능 향상 시키기](https://velog.io/@superlipbalm/improving-rendering-performance-with-css-content-visibility?utm_source=substack&utm_medium=email)
- https://web.dev/articles/content-visibility?hl=ko
  - emoji-picker-element에 [성능 이슈](https://github.com/nolanlawson/emoji-picker-element/issues/444)가 있었음.
    - 크롬에서 15% 사파리에서 5% 성능개선
  - 화면에 표시되지 않을 때 `content-visibility` 속성을 활용하면 렌더링을 건너뛰므로 초기 사용자 로드 속도가 훨씬 빨라질 수 있음
  - 역시나 가상 윈도우나 `IntersectionObserver` 를 사용해서 성능을 개선하는게 더 좋음.
    - `IntersectionObserver` 사용시 Chrome에서 약 40%, Firefox에서 약 35% 개선
  - 하지만, 복잡한 구조가 이미 건들이기 힘든 구조에서 기존 로직을 수정하고도 `content-visibility` 는 간단하게 성능개선을 할 수 있다.
    - 접근성 트리에도 큰 영향이 없음, 페이지내에 검색에 대해서 걱정할 필요 없음
- [React Conf 2024 - The next decade of React Native](https://www.youtube.com/live/0ckOUBiuxVY?si=9EoVRiNMbrnUafBp&t=26569)
  - React Native와 React Web은 둘 다 React를 사용하여 코드가 비슷해 보이지만, 실제로 네이티브와 웹 간에 코드를 공유하는 것 어려움... (대인정)
    - 이거를 해결하기 위해 코드 베이스 하나로 모바일 앱과 웹을 다 커버하기 위해서 React Native for Web이라는 방식이 나오기도 했음.
    - 하지만, 이는 웹 코드 작성 경험을 React Native 쪽에 맞춰버리는 단점이 있음.. 원하는게 이게 아님. (대인정)
    - **React Web을 작성하듯이 React Native를 작성하고 싶다! (해줘)**
    - React Native 팀은 Web의 표준 API들을 React Native 쪽에서 거의 동일하게 동작하도록 구현하여 두 플랫폼에서의 개발 경험의 차이를 줄이고, 더 많은 코드 공유를 가능하게 하는 것이 목표라고 함.
      - 예로 `ResizeObserver`, `IntersectionObserver` 같은 API들을 React Native 쪽에 구현할 예정
  - React Native가 CSS 스타일시트와 비슷하게 사용하는거를 도입했지만, 완벽하게 동일한게 아님.
    - 예를 들어, flex의 gap은 react native에서 사용하지 못함.
    - 그리고 px, rem등 단위는 사용하지 못하고 숫자만 사용가능.
    - CSS의 동작을 최대한 반영하여, `px`, `vw`, `%` 같은 단위를 쓸 수 있게 하고, `calc()`와 같이 복잡한 기능까지도 사용할 수 있도록 하는 것이 목표
  - bridge의 비동기적인 특성 때문에 JS 코드에서 일어나는 React Native의 렌더부터 네이티브 코드에서 일어나는 UI 컴포넌트의 레이아웃이 동기적으로 진행될 수가 없어서, `useLayoutEffect` 같은 API가 웹에서와 다르게 동작함.
    - React Native의 [New Architecture](https://reactnative.dev/docs/the-new-architecture/landing-page)는 이런 구조적인 문제를 해결하기 위해 개발됌.
    - 비동기 bridge 대신 JSI(javascript interface)를 도입해서 JS 코드와 네이티브 코드 간에 동기적인 통신을 가능하게함.
    - new architecture에서는 React 18에서 도입된 concurrent features를 지원한다고함.
    - 웹쪽의 모든 API를 React Native에서 사용할 수 있도록 polyfill 하는 [React Strict DOM](https://github.com/facebook/react-strict-dom)이라는 계층 구조에 대해 실험 중.
  - 그냥 10년이 아니라 1년안에 해줘라

### 11/06 수
- https://evertheylen.eu/p/node-vs-bun/?utm_source=substack
- https://medium.com/deno-the-complete-reference/node-js-vs-deno-vs-bun-server-side-rendering-performance-comparison-f80a5abc766f
- Bun과 Node.js의 백엔드 성능을 비교한 글
- 주요 테스트 결과에서 실제 성능차이는 거의 없음. (node.js가 약간 우세)
- 다양한 병렬 연결 수(100, 200, 800, 2000)에서도 비슷한 패턴 유지
- 멀티프로세싱 차이점
  - Node.js: cluster 패키지 사용 (더 사용하기 쉬움)
  - Bun: reusePort 옵션으로 여러 프로세스 실행 필요
  - Node.js가 워커 프로세스 과부하 방지 기능 제공
- 프레임워크 영향
  - 프레임워크 없이 사용하는 것이 가장 빠름
  - Node.js 프레임워크 속도: koa > express > hapi
  - Bun의 네이티브 serve API가 더 현대적이고 기능적
- 시작 시간
  - Bun이 Node.js보다 약 7ms 빠름
  - 서버 운영 관점에서는 큰 차이 없음
- 결론은 실제 서버 환경에서 Bun은 Node.js와 비교해 큰 성능 차이가 없음.
- bun이 주장하는 큰 성능 향상은 단순한 'hello world' 수준의 벤치마크에서만 유효
- 프레임워크 선택이 실제 성능에 더 큰 영향을 미칠 수 있음

### 11/07 목
- https://tidyfirst.substack.com/p/canon-tdd
- 켄트백도 지쳤나보오...
- Canon TDD의 5단계 프로세스:
  1. 테스트 목록 작성 (Test List)
  2. 하나의 테스트 작성 (Write a Test)
  3. 테스트 통과시키기 (Make it Pass)
  4. 리팩토링 (Optional Refactor)
  5. 반복 (Repeat)
- 오해와 실수
  - 모든 테스트를 한 번에 작성하는 것
  - assertion 없이 테스트 작성
  - 테스트 통과를 위해 assertion 값을 복사/붙여넣기
  - 구현과 인터페이스 설계를 동시에 하는 것
  - 너무 이른 추상화
- 실제 TDD를 제대로 이해하지 못한 상태에서 비판하는 경향이 있다
  - 흔한 잘못된 TDD 비판의 예시:
    - "모든 코드를 작성하기 전에 모든 테스트를 먼저 작성해야 해서 싫어요"
    - 이는 TDD의 잘못된 이해. TDD는 한 번에 하나의 테스트만 작성하고 통과시키는 방식.
  - 실제 Canon TDD 프로세스(위의 5단계)를 정확히 이해하고 실천해본 후 그 과정에서 발견된 실제적인 문제점이나 한계를 지적해야 함
  - 비판 시 주의할 점:
    - 자신이 실천한 방식이 실제 TDD가 맞는지 확인
    - "TDD는 이렇게 하는 거라고 들었는데..."와 같은 불완전한 이해에 기반한 비판 지양
    - TDD의 각 단계가 가진 의도와 목적을 이해하고 그에 대한 비판을 해야 함
  - 건설적인 TDD 비판 예시
    - "한 번에 하나의 테스트만 작성하는 방식이 특정 상황에서는 비효율적일 수 있다"
    - "리팩토링 단계에서 발생할 수 있는 구체적인 문제점"
    - "특정 유형의 프로젝트에서 TDD 적용의 한계"
- **무언가를 비판하려면, 실제 그것을 제대로 이해하고 비판하라**

### 11/08 금
- https://www.trevorlasn.com/blog/javascript-nullish-coalescing-assignment-operator
- 자바스크립트의 `??=` 연산자
- JavaScript에서 `??=`를 사용하여 `null` 및 `undefined` 값을 우아하게 처리하는 방법에 대한 가이드
- es2021에 나왔었는데 처음 알게된... 이런게 있었나?
- can i use에 보면 이미 많은 곳에서 지원.

```js
// Using if statement - verbose and repetitive
if (user.name === null || user.name === undefined) {
  user.name = 'Anonymous';
}

// Using || operator - catches too much
user.name = user.name || 'Anonymous';  // Replaces '', 0, false too

// Using ternary - gets messy with longer expressions
user.name = user.name === null || user.name === undefined
  ? 'Anonymous'
  : user.name;

// Using ??= - clean and precise
user.name ??= 'Anonymous';
```
