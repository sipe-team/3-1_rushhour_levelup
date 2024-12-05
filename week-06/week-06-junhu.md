### 12/2 WebClient 처리 방법 
WebClient 를 호출한 MainThread 에서 실행되는 영역
```java
    static {
        this.webClient = WebClient.builder().baseUrl("http://127.0.0.1:8090")
                .clientConnector(new ReactorClientHttpConnector(HttpClient.create().compress(true)))
                .build();
    }

	@PostConstruct
    public void webClientTest() throws InterruptedException {
        while(true) {
            Thread.sleep(30000);
            webClient.post()
                .uri("/hello")
                .bodyValue("Woongs Test")
                .exchange()
                .flatMap(clientResponse -> {
                    return clientResponse.bodyToMono(String.class);
                })
                .doOnNext(System.out::println)
                .subscribe();
        }
    }
```
@PostConstruct 를 사용하여 주기적으로 WebClient 가 호출되도록 테스트 코드 작성.

localhost 의 8090 포트로 post 요청을 30초마다 보내게된다.

 

PooledConnectionProvider.java
```java
		@Override
		public void onNext(PooledRef<PooledConnection> value) {
			pooledRef = value;

			PooledConnection pooledConnection = value.poolable();
			pooledConnection.pooledRef = pooledRef;

			Channel c = pooledConnection.channel;

			if (c.eventLoop().inEventLoop()) {
				run();
			}
			else {
				c.eventLoop()
				 .execute(this);
			}
		}
```
Channel 을 관리하는 PooledConnectionProvier 가 event 를 전달받어 onNext() 가 호출됨.

value 는 현재 처리할 PooledConnection 이 담겨있으며 value.channel 에 connection 맺은 Channel 이 들어있음

 


실행중인 Thread
@PostConsturct 에서 WebClient 를 실행하였으므로 WebClient 를 호출한것은 Main 쓰레드이다.

따라서 Else 문으로 전달된다.


Channel 에는 해당 Channel 의 Netty NioEventLoop 를 변수로 가지고 있어 NioEventLoop 로 Post 요청을 Execute() 한다.

 

SingleThreadEventExecutor.java (NioEventLoop.java)
```java
    private void execute(Runnable task, boolean immediate) {
        boolean inEventLoop = inEventLoop();
        addTask(task);
   		
        ...
    }
```
execute() 내부에는 addTask() 를 호출하고 있다.

 
```java
    protected void addTask(Runnable task) {
        ObjectUtil.checkNotNull(task, "task");
        if (!offerTask(task)) {
            reject(task);
        }
    }
    final boolean offerTask(Runnable task) {
        if (isShutdown()) {
            reject();
        }
        return taskQueue.offer(task);
    }
```
addTask() 함수를 호출하여 실행되어야 하는 POST 요청을 NioEventLoop 의 taskQueue 에 저장한다.

 

여기까지 실행되면 WebClient 를 호출한 MainThread 의 역할은 종료되는것을 알 수 있다.

그렇다면 add 된 Task 는 어디에서 처리가 되는것인지 확인해야 한다.

### 12/3 NioEventLoop 영역

 

아래 분석에서 알 수 있듯이 NioEventLoop 는 Channel 에서 발생한 IO Event 뿐만 아니라 Task Queue 의 Task 도 처리하는 역할을 가지고 있다.

 

https://woooongs.tistory.com/73

 
Netty NioEventLoop 코드 레벨 분석

Netty 의 NioEventLoop.java 코드 중 핵심이 되는 run() 메서드 분석 MacOS 를 사용중이기 때문에 KqueueSelectorImpl.java 가 사용되었다. EventLoop.java 는 두가지를 처리한다. I/O : Channel 로 부터 발생한 E..

woooongs.tistory.com
 

NioEventLoop.java
```java
                if (ioRatio == 100) {
                    try {
                        if (strategy > 0) {
                            processSelectedKeys();
                        }
                    } finally {
                        // Ensure we always run tasks.
                        ranTasks = runAllTasks();
                    }
                } else if (strategy > 0) {
                    final long ioStartTime = System.nanoTime();
                    try {
                        processSelectedKeys();
                    } finally {
                        // Ensure we always run tasks.
                        final long ioTime = System.nanoTime() - ioStartTime;
                        ranTasks = runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
                    }
                } else {
                    ranTasks = runAllTasks(0); // This will run the minimum number of tasks
                }
```
NioEventLoop 의 run() 메서드는 무한 루프를 돌면서 IO Event 와 Task Queue 의 테스크들을 처리한다.

runAllTasks() 의 Break Point 를 잡은 후에 MainThread 에서 WebClient 를 호출하면 addTask() 이후에 NioEventLoop 의 runAllTasks() 에 잡히는 것을 확인할 수 있다.


이때 Task 를 처리하는 것은 MainThread 가 아니라 NioEventLoop 쓰레드이다.

```java
    protected static void safeExecute(Runnable task) {
        try {
            task.run();
        } catch (Throwable t) {
            logger.warn("A task raised an exception. Task: {}", task, t);
        }
    }
```
safeExecute() 함수에 의해서 Task 의 run() 메서드가 실행된다.

 

PooledConnectionProvider.java

```java
				ChannelOperations<?, ?> ops = opsFactory.create(pooledConnection, pooledConnection, null);
				if (ops != null) {
					ops.bind();
					// First send a notification that the connection is ready and then change the state
					// In case a cancellation was received, ChannelOperations will be disposed
					// and there will be no subscription to the I/O handler at all.
					// https://github.com/reactor/reactor-netty/issues/1165
					sink.success(ops);
					obs.onStateChange(ops, State.CONFIGURED);
				}
```
run() 메서드에서 obs.onStateChange() 함수가 호출되면 실제 ByteBuf 에 Write 하기 위한 코드들이 실행된다.

Observer 를 통해서 상태 변경을 감지한 HttpIOHandlerObserver.java 가 상태 변경을 전달 받아 reactor component 들에게 전달한다.


 

HttpClientOperation.java 가 mapper 로 사용되어 HttpOpertions.send() 가 호출되게 된다.
```java
	@Override
	@SuppressWarnings("unchecked")
	public NettyOutbound send(Publisher<? extends ByteBuf> source) {
		if (!channel().isActive()) {
			return then(Mono.error(AbortedException.beforeSend()));
		}
		if (source instanceof Mono) {
			return new PostHeadersNettyOutbound(((Mono<ByteBuf>)source)
					.flatMap(msg -> {
						if (markSentHeaderAndBody(msg)) {
							try {
								afterMarkSentHeaders();
							}
							catch (RuntimeException e) {
								ReferenceCountUtil.release(msg);
								return Mono.error(e);
							}
							if (HttpUtil.getContentLength(outboundHttpMessage(), -1) == 0) {
								log.debug(format(channel(), "Dropped HTTP content, " +
										"since response has Content-Length: 0 {}"), toPrettyHexDump(msg));
								msg.release();
								return FutureMono.from(channel().writeAndFlush(newFullBodyMessage(Unpooled.EMPTY_BUFFER)));
							}
							return FutureMono.from(channel().writeAndFlush(newFullBodyMessage(msg)));
						}
						return FutureMono.from(channel().writeAndFlush(msg));
					})
					.doOnDiscard(ByteBuf.class, ByteBuf::release), this, null);
		}
		return super.send(source);
	}
```
channel().writeAndFlust() 함수에 의해서 해당 Channel 에 write 가 실행되게 된다.

 

HttpClientOperation 의 send 함수 호출 부분은 EncoderHttpMessageWriter.java 에서 MessageWriter 가 만들어질때 이미 결정되어있고 subscribe() 가 호출되고 onNext() 가 호출되었을때 그제서야 실제 콜백인 send() 가 호출되는 것이다.

 

NioSocketChannel.java
```java
   @Override
    protected void doWrite(ChannelOutboundBuffer in) throws Exception {
        SocketChannel ch = javaChannel();
        int writeSpinCount = config().getWriteSpinCount();
        do {
          ...
            switch (nioBufferCnt) {
             	...
                case 1: {
                    // Only one ByteBuf so use non-gathering write
                    // Zero length buffers are not added to nioBuffers by ChannelOutboundBuffer, so there is no need
                    // to check if the total size of all the buffers is non-zero.
                    ByteBuffer buffer = nioBuffers[0];
                    int attemptedBytes = buffer.remaining();
                    final int localWrittenBytes = ch.write(buffer);
                   ...
    }
```
ch.write() 가 호출되어 실제 write 가 실행되며 이 이후에 해당 요청을 받는 서버쪽에서 요청을 수신하게 된다.

 

SocketChannelImpl.java

var3 = IOUtil.write(this.fd, var1, -1L, nd);
native 메서드를 호출하여 write.

 

만약 WebClient 를 호출한 쓰레드가 MainThread 가 아니라 NioEventLoop 쓰레드라면?

 
```java
    @Override
    public ChannelHandlerContext flush() {
        final AbstractChannelHandlerContext next = findContextOutbound(MASK_FLUSH);
        EventExecutor executor = next.executor();
        if (executor.inEventLoop()) {
            next.invokeFlush();
        } else {
            Tasks tasks = next.invokeTasks;
            if (tasks == null) {
                next.invokeTasks = tasks = new Tasks(next);
            }
            safeExecute(executor, tasks.invokeFlushTask, channel().voidPromise(), null, false);
        }

        return this;
    }
```
AbstractChannelHandlerContext.java 에서 현재 Thread 가 EventLoop 쓰레드 이므로 TaskQueue 에 넣지 않고 즉시 실행한다.

### 12/4 Flyway
Flyway는 오픈소스 데이터베이스 마이그레이션 툴 입니다. 데이터베이스 마이그레이션 툴이란 데이터베이스의 변경 사항을 추적하고, 업데이트나 롤백을 보다 쉽게 할 수 있도록 도와주는 도구

주의할 점은, 파일 이름을 지을 때 규칙에 맞게 지어야 한다는 거예요.



도입은 간단하다. 아래의 내용을 build.gradle에 넣고 재빌드하면 됨.

 

    implementation 'org.flywaydb:flyway-core'
    implementation 'org.flywaydb:flyway-mysql'
 

💋 V1__init.sql: 최초의 데이터베이스 스키마 등록 
해당 파일을 main/resources/db/migration 폴더 아래에 생성한다. 위치가 중요함!
최초의 데이터베이스 스키마를 등록한다.
데이터베이스 스키마 변경 이전의 스키마를 등록해주면 된다.
우리팀의 최초 데이터베이스 스키마는 테이블 create문과 foreign key 설정에 대한 명령어였다.
전문은 매우매우 길어서 접는 글에 넣어놓았고, 아래 코드는 그중 일부다!

### 12/5 Flyway sql 파일명 설정
기본 형태는 V{버전}__{설명}.sql 예요.
버전과 설명 사이에 언더바는 2개가 들어가야 해요.
새로 추가되는 sql 파일의 버전은 이미 존재하는 버전의 숫자보다 커야해요.
- ex) V1 -> V2 -> V3 , V1.1.0 -> V1.1.1 -> V1.1.2


baseline-on-migrate 설정이란?
기본적으로 flyway는 마이그레이션 스크립트를 순차적으로 적용하기 때문에, 마이그레이션 스크립트에 작성한 내용이 이미 데이터베이스에 반영되어 있다면 에러가 발생한다.
기준 버전 이전의 스크립트를 무시하고 새로운 스크립트만 적용하고자 할 때 baseline-on-migrate 설정을 사용할 수 있다.
만약 마이그레이션 스크립트가 V2까지 이미 적용되어 있다면, baseline-on-migrate 설정을 사용하지 않으면 flyway가 새로운 마이그레이션 스크립트를 적용하려고 할 때 에러가 발생할 수 있다.
만약, 아직 마이그레이션 스크립트가 V1부터 적용된 적이 없는 새로운 데이터베이스의 경우, baseline-on-migrate 설정을 사용하든 사용하지 않든 결과는 동일하다. 이 경우에는 모든 마이그레이션 스크립트가 순서대로 적용된다.
