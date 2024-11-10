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
