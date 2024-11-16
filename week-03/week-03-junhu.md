## 11/11 부하테스트 k6
k6는 고성능 부하 테스트를 위한 오픈 소스 도구로, JavaScript 기반의 스크립트를 사용하여 API 및 웹 애플리케이션의 성능을 테스트할 수 있습니다. 경량화된 CLI 환경에서 빠르게 테스트를 수행할 수 있도록 설계되었으며, 특히 DevOps 환경에서의 통합과 자동화에 강점을 가지고 있습니다.

### 주요 특징
1. JavaScript 기반:
k6는 JavaScript (ES6+) 스크립트로 작성되며, 개발자에게 친숙한 환경을 제공합니다.
JavaScript로 작성된 테스트 코드는 직관적이며 재사용 가능하여 유지보수가 쉽습니다.

2. 경량화된 CLI 도구:
k6는 가벼운 CLI 도구로, 별도의 UI 없이 명령줄에서 실행됩니다.
설치와 실행이 간단하며, 빠르게 부하 테스트를 시작할 수 있습니다.

4. 비동기 및 비블로킹 I/O:
Node.js와 유사한 비동기 구조를 가지고 있어 다수의 가상 사용자(VU) 요청을 효율적으로 처리할 수 있습니다.
k6는 싱글 프로세스로 동작하지만, 이벤트 루프를 통해 다수의 VU 요청을 비동기적으로 관리하므로 고성능 부하 테스트가 가능합니다.

5. 스테이지 설정을 통한 부하 패턴 시뮬레이션:
k6는 부하 테스트 시나리오에서 단계별 사용자 증가/감소를 설정하여 다양한 부하 상황을 시뮬레이션할 수 있습니다.
이를 통해 실제 사용자 증가 패턴을 모방하거나 스파이크 테스트를 수행할 수 있습니다.
6. 통합성 및 확장성:
k6는 JSON, CSV 포맷 외에도 Prometheus, Grafana, Datadog 등의 모니터링 도구와의 연동을 지원하여 테스트 결과를 손쉽게 시각화할 수 있습니다.
CI/CD 파이프라인에 쉽게 통합할 수 있어 DevOps 환경에서 성능 테스트를 자동화하기에 적합합니다.

### k6의 주요 개념
Virtual User (VU): 테스트에서 사용자 역할을 하는 가상의 유닛으로, 각 VU는 독립적으로 스크립트를 실행합니다.
Stages: 테스트 시나리오를 단계별로 정의하여 특정 시간 동안 VU 수를 증가하거나 유지할 수 있는 설정입니다.
Threshold: 성공 기준을 설정하여 요청의 응답 시간, 에러율 등의 임계값을 지정하고 이를 통과 여부를 기준으로 삼을 수 있습니다.

### 예제: 간단한 k6 부하 테스트
```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 10 },   // 1분 동안 VU 10명까지 증가
    { duration: '5m', target: 50 },   // 5분 동안 VU 50명 유지
    { duration: '1m', target: 0 },    // 1분 동안 VU 0으로 감소
  ],
  thresholds: {
    http_req_duration: ['p(95)<2000'],  // 95%의 요청이 2초 이내에 응답해야 함
  },
};

export default function () {
  const res = http.get('https://example.com/api');
  check(res, {
    'status was 200': (r) => r.status === 200,
  });
  sleep(1);
}
```
예제 설명
Stages: 1분 동안 VU 10명으로 증가, 이후 5분 동안 50명의 VU를 유지하며, 마지막 1분 동안 0명으로 감소하는 시나리오를 정의합니다.
Thresholds: 요청의 응답 시간이 95% 이상 2초 이내로 반환되어야 테스트가 성공하도록 조건을 설정합니다.
check: 응답 상태가 200인지 확인하여 테스트 중 상태 검증을 수행합니다.

## 11/13 k6를 통해 SSE api 테스트 환경 구축
### k6 SSE(server-sent-event) 부하테스트 환경 구축 방법 
1. go 설치 https://go.dev/doc/install
2. k6 설치 https://subeen-lab.tistory.com/128
3. xk6 설치 `go install go.k6.io/xk6/cmd/xk6@latest`
4. xk6-sse git clone `git clone https://github.com/phymbert/xk6-sse`
5. 해당 디렉토리로 들어가서 xk6-sse 빌드  `xk6 build`
6. 부하테스트 실행 cmd `./k6.exe run ssetest.js`

### 참고
* xk6-sse 바이너리 빌드시 해당 레포에 있는 것처럼 github에서 바로 빌드하면 실패 떨어짐. 클론한 것을 빌드하는 것을 추천.
* 안되면 환경변수에 GOBIN(C:\Program Files\Go\bin), GOROOT(C:\Program Files\Go), GOPATH(%USERPROFILE%\go) 설정되어 있는지 확인해서 추가하기.

## 11/14 실제 테스트
### api 테스트 코드
```js
//파일명 ssetest.js
import sse from "k6/x/sse";
import {check} from "k6";

export default function () {
    const url = "https://echo.websocket.org/.sse";
    const params = {
        method: 'POST',
        body: '{"ping": true}',
        headers: {
            "content-type": "application/json",
            "Authorization": "Bearer XXXX"
        }
    }

    const response = sse.open(url, params, function (client) {
        client.on('event', function (event) {
            console.log(`event id=${event.id}, name=${event.name}, data=${event.data}`);
            if (parseInt(event.id) === 2) {
                client.close()
            }
        })
    })

    check(response, {"status is 200": (r) => r && r.status === 200})
}

```
### 명령어 실행
`./k6.exe ssejest.js`

## 11/15 후기
1. js로 코드를 짜서 생각보다 gpt의 도움을 수월하게 받을 수 있었다. ngrinder로 찍먹했었으나 코드 작성이 groovy여서 gpt를 돌렸을때 라이브러리라던지 문법이 틀렸을 때가 있었음.
2. sse api를 테스트하기 위해서 별도로 익스텐션을 설치해야하나 window 환경에서 여러 이슈가 있어서 살짝 불편했다.
3. 그럼에도 불구하고 테스트하기에는 너무 편리했음.
