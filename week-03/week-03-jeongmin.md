### 11/14 목

> 휴 퇴사를 (곧) 하니 출퇴근 공부를 해야한다는 자각이 안되네요.. 어제도 암 생각없이 패스해 버렸습니다ㅠ  
> 오랜만이어요 여러분~

- useImperativeHandle
  - ref로 노출되는 핸들을 사용자가 직접 정의할 수 있게 해주는 React 훅
  - forwardRect로 컴포넌트에 ref를 전달할 수 있지만, 돔 객체에 한정되어 있음. (함수형 컴포넌트의 한계)
  - useImperativeHandle를 사용하면 ref에 할당되는 값을 돔 객체가 아닌 컴포넌트 내부에서 커스텀한 객체로 변경할 수 있음.
  - 이러한 명령형 훅은 리엑트에서 권장되는 방향은 아님. 대부분의 경우 props와 useEffect로 구현할 수 있음. 돔 객체 함수를 실행해야하는 경우처럼 꼭 필요할 때만 사용하는 게 좋을 듯.
  - ref 자체가 권장되는 방향은 아니지만 쓸 수밖에 없다는 점에서, 부모에게 자식의 메서드를 넘기고자 할 때 유용하게 잘 쓸 수 있을 듯
- branded types
  - __brand 라는 사용되지 않는 프로퍼티를 추가해서 타이핑 용도로 쓰는 개념
  - 기존 타입에 프로퍼티나 라벨들을 추가하여 보다 명확하고 구체적인 새로운 데이터 타입을 만들 수 있음.
  - 실질적으로는 같은 자료형이더라도 명확히 구분지어 런타임 에러를 방지하게 해주는..
  - 유사하지만 서로 다른 모델들이 어지럽게 사용되는 프로젝트라면 유용할 것 같은데 일반적인 프로젝트에서는 굳이?일 것 같기도