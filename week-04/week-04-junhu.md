### 11/18 Actuator 안전하게 사용하기

Spring Actuator

Spring Actuator는 org.springframework.boot:spring-boot-starter-actuator 패키지를 Dependency에 추가만 해주면 바로 사용할 수 있는 기능으로, Spring boot를 사용하여 Backend를 구현할 경우 애플리케이션 모니터링 및 관리 측면에서 도움을 줄 수 있습니다.

Actuator 보안 이슈

Spring Actuator의 env endpoint가 필요하여 enable시키고 expose까지 시켜두었다면, 서비스에서 사용 중인 환경 변수를 볼 수 있게 되기 때문에, 의도치 않게 설정해둔 중요 정보가 유출될 수 있습니다.
Spring Actuator는 heapdump라는 endpoint를 제공함으로써 현재 서비스가 점유 중인 heap메모리를 덤프 하여 그 데이터를 제공해 주는 기능이 있어, 덤프 된 메모리 값을 통해 중요 정보가 유출될 위험이 있습니다.
Shutdown endpoint를 활성화하여 사용할 경우 문제가 될 수 있습니다. 기본적으로 Shutdown endpoint는 비활성화되어 있는데, 이를 임의로 활성화시킨 뒤 사용하고자 할 때 발생하는 문제로, Shutdown endpoint를 사용할 경우 임의로 웹 애플리케이션을 말 그대로 중지시켜버릴 수 있기 때문에, 서비스 가용성에 문제가 발생할 수 있습니다.

Spring Actuator 안전하게 사용하는 방법

Actuator는 shutdown endpoint를 제외한 나머지 endpoint는 enable 되어있는 것이 기본 설정입니다. 하지만 이 기본 설정 그대로 유지할 경우, 불필요한 endpoint가 활성화되어 추후 잠재적 위험이 될 수 있어, 기본 설정을 따르지 않겠다는 설정을 해주어야 합니다.

이때 사용하는 것이 management.endpoints.enabled-by-default 속성입니다.
해당 속성을 false로 만들어 줌으로써, 모든 endpoint에 대하여 disable 상태를 유지할 수 있습니다. 운영 중 필요한 endpoint가 있다면 management.endpoint.[endpoint name].enable 속성을 true로 설정하면 됩니다.

– shutdown endpoint는 enable하지 않는다.
shutdown endpoint는 말 그대로 웹 애플리케이션을 shutdown 시킬 수 있는 기능을 제공하기에, 서비스 가용성을 침해할 우려가 있음을 위에서 예시로 설명했었습니다. shutdown endpoint는 기본적으로 disable되며, expose도 되지 않기 때문에, 절대로 enable하지 않도록 각별히 신경을 써주어야 합니다.

– JMX형태로 Actuator 사용이 필요하지 않을 경우, 반드시 disable한다.
JMX는 Default로 expose되어있는 endpoint가 많기 때문에, 사용하지 않음에도 enable 시켜두면 잠재적 위험이 될 수 있습니다.

이에 JMX형태로 Actuator 사용을 하지 않는 경우 *management.endpoints.jmx.exposure.exclude = 형태로 속성을 추가함으로써, 모든 endpoint가 JMX로 사용 불가하게 설정해 주어야 합니다.**

– Actuator는 서비스 운영에 사용되는 포트와 다른 포트를 사용한다.
Actuator가 다양한 기능을 가진 만큼, 공격자들은 웹 사이트를 공격할 때 Actuator 관련 페이지가 존재하는지를 스캐닝 하는 경우가 굉장히 빈번합니다.

이때 서비스를 운영하는 포트와 다른 포트로 Actuator를 사용할 경우, 공격자들의 스캔으로부터 1차적으로 보호받을 수 있다는 장점이 있습니다. 이에 management.server.port 속성을 통해 서비스를 운영하는 포트와 다른 포트로 설정하여 사용할 것을 추천합니다.

– Actuator Default 경로를 사용하지 않고, 경로를 변경하여 운영한다.
포트 관련 항목에서 언급했듯이, Actuator가 다양한 기능을 가진 만큼, 공격자들은 웹 사이트를 공격할 때 Actuator 관련 페이지가 존재하는지를 스캐닝 하는 경우가 많은데, 이때 주로 알려진 URI형태의 스캐닝을 많이 수행하게 됩니다.

그렇기에 Actuator 서비스에서 주로 사용하는 알려진 기본 경로(/actuator/[endpoint]) 대신 다른 경로를 사용함으로써 외부 공격자의 스캐닝으로부터 보호받을 수 있기 때문에 경로 변경을 추천하고자 합니다.
management.endpoints.web.base-path 속성을 통해 설정이 가능하며, 유추하기 어려운 문자열의 경로로 설정함으로써 보안성을 향상시킬 수 있습니다.


### 11/19 Actuator 안전하게 사용하기2
지금까지 소개드린 보안대책들이 모두 반영된 application.properties 는 아래와 같습니다.

# Actuator 보안 설정 샘플

```
# 1. Endpoint all disable
management.endpoints.enabled-by-default = false

# 2. Enable specific endpoints
management.endpoint.info.enabled = true
management.endpoint.health.enabled = true

# 3. Exclude all endpoint for JMX and Expose specific endpoints
management.endpoints.jmx.exposure.exclude = *
management.endpoints.web.exposure.include = info, health

# 4. Use other port for Actuator
management.server.port = [포트번호]

# 5. Change Actuator Default path
management.endpoints.web.base-path = [/변경된 경로]
```
지금까지 안전하게 Actuator를 사용하는 방법에 대해서 작성을 해보았습니다.
제가 작성한 보안대책은 이론상 Best Practice이기 때문에 보안적으로 최대한 안전한 상태를 가이드하고 있습니다. 그렇기에 서비스 환경에 따라 조치 방안이 달라질 수 있다는 점 참고를 부탁드립니다. 반드시 현재 환경과 상황에 맞추어 충분한 검토 후 적용하실 것을 추천드립니다

### 11/20 prometheus와 grafana를 통한 시각화
https://bes99.tistory.com/36

### 11/21 jpa엔티티와 도메인 엔티티를 동일하게 사용할 지에 대한 고민에서 나온 DDD 공부
새로운 서비스를 준비하면서 컨벤션을 정의하는 과정에서 jpa엔티티와 도메인 엔티티를 동일하게 사용할지 분리해서 사용할 지에 대해서 팀원들과 논의를 하게 됨.

동일하게 가져갈 경우에 무분별한 연관관계 맵핑, application 모듈에 도메인 로직이 담기는 케이스 등과 같은 부작용이 발생하게 되었다는 의견이 있었음. 물론 app 모듈 단에 도메인 로직이 담기는 케이스는 분리할 때도 발생할 수 있지만 동일하게 사용할 때 더 많이 발생하는 것 같음.
특히 두번째 내용에 대해서 왜 app 모듈 단에 도메인 로직이 담기는 경우가 생길까에 대해서 고민을 해봤음.

고민에 대한 결론은 도메인 서비스를 제대로 활용하고 있지 않다는 생각이 들었음.

어떻게 제대로 사용하고 있지 않나면

첫 번째, 보통 도메인 로직을 엔티티 안에 다 넣으려고 하고 여러 애그리거트가 필요한 로직에 대해서 app 모듈 단에서 구현하는 부분이 있었음. → 도메인 서비스를 잘 사용할 수 있어야 됨.

두 번째, 도메인 서비스의 명칭과 위치에 대해서 애매했던 것 같음. 명칭은 해당 도메인서비스의 책임을 명확하게 드러낼 수 있는 용어+Service가 나을 것으로 판단되고, a애그리거트와 b애그리거트를 사용해서 도메인 서비스를 만든다 했을때 이 부분을 어디에 둘지 애매함. 이 부분은 좀더 고민해봐야 될 것 같음.

세 번째, 도메인 서비스내에 의존성을 주입받는 형태로 사용하는 케이스가 있어, 도메인 로직 테스트 코드를 짜는 데 방해를 끼침. 이 부분은 처음 시작하는 만큼 좋은 코드 사례를 많이 만들어 참고할 수 있게하고, 코드리뷰를 통해 지켜나가는 방향으로 해보는 걸로…

