### 11/19 화

> 상태관리팀 스터디에 꼽사리껴서 끄적여본 문장들입니다. 정리안됨 주의...:루피_눈물:
- zustand는 context없이 클로저 개념으로 전역 상태를 관리함.
- Object.is 얕은비교. 주소값만 비교해서 상태가 변경되었는지 판단
- useSyncExternalStore → 리액트18의 동시성 모드를 위한 핵심 훅
- zustand를 context로 감싸서 사용? provider가 리렌더링되는 경우는 없으니 더 안정적?
  - 불필요한 리렌더링을 방지하기 위해서 useStoreWithEqualityFn 와 같이 사용?
- vanilla store - 명령형. 구독 해제를 직접 관리
- react store - 선언형. 컴포넌트 생명주기와 함께 관리
- useShallow를 활용해서 useStore 최적화
  - 불필요한 리렌더링 방지
  - 구조분해 할당으로 여러 상태에 한꺼번에 접근하고 싶을 때 사용 (새로운 객체/배열 반환할 때)
  - 단일 상태를 추출하는 커스텀 훅 생성도 권장됨 → 테스트코드에서 유용하지 않을까?
  - 중첩 객체의 깊은 변화는 감지 못할 수도
- getState는 현재 최신화된 스토어를 가져옴. 구독은 안함
- zustand의 전역상태는 리액트 앱 외부에 존재. create를 사용한다면 providerless하게, createStore를 직접 사용한다면 provider와 함께?
- zustand의 immer 미들웨어와 valtio는 proxy 개념을 사용
  - immer는 완벽한 불변성을 보장하지는 못함. js의 특징때문이기도 함 → valtio 쓰라고 답변옴
  - 둘의 프록시는 조금 다름.
- immer 라이브러리
  - 메모리 최적화와 불변성 유지
- XState
  - 유한 상태 기계(FSM) → 게임 상태 패턴