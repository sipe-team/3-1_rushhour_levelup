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
