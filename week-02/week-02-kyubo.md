### 11월 4일
1.  Xcode optimization level 종류
 - Onone: debug 모드 (최소한의 최적화와 디버그 정보 기록 o)
 - O: release 모드 (최적화 수행과 디버그 정보 기록 x)
 - Osize: 성능보다 코드 사이즈를 줄이는 것에 최적화
2.  릴리즈 앱에서 assertFailure가 발생했다?
> 그럼 assert가 머지?
- assert: 내부 조건이 false가 되면 runtime error를 발생하여, 주의해야한다는 의미로 사용
- assertionFailure: 별도의 조건은 없고 바로 runtime error 발생
- precondition: assert는 true여야 runtime error가 나지만, precondition은 false여야 runtime error 발생
- preconditionFailure: 별도의 조건은 없고 바로 runtime error 발생
- fatalError: 별도의 조건이 없고, 무조건 runtime error 발생
3. 적용되는 범위
> (-Oneon은 debug모드의 디폴트 값, -O은 Release 모드의 디폴트 값)
- assert, assertFailure: -Onone일때만 활성화
- precondition, preconditoinFailure: 항상 활성화
- fatalError - 항상 활성화
4. 결론
- debug 모드 (Onone) 에서만 runtime error가 나가끔 하려면 assert, assertFailure 사용
- release 모드 (O) 에서도 runtime error가 나게끔 하려면 precondition, preconditionFailure 사용
- assert, precondition 조건 없이 에러가 나는 의미를 전달하고 싶을땐 fatalError 사용
- 
### 11월 6일
대부분 회사에서 "네트워크 통신 == Alamofire == 편함" 사용을 디폴트로 여기는 시점에서 다시금 생각나서 읽어보았어용

URLSession을 사용하여 async/await 방식으로 네트워크 요청을 수행하는 방법.
- Alamofire 같은 서드파티 라이브러리가 널리 쓰이지만, URLSession의 기본 기능을 활용해도 안정적인 네트워크 통신이 가능하다는 점이 인상적임.
- 생각과는 다르게 GET 요청은 물론 URLComponents를 활용해 파라미터를 추가하거나 POST 요청에서 데이터를 전송하는 과정도 생각보다 꽤 간단히 구현 가능함
- 추가로 async/await을 활용해서 비동기적으로 처리도 충분히 가능하고, 에러 핸들링 역시 추상화없이 세분화된 에러 타입으로 관리가능함. 코드 가독성도 나쁘지 않음
- 서드파티 대신 대신 퍼스트파티만트로 외부 종속성 없이충분히 견고한 네트워킹 레이어를 구축할 수 있는거 다시 확인하게 되었어용.


### 11월 8일
카카오에서 tuist를 쓰면서 대한민국이 전세계 tuist 사용자 2위국가가 됐다. 블로그보면 너도나도 써보고 있는데 부정적인 입장에서 적어본 TIL
1.종속성 문제
- 만약 다 적용해놨는데 바꿔야될 일이 생기면? 그땐 진짜 머리아파 질거 같다.
- 이 친구가 자동 생성한 프로젝트 파일을 직접 관리해야 한다면, 수동으로 모듈별 의존성을 다시 설정하고 스킴과 타깃 구성도 전부 수작업으로 만들어야 될듯. 적고보니 1번 2번이 비슷하지만 같은 얘기일래나
- 특히 모듈 구조가 복잡하면 설정 충돌을 해결하는 데 꽤나 골치 아픈 작업이 될 수 있을 것 같다. 혹 떼려다 붙히는 꼴이 될수도
2.작은 팀에겐 복잡함
- 당연한 얘기겠지만 소규모 팀에는 오버테크가 될수도 있겠다
3. 학습 곡선?
- 회사입장에서는 쓸줄 아는 사람을 뽑으면 되지만 이래서 취준생들이 많이 하나보다..
결론) 기존의 xcode를 사용하면서 가려운부분을 확실히 긁어주는것 맞지만 신규 프로젝트가 아니면 굳이? 인가 싶음
