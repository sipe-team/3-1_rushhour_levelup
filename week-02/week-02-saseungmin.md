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
