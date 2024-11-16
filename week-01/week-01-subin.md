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
