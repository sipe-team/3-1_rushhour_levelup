### 11/5 Alpine linux와 python
- Alpine linux 에서는 gclib 대신 musl을 사용함
- musl 라이브러리는 가볍지만, 모든 바이너리에 static하게 연결해야하는 단점 보유
- 따라서 C 구현체 라이브러리가 많을수록, alpine linux를 사용하면 빌드 시간이 매우 늘어남
- glibc 라이브러리를 사용하는 이미지 (slim 등) 을 추천합니당
