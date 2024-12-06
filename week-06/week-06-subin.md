### 12/02 월

GitHub이 저장소 크기를 줄인 방법

- GitHub는 19PB의 데이터를 저장하고 있음
- Git/GitHub에서는 Unreachable Data가 발생함
  - 브랜치를 삭제할 때, 브랜치에 대한 참조는 사라지지만 객체들은 남아있음
  - force push 할 때, 이전 커밋들은 참조를 잃음
  - 민감한 정보를 삭제할 때, 새로운 커밋이 생성되면서 이전 커밋들은 접근 불가하게 됨
- git의 gc 기능이 존재하고 자동으로 수행되며, 수동으로도 트리거 가능
  - 일반적으로 삭제해도 안전할 것으로 간주되는 - 약 2주 이상 오래된 객체들만 삭제함
  - 비교적 최근에 삭제된 객체들은 git의 objects 폴더에 저장되며, 이를 loose objects라고 함
  - gc는 reachable loose object들을 packfiles로 압축함
  - unreachable loose object들은 packfiles에 추가되지 않음 (마지막 수정 시간을 알 수 있도록)
- 대규모 프로젝트에서는 많은 unreachable loose object들을 생설할 수 있어 gc가 효율적이지 않음
  - GitHub의 Cruft pack은 unreachable loose object들도 압축하는 방법
  - 압축이 되면 object들의 수정 시간을 확인하기 어려워지는데, 이 문제는 별도의 .mtimes 파일을 두어서 해결함
  - .idx 인덱스 파일을 통해 object의 id와, packfile 내의 위치(오프셋) 값을 포함함
  - git 2.37.0에 추가되어 이용 가능

https://newsletter.betterstack.com/p/how-github-reduced-repo-storage-size

### 12/03 화

한 줄에 JSON이 하나씩 들어가면 JSON Lines

- 비슷한 이름들
  - NDJSON - Newline delimited JSON
  - LDJSON - Line delimited JSON
  - 그리고 JSON Lines까지 총 3가지의 표준에 가까운 구현체들이 있어서 혼동될 수 있음
- `\n`이나 `\r\n`으로 구분된 JSON 문자열들이 한 줄 한 줄 들어가있으면 JSON Lines 임
- Amazon Athena도 `org.openx.data.jsonserde.JsonSerDe` 사용하면 JSON Lines 읽을 수 있는데 아래 사이트에는 없음 :(
- Amazon DynamoDB에서 S3로 Export 하면 그 파일 포맷도 JSON Lines 임

https://jsonlines.org/on_the_web/

### 12/04 수

Amazon Athena Tips

- Athena는 파티션이 매우 중요. 폴더를 나눠서 파티션을 타게 쿼리를 짜면 매우 좋습니다. (ex. [`s3://foo/bar/year=2024/month=12/day=03/xxx.json.gz`](s3://foo/bar/year=2024/month=12/day=03/xxx.json.gz))
- Athena를 S3 데이터를 기반으로 쿼리할 때, S3 Prefix에 파일들이 너무 쪼개져있으면 좋지 않다. Round Trip 시간이 오래 걸리므로 적절한 단위로 파일들을 합쳐주면 좋습니다.
- Athena는 한 번 실행한 쿼리의 결과가 S3 어딘가에 남습니다. 이 Query Execution은 일종의 Snapshot처럼 재활용이 가능해서, Query Execution을 이용해서 계속해서 해당 시점의 쿼리된 데이터를 조회할 수 있습니다.
- DynamoDB를 PITR로 export 하면 S3 Bucket에 `.json.gz` 파일로 저장되는데, 이를 Athena에서 바로 읽어갈 수 있다.
- Athena는 느려서 그냥 왠만하면 안 쓰는게 좋은 것 같습니다.

### 12/05 목

Amazon SQS의 잘 안 알려진 제한

- Standard 큐는 최대 약 120,000개의 in flight 메시지까지 들고 있을 수 있고, 그 이상 생성하게 되면 OverLimit 오류가 발생한다. (무적이 아님)

### 12/06 금

CloudEvents 알아보기

- CNCF의 Graduated 프로젝트
- Vendor에 종속되지 않은 이벤트 명세
- Schema만으로도 어떤 상황, 어떤 사건에 대해 발행된 이벤트 메시지인지 이해할 수 있어야 함
- 아래는 예시

```json
{
    "specversion" : "1.0",
    "type" : "com.github.pull_request.opened",
    "source" : "https://github.com/cloudevents/spec/pull",
    "subject" : "123",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "data" : {
        "profileName" : "sabarada"
    }
}

```

- 다양한 이벤트 전달 방법(http, kafka)들에 대한 전송 방식이 마련되어 있고, sdk로 쉽게 가져다 사용할 수 있음

꽤 좋은 것 같아서 나중에 이벤트를 새로 정의할 때 고민해볼 것 같은 느낌.

https://github.com/cloudevents
