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
## 심규보

### 11월 4일
1.  Xcode optimization level 종류
 - Onone: debug 모드 (최소한의 최적화와 디버그 정보 기록 o)
 - O: release 모드 (최적화 수행과 디버그 정보 기록 x)
 - Osize: 성능보다 코드 사이즈를 줄이는 것에 최적화
2.  릴리즈 앱에서 assertFailure가 발생했다?
> 그럼 assert가 머지?
- assert: 내부 조건이 false가 되면 runtime error를 발생하여, 주의해야한다는 의미로 사용
- assertionFailure: 별도의 조건은 없고 바로 runtime error 발생
- precondition: assert는 true여야 runtime error가 나지만, precondition은 false여야 runtime error 발생
- preconditionFailure: 별도의 조건은 없고 바로 runtime error 발생
- fatalError: 별도의 조건이 없고, 무조건 runtime error 발생
3. 적용되는 범위
> (-Oneon은 debug모드의 디폴트 값, -O은 Release 모드의 디폴트 값)
- assert, assertFailure: -Onone일때만 활성화
- precondition, preconditoinFailure: 항상 활성화
- fatalError - 항상 활성화
4. 결론
- debug 모드 (Onone) 에서만 runtime error가 나가끔 하려면 assert, assertFailure 사용
- release 모드 (O) 에서도 runtime error가 나게끔 하려면 precondition, preconditionFailure 사용
- assert, precondition 조건 없이 에러가 나는 의미를 전달하고 싶을땐 fatalError 사용
- 
### 11월 6일
대부분 회사에서 "네트워크 통신 == Alamofire == 편함" 사용을 디폴트로 여기는 시점에서 다시금 생각나서 읽어보았어용

URLSession을 사용하여 async/await 방식으로 네트워크 요청을 수행하는 방법.
- Alamofire 같은 서드파티 라이브러리가 널리 쓰이지만, URLSession의 기본 기능을 활용해도 안정적인 네트워크 통신이 가능하다는 점이 인상적임.
- 생각과는 다르게 GET 요청은 물론 URLComponents를 활용해 파라미터를 추가하거나 POST 요청에서 데이터를 전송하는 과정도 생각보다 꽤 간단히 구현 가능함
- 추가로 async/await을 활용해서 비동기적으로 처리도 충분히 가능하고, 에러 핸들링 역시 추상화없이 세분화된 에러 타입으로 관리가능함. 코드 가독성도 나쁘지 않음
- 서드파티 대신 대신 퍼스트파티만트로 외부 종속성 없이충분히 견고한 네트워킹 레이어를 구축할 수 있는거 다시 확인하게 되었어용.


### 11월 8일
카카오에서 tuist를 쓰면서 대한민국이 전세계 tuist 사용자 2위국가가 됐다. 블로그보면 너도나도 써보고 있는데 부정적인 입장에서 적어본 TIL
1.종속성 문제
- 만약 다 적용해놨는데 바꿔야될 일이 생기면? 그땐 진짜 머리아파 질거 같다.
- 이 친구가 자동 생성한 프로젝트 파일을 직접 관리해야 한다면, 수동으로 모듈별 의존성을 다시 설정하고 스킴과 타깃 구성도 전부 수작업으로 만들어야 될듯. 적고보니 1번 2번이 비슷하지만 같은 얘기일래나
- 특히 모듈 구조가 복잡하면 설정 충돌을 해결하는 데 꽤나 골치 아픈 작업이 될 수 있을 것 같다. 혹 떼려다 붙히는 꼴이 될수도
2.작은 팀에겐 복잡함
- 당연한 얘기겠지만 소규모 팀에는 오버테크가 될수도 있겠다
3. 학습 곡선?
- 회사입장에서는 쓸줄 아는 사람을 뽑으면 되지만 이래서 취준생들이 많이 하나보다..
결론) 기존의 xcode를 사용하면서 가려운부분을 확실히 긁어주는것 맞지만 신규 프로젝트가 아니면 굳이? 인가 싶음


## 강준후
### 11/5 spring-ai에 대해서 알아보았습니다.

이미 서비스에서 외부 ai 서비스를 사용하고 있는데, 나름 구조화를 하여 사용하고 있지만 제한된 점이 많이 있는데요. 그 부분에 대해서 어느정도 해결할 수 있는 spring-ai를 확인해보았습니다.

특징 몇 가지만 공유드리겠습니다.

1. 출력을 Pojo로 손쉽게 매핑할 수 있습니다.
2. 사용량, limit 등의 정보를 쉽게 확인할 수 있게 되었습니다.
3. 요청에 필요한 정보들이 구조화되어서 손쉽게 사용도 가능하고, 통일된 방식으로 요청도 가능하다는 이점도 있는 것 같아요!

참고 링크 공유드립니다.

https://spring.io/projects/spring-ai

### 11/6 리액티브 프로그래밍

리액티브 프로그래밍(Reactive Programming)은 **비동기적인 프로세싱 파이프라인**을 만들기 위한 목적으로 개발되었고, 함수형 프로그래밍과 유사한 "선언적 코드"를 사용한 새로운 패러다임입니다. 

Reactive Streams Interface 명세

Reactive Stream의 시퀀스는 Publisher가 데이터를 생산해냅니다.

하지만, 기본적으로 Subscriber가 등록되기(구독하기)전까지 아무런 일도 하지 않습니다.

즉, Publisher는 Subscriber가 등록되는 시점부터 데이터를 push 하게 됩니다.

그림을 자세히 보시면, Subscriber는 Publisher에게 피드백을 줍니다.

간단하게 설명하자면, 이 피드백은 혼잡을 제어할 수 있는 도구로 "모든 이벤트를 다 push하지 말고, N개만 보내줘"와 같은 요구에 해당한다고 합니다. 이로써 Subscriber에 데이터가 지나치게 쌓여서 감당할 수 없는 현상을 제거하기 위해 이 피드백은 굉장히 중요한 역할을 합니다.

이러한 피드백을 관장하는 요소가 바로 Back-Pressure 

Reactor는 어떤 데이터가 대해서 각각의 단계를 적용하기 위한 처리에 대한 내용을 기술하기 위해 서로 묶여있는, **오퍼레이터**라는 개념을 추가했습니다.

오퍼레이터를 적용하면, 중간 퍼블리셔(upstream에 대한 subscriber, downstream에 대한 publisher)를 리턴하게 됩니다.

```
Copy
Flux<String> flux = Flux.just("A");

flux.map(s -> "foo" + s);// 새로운 Publisher 생성, 걔를 구독해야 fooA가 찍힘.
flux.subscribe(System.out::println);// 얘는 1번 라인의 flux를 구독, 따라서 "A"가 나온다.
Flux<String> flux = Flux.just("A");

Flux<String> flux2 = flux.map(s -> "foo" + s);

flux2.subscribe(System.out::println);// fooA

```

두 번째 줄의 코드를 보면, flux.map()을 통해서 새로운 중간 publisher가 생성되게 됩니다.

하지만 셋째 줄에서 첫째 줄의 flux를 subscribe 했기 때문에 "fooA"가 아닌 "A"가 나오게 되죠.

그 아래의 코드대로 사용하면, 기존의 의도대로 코드가 나오게 됩니다. 물론 flux2를 생성하지 않고 이어서 선언할 수도 있습니다.

### 11/7 **반응형 스트림(Reactive Stream) 수행과정**

**반응형 스트림 처리 과정**

https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdT2vwe%2FbtsqBwUlPkC%2FjfmERVJtjeqmWbi125Aag0%2Fimg.png

1.**[subscribe]**

Subscriber를 Publisher에 ‘등록’하고 데이터 스트림을 ‘수신할 준비’가 되었음을 Publisher에게 알립니다.

2.**[onSubscribe]**

Publisher가 Subscriber에게 데이터 스트림을 ‘전송하기 시작하기 전에 호출’됩니다. 이 메서드를 통해 Subscriber는 Subscription 객체를 받아들여 데이터의 양을 제어할 수 있습니다.

3.**[request(n)/cancel]**

request(n) 메서드는 Publisher에게 n개의 데이터를 요청하고, cancel 메서드는 데이터 스트림을 취소합니다.

4.**[onNext(data)]**

Publisher가 ‘생성한 데이터’를 Subscriber에게 전달합니다. 이 메서드는 데이터가 전송될 때마다 호출됩니다.

5.**[onComplete/onError]**

Publisher가 모든 데이터를 전송하고, 더 이상 데이터가 없을 때 호출됩니다.

- onComplete 메서드는 모든 데이터가 성공적으로 전송되었음을 나타냅니다.
- onError 메서드는 데이터 전송 중 오류가 발생했음을 나타냅니다.

### 11/8  **Mono/Flux**

Mono

- **Reactor 라이브러리에서 제공하는 Reactive Streams의 Publisher 중 하나로 오직 ‘0개 또는 하나의 데이터항목 생성’하고 이 결과가 생성되고 나면 스트림이 종료되면 결과 생성을 종료합니다.**
- Mono를 사용하여 비동기적으로 결과를 반환하면 해당 결과를 구독하는 클라이언트는 결과가 생성될 때까지 블로킹하지 않고 다른 작업을 수행할 수 있습니다.

| **메소드** | **설명** |
| --- | --- |
| **just()** | 주어진 데이터를 포함하는 Mono를 생성합니다. |
| **empty()** | 데이터가 없는 Mono를 생성합니다. |
| **error()** | 에러 상황을 나타내는 Mono를 생성합니다. |
| **fromCallable()** | Callable 객체를 이용해 Mono를 생성합니다. |
| **fromFuture()** | Future 객체를 이용해 Mono를 생성합니다. |
| **fromRunnable()** | Runnable 객체를 이용해 Mono를 생성합니다. |

```java
Mono.just("Hello, world!")
    .map(String::toUpperCase)
    .flatMap(s -> Mono.just("Mono: " + s))
    .subscribe(System.out::println);

```

Flux

- **Reactor 라이브러리에서 제공하는 Reactive Streams의 Publisher 중 하나로 Mono와 달리 ‘여러 개의 데이터 항목’를 생성하고 스트림이 종료되면 결과 생성을 종료합니다.**
- 비동기 작업을 수행하면 작업이 완료될 때까지 블로킹하지 않고 다른 작업을 수행할 수 있습니다.
- Spring WebFlux에서 Flux를 사용하여 HTTP 요청을 처리하는 경우, 요청을 수신한 즉시 해당 요청을 처리하고 결과를 생성하는 대신 결과 생성이 완료될 때까지 다른 요청을 처리할 수 있습니다.

| **메소드** | **설명** |
| --- | --- |
| **just()** | 주어진 데이터를 포함하는 Flux를 생성합니다. |
| **fromIterable()** | Iterable 객체를 이용해 Flux를 생성합니다. |
| **fromArray()** | 배열을 이용해 Flux를 생성합니다. |
| **fromStream()** | Stream을 이용해 Flux를 생성합니다. |
| **empty()** | 데이터가 없는 Flux를 생성합니다. |
| **error()** | 에러 상황을 나타내는 Flux를 생성합니다. |
| **range()** | 지정된 범위의 정수를 포함하는 Flux를 생성합니다. |
| **interval()** | 일정 시간 간격으로 값을 생성하는 Flux를 생성합니다. |
| **merge()** | 여러 개의 Flux를 병합하여 하나의 Flux로 만듭니다. |
| **concat()** | 여러 개의 Flux를 순차적으로 이어붙여 하나의 Flux로 만듭니다. |
| **zip()** | 여러 개의 Flux를 조합하여 튜플 형태로 만듭니다. |
| **collectList()** | Flux에 포함된 모든 데이터를 리스트로 수집합니다. |
| **collectMap()** | Flux에 포함된 모든 데이터를 맵으로 수집합니다. |

```java
Flux.just("apple", "banana", "cherry")
    .map(String::toUpperCase)
    .flatMap(s -> Mono.just("Flux: " + s))
    .subscribe(System.out::println);

```
