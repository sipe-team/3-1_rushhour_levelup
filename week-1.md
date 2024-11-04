# 1주차: 10/28 ~ 11/02

&nbsp;

## 이정민

### 10/30 수

> Bytes 라는 FE 기술레터를 구독하고 있는데, 이모지가 포함된 문자열을 뒤집는 내용을 옛날에 읽고 언젠가 블로그 글로 써봐야지 하고 미룬 적이 있었습니다. 오늘 그 주제를 다시 살펴보았습니다!

> 블로그 글도 하나 완성했습니다! 👉[개발자 단민 | 제목은 문자열 뒤집기로 하겠습니다. 근데 이제 이모지를 곁들인..](https://www.jeong-min.com/76-emoji-reverse/)👈

- 하나의 기본 이모지는 두 개의 UTF-16 코드 유닛으로 이루어져있고, 하나의 유니코드 코드 포인트를 가진다.
- 자바스크립트는 내부적으로 UTF-16 인코딩을 사용하기 때문에 기본 이미지의 길이는 2이다.
- ZWJ(Zero Width Joiner)는 유니코드 문자 U+200D로, 이모지들을 결합하여 새로운 의미를 만드는 특수한 보이지 않는 결합 문자다.
- 이러한 결합 이모지를 ZWJ 시퀀스라고 한다.
- split('')은 문자열을 UTF-16 코드 유닛 단위로 분리한다.
- 스프레드 연산자와 Array.from()은 문자열을 유니코드 코드 포인트 단위로 분리한다.

### 10/31 목

> 서핏을 구독하고 있는데, [Question Growth Club에서 발행된 아티클](https://maily.so/qtog/posts/x1zg071prqg)을 소개하길래 한 번 읽어보았습니다! 기술적이라기 보다는 성장에 관한 내용이라 재밌고 편하게 읽을 수 있었어요.

- 성장이란 꾸준히 자신의 안전지대를 넓혀가는 것
- 성장 마인드셋: 자신의 재능이 개발될 수 있다고 생각함
- 고정 마인드셋: 재능은 타고나는 것이라고 생각함
- 재능 기반, 즉 역량만을 보는 회사 환경은 성장 마인드셋을 저해함
- 골디락스 각성 영역 - 적당한 스트레스가 있을 때 우리의 성과가 가장 좋다! (두려움은 오히려 학습 및 성장 영역으로 가는 과정에서 필요한 단계)
- 안전지대 -> 불안지대 -> 학습지대 -> 성장지대
- 안전지대에 위치하고 있다는 것을 아는 것 자체가 메타인지
- 중요한 건 꾸준함. 하루아침에 큰 변화를 기대하기보다는, 매일 조금씩 성장해나가는 것이 중요하다!

사실 최근에 선택한 이직이라는 길이 이게 맞는 길일까 하는 의문이 계속해서 들었어요. 안정적이고 편한 곳을 떠나서 힘들고 어쩌면 더 척박한 환경일지도 모르는 곳으로 가는 게 맞을까? 이 글을 읽으면서 제 선택에 대한 확신을 좀 더 가질 수 있었어요. 얼른 불안지대를 안전지대로 만들고 싶습니다 :)

### 11/1 금

> ECMAScript에 제안된 것들 보면 좋은 게 많더라고요~ 기대가 됩니다. (stage 올라가는 데 시간이 넘 많이 걸림🥲)

- 최근 자바스크립트 Date를 대체할 Temporal API가 ECMAScript stage 3단계로 올라감
- 자바스크립트의 Date 객체는 문제가 있음
  - ECMAScript 시간 값은 number임
  - JS에서 날짜는 UTC 기반이지만, 윤초를 무시하는 POSIX 시간을 따르기 때문에 정확한 UTC 시간과는 차이가 있을 수 있음
  - 숫자로 표기하면 날짜의 기본적인 의미가 사라짐 (숫자를 날짜로 변환할 때, 그 기준에 따라 값이 달라질 수 있음)
  - 오프셋을 포함하여 ISO 형식으로 사용하더라도, UNIX 기준 시간으로부터 경과한 밀리초와 오프셋만 알 수 있기에 현실적인 시간대를 알기에는 여전히 충분하지 않음
- 이를 해결하는 것이 ZonedDateTime
  - Temporal API는 날짜와 시간을 해당 시간대와 함께 표현하도록 특별히 설계된 Temporal.ZonedDateTime 객체를 도입함

&nbsp;

## 강준후

### 10/30 수

React query 낙관적 업데이트 방법
React Query에서 뮤테이션이 완료되기 전에 UI를 낙관적으로 업데이트하는 방법을 배웠습니다. onMutate 옵션을 사용하여 캐시를 직접 업데이트해서, useMutation 결과에서 UI를 업데이트할 수 있습니다.
[React Query 낙관적 업데이트](https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates#via-the-ui)

### 10/31 목

aws api gateway timeout 시간이 기존 30초에서 그 이상으로 늘릴 수 있는 업데이트가 6월에 있었다.
llm을 사용한 서비스들이 많아지면서 생긴 업데이트라고 한다.
[링크](https://aws.amazon.com/ko/about-aws/whats-new/2024/06/amazon-api-gateway-integration-timeout-limit-29-seconds/)

### 11/1 금

어제 이어서 aws api gateway 관련 아티클 확인
api gateway와 lambda를 사용해 llm api 제공했던 케이스를 읽어봤습니다.
핵심은 커넥션 id를 통해 티켓형식으로 llm api에서 필요한 QPM(query per minute) 제어 및 관리 기능을 제공하는 형태였습니다.
[링크](https://aws.amazon.com/ko/blogs/tech/how-selectstar-builds-ai-redteaming-websocket-api-with-amazon-apigateway/)

## 이동현

### 10/31 목

> 전회사에서 어드민 개발 했을때 Vue 기반 프로젝트에선 Vuetify, React 기반 프로젝트에선 AntD를 사용했었는데 shadcn은 어떤게 다르고 매력적이라 인기 있는지 궁금해 가볍게 알아봤습니다! 샤드시엔이라고 부른다죠?

shadcn/ui 란?

- Radix UI와 Tailwind CSS를 기반으로 구축된 React 컴포넌트 컬렉션, 접근성과 커스터마이징에 중점을 두고 있음.
- 기존의 컴포넌트 라이브러리와 달리, 패키지로 설치하는 것이 아니라 필요한 컴포넌트를 직접 프로젝트에 복사하여 사용하는 것도 가능.
- 컴포넌트의 소유권을 사용자에게 넘김으로써 직접 코드 수준에서 관리가 가능하여 커스터마이징과 관리의 유연성을 높임.

레퍼런스
https://ui.shadcn.com/
https://pyjun01.github.io/v/shadcn-ui/

## 송재룡

### 11/1 금

> 도커 컨테이너가 "더 빨리 뜨면" 좋겠다. 어떻게 하면 빨리 띄울 수 있을까?

- 단순 도커 이미지 크기는 생각보다 큰 영향을 미치지 않는다.
- 실제 영향을 미치는건 Docker Layer의 수다. 레이어를 로드하는 오버헤드를 줄여라.
- 도커 이미지 레이어는 Dockerfile의 한 줄마다 새로운 이미지 레이어를 쌓는게 아니다. 파일 시스템에 변화가 생길 때, 하나의 레이어를 쌓는다.

#### Reference

- [Reddit - 도커 이미지 크기가 컨테이너 시작에 끼치는 영향](https://www.reddit.com/r/googlecloud/comments/1bponng/docker_image_size_impact_on_container_startup)
- [최적화 아티클](https://www.fullstack.com/labs/resources/blog/small-is-beautiful-how-container-size-impacts-deployment-and-resource-usage)

## 김수빈

### 10/30 수

JS 생태계에서 툴링들을 Rust/Zig/Go 같은 언어들로 재구현해서 성능을 높이는 시도가 늘어나고 있는데, 여기에 대한 글입니다.

- 더 나아진 성능
  - 사실 수 년에 걸쳐 정해진 고정된 인터페이스를 기반으로 동일하게 구현한다는 점, 성능을 고민하면서 작성한다는 점, 더 나아가 재작성을 한다는 점 자체에서 성능이 향상되기도 함
  - Node.js에서는 구조적으로 JIT의 이점을 누리기 어려운데, NODE_COMPILE_CACHE 설정이나 AOT 도구로도 개선해볼 시도
- 다른 언어로 작성되는 툴링의 문제점
  - JavaScript로 작성되면 node_modules에서 쉽게 슉 수정해서 테스트 하거나, 문제가 생겼을 때 원인을 식별하고 오픈소스 기여하는 것도 훨씬 쉬워짐
  - 바이너리에서 segfault 나고 펑 터져버리면 JavaScript 개발자는 막막한 마음으로 기다릴 수밖에 없다....

https://nolanlawson.com/2024/10/20/why-im-skeptical-of-rewriting-javascript-tools-in-faster-languages/

### 10/31 목

Python의 logging 모듈의 신비로움 (원래 알았었지만 오랜만에 로거 개발을 위해 복습..)

- logger에서 로그를 찍을 떄는 먼저 `filter` 를 통해 로그를 찍을지 말지 결정 (여러 개의 `filter` 사용 가능)
- `handler` 를 통해 로그를 찍는 클래스로 넘어감 (여러 개의 `handler` 사용 가능)
- `handler` 내에서는 각 handler에 등록된 `formatter`를 통해 log record를 포매팅함 (포매터는 하나만)

Python의 uv의 신비로움

- uv는 Python을 위한 새로운 패키지 매니저 겸 설치 도구 (pip replacement 이기도 함)
- pyproject.toml의 `tool.uv.sources` 를 통해서 각 dependency를 재정의해 git에서 땡겨오도록 설정하거나 할 수 있음
- `tool.uv` 는 uv package manager만 이해하는 것이고, build backend는 해석할 수 없으니, 현재 패키지가 의존하는 하위 패키지의 `tool.uv` 는 무시되지 않을까 가정
  - 그러나 uv로 설치할 때에는 하위의 하위 패키지의 `tool.uv.sources` 까지 잘 존중하여 설치해오고 있고, 반면에 다른 패키지 매니저를 사용하면 바로 박살남
- 흠터레스팅
- 하지만 dependencies에는 어느 source 로부터든 설치되어야 하는 의존성들만 명시하고, 실제로 이걸 어디로부터 설치하는지는 패키지 매니저가 해주는 것이 맞는 것 같기도 해서 그런가? 싶기도 함
  - 뭔가 뭔가임....

### 11/1 금

Go가 좋지 않은 이유

- https://medium.com/@roma.gordeev/why-go-is-the-worst-language-you-could-ever-learn-4a7be0d37509
- 오류 처리가 매우 번거롭고, 고루틴을 이용한 구현이 여러 동시성 문제와 함께 남용되기 쉽다

Vercel의 마이크로프론트엔드 도입

- https://vercel.com/blog/how-vercel-adopted-microfrontends
- 수직 분할, Next.js의 Multi Zones 사용
- 기술적인 내용이 없고 그냥 잘 했다로 소개만 하고 있어서 아쉬운 글..

타입스크립트를 타입스크립트스럽게 사용하기

- https://blog.0chan.dev/2024-10-21-Make-Typescript-Typescriptly/
- 오랜만에 다시 본 TypeScript의 타입 추론 예시들
