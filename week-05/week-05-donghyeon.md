## 11/26
Drizzle ORM  
Prisma가 대세인 JS,TS ORM에 새롭게 나타난 ORM  
헤드리스 ORM: 프로젝트 구조에 간섭하지 않고 유연성 제공  
SQL 친화적: SQL 유사한 구조(낮은 러닝커브), 다른 ORM과 마찬가지로 쿼리 API도 제공  
```
await db
	.select()
	.from(countries)
	.leftJoin(cities, eq(cities.countryId, countries.id))
	.where(eq(countries.id, 10))
-------------------------------------------------------------------------------
const result = await db.query.users.findMany({
	with: {
		posts: true
	},
});
```
Zero 의존성: 단 0개의 외부 종속성으로 가볍고 뛰어난 성능, 서버리스에 적합한 설계  
Prisma 성능에 아쉬움이 있다면 고려해볼만도?  
https://orm.drizzle.team/docs/overview  

## 11/28
RSC가 SPA에 미치는 영향  
RSC는 React Server Component의 약자이지만 여기서는 좀더 포괄적인 아키텍처를 지칭  
- 핵심기능 : React 컴포넌트와 다른 값들을 직렬화하고 역직렬화할 수 있는 능력

SPA를 위한 RSC의 즉각적인 이점  
- 런타임 서버가 없다면 RSC가 SPA에 이점이 없어 보일 수 있지만 특정 이점을 제공함.  
- RSC의 직렬화 기능은 JS 번들 크기를 줄이고 싶을 때 SPA에 장점이 있음. 빌드 시점에 컴포넌트를 렌더링하고 직렬화할 수 있다.
- RSC 페이로드는 텍스트이며 정적 파일로 제공될 수 있음. 즉 페이로드를 생성하는 JS 코드를 보낼 필요가 없다.
- RSC 페이로드의 정적 파일은 초기에 로드될 필요가 없음. 일부 클라이언트 작업을 빌드 프로세스로 오프로드할 수 있다.
- 단점 : 어떤 것을 서버 컴포넌트로, 어떤 것을 클라이언트 컴포넌트로 만들어야 할지 고민해야 한다는 것
  
SPA용 API 서버가 있는 경우 RSC가 할 수 있는 일  
- 일반적으로 API 응답에 JSON을 사용하지만, 이를 RSC 페이로드로 대체할 수 있다. 이를 이용한다면
  - 렌더링된 컴포넌트와 데이터 동시 전송
    - 다시 컴포넌트를 렌더링하기 위한 JS 번들을 보내지 않아도 됨을 의미.
  - 데이터 스트리밍 지원
    - 데이터를 시간에 따라 청크 단위로 보낼 수 있어, 사용자는 데이터가 도착하는 대로 볼 수 있음.
  - 데이터 형식을 정의할 필요가 없음
    - RSC 페이로드는 클라이언트에서 그대로 렌더링 가능.

https://blog.axlight.com/posts/thoughts-on-what-rsc-means-for-spas/ 
