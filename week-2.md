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
