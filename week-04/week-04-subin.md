### 11/18 월

JavaScript 엔진과 런타임의 차이

- ECMAScript 언어 사양 ECMA-262에서는 JavaScript의 핵심 기능을 정의
- JavaScript 엔진은 ECMAScript 엔진이라고 봐야할 수 있는데, 실제로 추가 기능 없이 ECMA-262를 거의 그대로 구현함
  - ex. V8, SpiderMonkey, JavaScriptCore
- JavaScript Runtime은 ECMAScript 호스트로, 자유롭게 추가 기능을 구현함. File API, 이벤트 루프도 여기에서 구현됨.
  - ex. Chromium, Node.js, Deno, Bun

https://humanwhocodes.com/blog/2024/03/javascript-engines-runtimes/

### 11/19 화

`curl`을 이용해서 Host를 속이는 여러 방법

- `curl --header "Host: [example.com](http://example.com/)" http://127.0.0.1/`
  - 무난하게 잘 동작하지만, follow redirect 하는 경우에는 Host 헤더가 location으로 이동한 후에도 계속 들어가면서 실패함
- `curl --resolve example.com:443:127.0.0.1 https://example.com/`
  - curl dns cache를 내부적으로 변경하여 접근
  - 필요하면 여러 host에 대한 dns cache를 변경하여 사용하는 것도 가능
- `curl --connect-to example.com:443:host-47.example.com:443 https://example.com/`
  - ip 대신 host를 쓰고 싶으면 이렇게
- 동시에 다 같이 사용하는 것도 가능하고, libcurl에서도 가능함

https://daniel.haxx.se/blog/2018/04/05/curl-another-host/

### 11/20 수

Meta의 MobileConfig

- 개발자가 Config 파라미터를 쉽게 읽을 수 있도록 API로 제공해주는 서비스
  - 정적 값, 또는 Meta 내부 실험/Config와 연동되어 있을 수 있음
- 앱이 시작할 때 한 번 fetch 해오고, 이후로 주기적으로 poll 해서 클라이언트가 항상 최신 값을 볼 수 있도록 함
  - 클라이언트의 컨텍스트, 각 파라미터에 연동된 백엔드를 이용하여 값을 생성해서 내려줌
  - 사용자가 앱과 interaction 하는 동안에는 일관성과 예외 방지를 위해 세션 락
- 원래는 Meta의 각 플랫폼에서 SDK를 제공했지만, 러닝 커브와 엔지니어링 리소스 필요했음
  - 그리고 다양한 플랫폼들을 바꿔 사용할 때 SDK를 매번 연동해야 하는 단점
  - MobileConfig SDK는 이를 개선하기 위한 것으로, MobileConfig API를 사용하는 하나의 인터페이스로 사용 가능
  - Feature Flag, A/B 테스트 모두 가능
- iOS, Android, Oculus, Instagram Threads 등등 순차 지원
- 한 앱에서 수천 명의 개발자가 주간 300번 이상 변경하기 때문에, 충돌 위험도 커지고 안정성이 떨어질 수 있음
  - Feature Flag를 이용해서 일단 꺼둔 채로 배포하고 나중에 켜기 가능
  - 팀 내에서만 켜서 테스트 후 안정화되면 나중에 켜는 것도 가능
- Gatekeeper, Configurator(Meta의 구성 시스템)와 연동됨
  - 사용자과 관련한 정보들을 컨텍스트로 사용해서 Evaluation 후 A/B 테스트에 사용되는 도구 같음 (앱 버전, 앱 종류 등)
- 카나리 배포에서 서버측과는 달리, 앱은 다양한 플랫폼과 하드웨어를 고려해야 함
  - pull-versus-push 배포 모델, 그리고 사용자에게 잘 노출되지 않는 일부 메뉴들로 인해 Canary로 테스트할 때 어려움이 있음
  - MobileConfig 카나리는 multi-stage (pre-land and post-land) 방식으로 접근
  - pre-land는 적은 사용자 군에게 짧은 카나리 시간으로 실행. 이 카나리는 무조건 먼저 적용되고, 다른 개발자들은 변경 사항을 적용하기 위해 카나리 완료까지 기다리는 구조.
  - post-land는 0.05% 정도의 많은 사용자 군에게 오랜 시간에 걸쳐(4시간 이상) 적용되고, 필요하면 중간에 조치함
- Emergency Push
  - 값 전파까지는 최대 24시간까지도 걸릴 수 있기 때문에 EP를 사용하면 앱의 구성 캐시 업데이트, 또는 앱 재시작 가능
  - MQTT, HTTP 헤더 등의 다양한 프로토콜을 사용할 수 있음
- Partial Fetch
  - 새로 앱 설치를 하거나 업데이트 되는 경우, 구성 캐시가 없을 수 있음
  - 이 때 기본값을 제공하거나 fetch를 기다리면 좋지 않을 수 있고, 애플리케이션 작동에 필요한 최소한의 구성을 식별해서 그것만 불러옴
  - 나머지 구성들은 비동기로 fetch
- 그 외 성능 향상, 페이로드 크기를 줄이기 위한 여러가지 노력들.. (스킴 해시, Value 해시, Boolean 최적화)

https://atscaleconference.com/mobile-configuration-at-meta-the-key-to-mobile-agile-development-at-scale/

### 11/21 목

PEP 723 (인라인 스크립트 메타데이터)

```python
# /// script
# requires-python = ">=3.11"
# dependencies = [
#   "requests<3",
#   "rich",
# ]
# ///

import requests
from rich.pretty import pprint
```

이런 식으로 Python 스크립트 맨 위에 주석으로 dep를 정의하면 호환되는 구현체들이 알아서 의존성들을 version constraint에 맞게 다운로드 받아서 실행하는 것 같음

https://peps.python.org/pep-0723/

uv, pdm, hatch 등 일부 도구들이 지원하고 있음

### 11/22 금

Kubernetes의 Sidecar Container

- 원래 Kubernetes Sidecar는 normal container로 많이 띄웠는데, Kubernetes 1.28부터 Native Sidecar 지원이 추가됨
  - 동일한 init container로 띄우는데, 새로운 속성이 추가되어서 완전히 다르게 동작함
  - 좋은 결정 맞아? → 맞다
- init container의 동작
  - normal container보다 먼저 트리거 됨
  - 정의된 순서대로 하나씩 시작되고 종료됨
  - 보통 일회성 스크립트(ex. Secret, Config fetch 등)
- 하지만 init container에 맞지 않는 경우도 있음
  - 네트워크 프록시, 로깅 및 메트릭 수집기, 모니터링 에이전트 등
  - 이런 경우 main container보다는 먼저 시작해야 하면서도 main container가 실행되는 동안만 실행되어야 하고, Pod의 ready 상태에 영향을 줄 수도 있음
  - 지금까지는 normal container를 통해 구현해왔는데, 순서 보장도 없고 restart policy가 하나라 힘들었음
  - 1.28 버전부터 sidecar container 등장
- init container에 등장한 restartPolicy
  - Pod 전체에 적용되는 restart policy 외에도 각 container가 restart policy를 가질 수 있게 됨
  - 단, init container이면서 깂이 Always인 경우에만 가능
- 차이점은 무엇일까?
  - 다음 init container, normal container의 시작을 차단하지 않음
  - 종료하면 재시작 됨
  - probe 지원도 됨
  - 다른 모든 일반 container가 내려가면 종료됨
  - 시작 순서를 따른다는 것 외에는 기존 init container와 매우 다르게 동작함 (normal container와 다른 것이 딱히 없는 듯)
- 읽어보니 아주 깔끔한 빙법인지는 모르겠지만 꽤 납득이 가는 결정인 것 같고, KEP-753도 기대하게 됨

https://labs.iximiuz.com/tutorials/kubernetes-native-sidecars
