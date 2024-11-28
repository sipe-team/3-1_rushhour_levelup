### 11/25 월

`HTTP_PROXY`가 문득 신기해서 찾아봐서 정리하고 올린 글이에요

- `HTTP_PROXY` 사용하면 다양한 라이브러리, 소프트웨어들이 자동으로 HTTP 요청들을 프록시 서버를 태워 보냄
- 그렇다면 이 Proxy 서버는 어떻게 구현이 되는 건지 궁금해서 테스트 해봤는데
  - 프록시 서버에 Path로 Target 서버의 "전체 URL"이 전달됨
  - Path는 `/` 로 시작해야 되는 거 아닌가 싶었는데..
  - [RFC 7230](https://www.rfc-editor.org/rfc/rfc7230)의 중개자(intermediary) - Proxy에서 정의하는 `absolute-form` 은 Request Target이 일반적인 path가 아닌 full uri를 그대로 넣을 수 있도록 정의하고 있음
- 실제로 Proxy 서버를 구현하면 고민해야 할 점
  - Transfer-Encoding이나 일부 헤더들이 무한 루프를 일으킬 수 있어서 빼고 응답으로 내려준다거나 하는 것들이 필요함
  - Streaming 요청/응답 잘 처리해줘야 함

[https://medium.com/@sudosubin/http-proxy-환경-변수는-뭘까-a1471a4e5cee](https://medium.com/@sudosubin/http-proxy-%ED%99%98%EA%B2%BD-%EB%B3%80%EC%88%98%EB%8A%94-%EB%AD%98%EA%B9%8C-a1471a4e5cee)

### 11/26 화

네트워크에서의 연결

- Socket(BSD Socket): 통신을 위한 인터페이스
  - 리눅스의 철학 상 파일이고, 소켓을 구분할 때에도 파일 디스크립터를 사용함
  - 파일이기 때문에 연결의 수가 파일 정책을 적용 받아 제한될 수 있음
  - 하나의 프로세스는 여러 소켓을 열 수 있음
- Network Socket: 일반적으로 웹에서 네트워크 통신에 쓰이는 소켓
  - Transport Layer에 대한 이해 없이 API의 형태로 통신 흐름을 제어할 수 있음
  - 웹소켓(7계층)과 소켓(4계층)을 혼동하지 말자
- Unix Domain Socket: 운영체제 내에서 프로세스간 통신을 위한 소켓
  - 흔히 보이는 .sock 파일
- Client, Server
  - Active Close(연결을 끊는 주체), Passive Close(연결 끊는 요청을 받는 대상)
- DSR(Direct Server Routing)
  - 요청은 클라이언트 > LB > 서버
  - 응답은 서버 > 클라이언트
  - TCP Keepalive에 의해 클라이언트와 서버 간의 연결이 유지될 수 있음

https://de-vlog.vsfe.me/blog/network-connection/

### 11/27 수

Vite 6.0이 나왔어요

- 주요 변경 사항은 Environment API ([소개글](https://green.sapphi.red/blog/increasing-vites-potential-with-the-environment-api))
  - Vite 5.1에서는 Vite Runtime API 이었다가 이름 변경됨
  - Node.js 외의 다른 런타임도 많이 사용되기 시작 (Deno, Bun, Vercel Edge Runtime, Cloudflare Workers)
    - RSC 사용하면 클라이언트, SSR, RSC 총 3개의 번들을 처리해야 함
    - Vite는 브라우저 번들 하나, SSR 번들 하나만 가정했고, SSR은 항상 Node.js에서 실행된다고 가정했었음
  - 기존 구조에서는 SSR을 할 때 Vite의 Code Transformer이 호출되고, 변환된 코드가 실행됨. 이 때 Code Transformer, Code Executor 모두 Node.js 의존성으로 Node.js에서만 실행 가능했음
  - 그래서 Environment API에서는 Code Executor와 Code Transformer를 분리하여 Function Call 대신 다른 무언가로 이루어지도록 함 (클라이언트가 HTTP로 호출하듯, 이를 Runner Transport라고 함)
    - 이렇게 하여 Code / Code Executor / Code Transformer를 모두 횡으로 분리 (격리)
    - 설명이 장황한데 블로그 글의 이미지를 보면 뭔 말인지 더욱 쉽게 이해 가능
  - Browser, Service Worker, Edge Server, Origin Server를 모두 지원하는 신비로운 구조도 가능
  - 생태계의 대부분의 플러그인들이 테스트를 통과했지만 테스트는 필요할 것 같은 느낌
- 이외에는 VoidZero, ViteConf 관련 소개

https://vite.dev/blog/announcing-vite6

### 11/28 목

CORS 요리조리 피해가기

- CORS를 사용하면 Preflight 요청 (OPTIONS 메소드) 사용으로 인해 Round Trip이 한 차례 더 발생하고, 성능 저하가 발생할 수 있음
- 캐시
  1. `Access-Control-Max-Age` 헤더를 이용해 OPTIONS 요청을 브라우저에서 캐시할 수 있음
  2. HTTP 메소드와 URL(Query Param 포함)이 캐시 키
  3. 단 Firefox는 1일, Chromium은 2시간으로 최대 제한이 있음
- iframe 사용하기
  1. Origin이 동일하면 CORS가 발생하지 않기 때문에 같은 Origin을 iframe 내에 띄워서 window 간 postMessage로 통신하는 방법
  2. 이 오버헤드가 더 클 수도 있고, 구조가 복잡해지는 단점이 있음
- Proxy 사용하기
  1. 내부적으로 웹 페이지의 특정 경로를 실제 Target 서버로 프록시 해주는 방안
  2. 네트워크의 홉이 생기므로 성능을 고려해야 함
  3. 프론트엔드의 SSR 서버에서 프록시해주기보다는 CDN, LB 레벨에서 해주면 좋을 것 같음
- Simple Requests 사용하기
  1. HTTP 메소드(GET, HEAD, POST), 헤더 등의 조건에 따라 어떤 간단한 요청들은 Origin이 달라도 CORS 요청 없이 호출할 수 있음
  2. 이를 위해 커스텀 헤더들은 페이로드나 쿼리 파라미터로 넣는 마개조를 할 수도 있음

https://webperf.tips/tip/optimizing-cors/

### 11/29 금

Quickwit으로 100PB 로그 서비스 만들기 (바이낸스)

- Quickwit의 목표는 ElasticSearch보다 10배 이상 저렴하고 구축/운영하기 쉽고, 스케일 가능한 것
- 일반적인 테스트는 100TB, 1GB/s 인덱싱에서 진행했고, 그 이상은 데이터 세트와 컴퓨팅 리소스의 부족으로 어려웠음
- 바이낸스의 2명의 엔지니어가 실험을 시작했고, 몇 달만에 성공!
  - PB 규모의 ElasticSearch 클러스터에서 Quickwit으로 마이그레이션 완료
  - 하루 1.6PB 인덱싱
  - 100PB 로그 처리
  - 컴퓨팅 비용 80% 감소, 저장 비용 20배 감소
- 초당 2,100만 개의 로그, 18.5GB/s, 하루 1.6PB
- 기존에는 20개의 ElasticSearch 클러스터로 처리. 600개의 Vector Pod가 Kafka 토픽에서 로그를 끌어와 처리하고, ES에 적재
  - 운영하기 어렵고, 로그를 장기간 보관하기에는 비용도 어려웠음
  - 비용과 성능을 고려하면 가용성을 포기하고 복제 없이 구성해야 했음
- Quickwit은 거의 완벽했는데,
  - Native Kafka 통합, Vector 없이 로그 변환, S3로 Stateless 구성 가능
  - ES 대비 2배 더 높은 압축률
  - 하지만 아직 PB 규모를 감당할 수 있을지 알 수 없었음
- 바이낸스에서 테스트 해보니 2가지 문제를 발견함
  - 클러스터 안정성: Gossip 프로토콜이 수백 개의 Pod 환경에서 불안정했고, 인덱싱 처리량이 안정적으로 유지되지 못함
  - 불균형한 작업 분산: 인덱스에 따라 처리량이 달랐고, 문제 해결 예정인 상태
    - 이븐하지 못했대요 ㅇㅁㅇ
  - 이를 해결하기 위해 고처리량이 필요한 토픽은 별도의 인덱싱 클러스터를 할당했고, 이렇게 격리하더라도 Stateless 구성이 가능해서 운영 부담이 없었음
  - Vector Pod 들도 모두 내림
  - 10개의 인덱싱 클러스터, 700개의 Pod, 6TB의 메모리로 1.6PB 인덱싱 처리 성공 (vCPU 1개당 6.6~11MB/s)
- 검색 확장
  - 10개의 인덱싱 클러스터를 사용하면 각 클러스터마다 Searcher Pod 배포가 떠야하고, Searcher 리소스를 폴링해서 모든 인덱스가 공유하는 Object Storage에 접근하기 어려워지는 문제가 있었음
  - 바이낸스에서는 각 인덱싱 클러스터 메타스토어의 모든 메타데이터를 하나릐 PG DB로 복제해서 유지함
  - 이를 통해 모든 인덱스를 검색할 수 있는 중앙 검색 전용 클러스터 배포할 수 있게 됨 😮
  - 현재 30개의 Searcher Pod 유지 중

https://quickwit.io/blog/quickwit-binance-story
