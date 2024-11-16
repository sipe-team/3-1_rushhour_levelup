### 11/11 월

FMOps라는 것을 듣게 되어서 이건 또 무엇인가.. 찾아보고 기록

- FMOps(Foundation Model Operations)는 파운데이션 모델을 개발, 배포, 운영, 모니터링하는 것을 돕는 것
- MLOps와는 도대체 뭐가 다른가
  - 훨씬 더 큰 대규모의 파운데이션 모델을 대상으로 한다는 점 (ex. LLM)
  - 방대한 데이터셋
  - 이후 다운스트림에서 다양하게 사용할 수 있도록 하는 것도 지원
- 이것 외에는 딱히 MLOps와는 다른 것이 없는 듯

> LLMOps와는 또 무엇이 다른가는 나중에 또 찾아봐야지..
> > FMOps요...???!! 멀고도 험난한 옵스의 길....
>
> > 정말 웁스네요

### 11/12 화

Medallion Architecture

- 레이크 하우스에서 데이터를 논리적으로 Bronze > Silver > Gold 계층의 테이블로 계층화하는 것 (그래서 이름이 메달리언 인 듯)
- Bronze
  - Raw 데이터, 모든 소스 데이터를 그대로 적재하고 시간, 프로세스 ID 등만 추가로 저장
  - 필요하면 이후 계층에서 데이터를 다시 수집하지 않고 Bronze 계층에서 재처리 할 수 있음
- Silver
  - 정제된 데이터, Bronze 계층의 데이터를 연결하거나 합치거나 정리해서 보기에 유의미한 데이터를 만듦
  - 데이터 과학자들에게 비즈니스 문제를 분석하고 해결하기 위한 소스이기도 함
- Gold
  - 큐레이션 된 비즈니스 레벨의 데이터
  - Reporting을 위한 데이터로, 비정규화 되어 즉시 사용 가능 (최종 Presentation 데이터)
- 왜 이런 아키텍처를 쓰는가
  - 간단하고 이해하기 쉬움 (확실히 이름이 제일 어려운 아키텍처 같다고 생각)
  - 증분형 ETL (공감)
  - 언제든지 원시 데이터로부터 데이터를 다시 생성 가능 (공감, 저도 언제나 Raw 데이터는 별도로 보관되어 있어야 한다고 생각)

https://www.databricks.com/glossary/medallion-architecture

### 11/13 수

JSON:API

- JSON 응답에 페이지네이션, 모델 간의 관계 등을 표준으로 담기 위한 노력
- 이렇게까지 할 필요가 있을까 생각이 들었는데요,
- API를 ORM으로 사용하려는 SDK들을 보면서 나쁘지 않을 수도 있겠다는 생각도 들었습니다.
  - https://github.com/socialwifi/jsonapi-requests/
  - https://github.com/dipscope/JsonApiEntityProvider.TS
- 하지만 제가 만들고 싶지는 않아졌습니다...

https://jsonapi.org/

### 11/14 목

(회사 워크샵으로 면제 신청)

### 11/15 금

(회사 워크샵으로 면제 신청)
