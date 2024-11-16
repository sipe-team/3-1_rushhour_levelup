### 11/1 금

> 도커 컨테이너가 "더 빨리 뜨면" 좋겠다. 어떻게 하면 빨리 띄울 수 있을까?

- 단순 도커 이미지 크기는 생각보다 큰 영향을 미치지 않는다.
- 실제 영향을 미치는건 Docker Layer의 수다. 레이어를 로드하는 오버헤드를 줄여라.
- 도커 이미지 레이어는 Dockerfile의 한 줄마다 새로운 이미지 레이어를 쌓는게 아니다. 파일 시스템에 변화가 생길 때, 하나의 레이어를 쌓는다.

#### Reference

- [Reddit - 도커 이미지 크기가 컨테이너 시작에 끼치는 영향](https://www.reddit.com/r/googlecloud/comments/1bponng/docker_image_size_impact_on_container_startup)
- [최적화 아티클](https://www.fullstack.com/labs/resources/blog/small-is-beautiful-how-container-size-impacts-deployment-and-resource-usage)
