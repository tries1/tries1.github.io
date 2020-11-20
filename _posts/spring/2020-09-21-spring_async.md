---
layout: post
title:  "Spring 비동기 처리의 이해"
date:   2020-09-21 12:00:00
author: Glenn
categories: Spring
cover:  "/assets/instacode.png"
comments: true
---

#### 해당 내용은 [Toby님의 Yutube의 자바와 스프링 비동기 기술](https://www.youtube.com/channel/UCcqH2RV1-9ebRBhmN_uaSNg) 내용을 보고 개인적으로 정리한 내용입니다.

## Web Server Architecture 변화

![Monolithic Architecture 이미지](https://github.com/tries1/glenn-blog/blob/master/assets/spring/monolithic_architecture.png?raw=true)

과거 많은 서비스들이 위와같이 하나의 DB, 하나의 어플리케이션에서 데이터를 처리하였습니다.  
하지만 지속적인 배포 및 어느 한부분의 장애가 발생하였을때의 서비스중단등 의 문제를 가지고있습니다.


![Micro Server Architecture 이미지](https://github.com/tries1/glenn-blog/blob/master/assets/spring/web_architecture.png?raw=true)

최근 많은 서비스들이 위의 이미지와 같이 수많은 Front, Back-end Server가 서로 호출하는 구조를 가지고 있으며   
각 기능들이 분리되어있어, 작은 단위로 지속적인 배포가 가능하고, 필요에따라 Scale-out등이 유연합니다.  


![Thread Pool Hell 비교 이미지](https://github.com/tries1/glenn-blog/blob/master/assets/spring/thread_pool_hell.png?raw=true)

하지만 각 기능들이 물리적으로 분리되어 데이터를 가져오거나,  
전달이 필요할때 네트워크를 통해서 가져오게 되면서 API 호출이 증가하게 되었고,  
위의 이미지와 같이 많은 Thread 사용 및 Thread Block이 발생하며 서버성능이 떨어지게 되었습니다.

## Thread Blocking의 문제
Client -> Servlet Thread -> request -> logic(Block IO - DB, API) -> response  

위와같은 상황일때 Block IO에서 DB나 API의 결과를 기다리기위해 Thread는 wait하게되고,  
그 상황에서 새로운 Client요청이 들어오면 Servlet Thread가 새로 생성되는데,  
Thread가 새로 생성되면 서버의 Memory는 증가하고, CPU는 Blocking된 Thread의 상태를 저장하고,  
Context Switching이 된다.(최소 2번의 Context Switching이 발생)  
Context Switching는 CPU소모가 큰작업

## Thread Blocking의 문제를 해결하는방법
![비동기 처리 이미지](https://github.com/tries1/glenn-blog/blob/master/assets/spring/async_servlet_structure.png?raw=true)

Servlet Thread가 요청을 받은후 Worker Thread로 작업을 위임하고 다른 요청을 받는다.

*Servlet Thread : 웹요청을 처리하기위한 Thread*  
*Worker Thread : 별도의 작업을 처리하기위한 Thread*

## Spring, Java에서 비동기 처리방법

`ListenableFuture`
 - `ListenableFuture`은 Spring 4.0부터 사용 가능하며 처리 결과를 받을수있는 `CallBack` 메소드를 제공 
 - Spring 4.1부터 `@Async`가 붙은 메소드의 결과를 `AsyncResult`이용하여 받을수 있다.

```java
@Configuration
@EnableAsync
public class AsyncConfiguration{...}

@Async
public ListenableFuture<String> async() {
    return new AsyncResult<>("hello");
}
```
`ThreadPoolTaskExecutor`
 - `@Async`가 붙은 메소드를 호출하면 `SimpleAsyncTaskExecutor`를 사용하는데, 호출마다 Thread 생성하여 비효율적이다.
 - ThreadPoolTaskExecutor Bean을 만들어 등록하면 Thread가 관리되며 효율적임.  
   
```java
@Bean(name = "myAsyncThreadPoolTaskExecutor")
public Executor myAsyncThreadPoolTaskExecutor() {
    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
    taskExecutor.setCorePoolSize(THREAD_POOL_SIZE);
    taskExecutor.setMaxPoolSize(THREAD_POOL_SIZE);
    taskExecutor.setQueueCapacity(Integer.MAX_VALUE);
    taskExecutor.setThreadNamePrefix("myAsyncThreadPoolTaskExecutor-");
    taskExecutor.initialize();
    return taskExecutor;
}
```

`AsyncRestTemplate`
 - `RestTemplate`은 동기방식으로 `AsyncRestTemplate` 대체하여 사용  
 - 그냥 호출하면 Worker Thread가 많이 생성되어 Netty를 사용하여 비동기 Thread처리가 가능
  - AsyncRestTemplate art = new AsyncRestTemplate(new Netty4ClientHttpRequestFactory(new NioEventLoopGroup(1))); 
 
```java
AsyncRestTemplate art = new AsyncRestTemplate();

@GetMapping("sample1")
public ListenableFuture<ResponseEntity<String>> sample1(String req) {
    return art.getForEntity("{API URL}", String.class, req);
}
```

`DefferedResult`
 - Spring 3.2사용 가능, 비동기 요청 처리하기위해 사용

```java
// 중첩된 AsyncRestTemplate의 결과를 DeferredResult로 처
@GetMapping("sample2")
public DeferredResult<String> sample2() {
    AsyncRestTemplate art = new AsyncRestTemplate();

    DeferredResult<String> dr = new DeferredResult<>();

    ListenableFuture<ResponseEntity<String>> lf1 = art.getForEntity("http://localhost:8081/some-api1", String.class);
    lf1.addCallback(res1 -> {
        ListenableFuture<ResponseEntity<String>> lf2 = art.getForEntity("http://localhost:8081/some-api1", String.class);
        lf2.addCallback(res2 -> {
            ListenableFuture<ResponseEntity<String>> lf3 = art.getForEntity("http://localhost:8081/some-api1", String.class);
            lf3.addCallback(res3 -> {
                dr.setResult(String.format("%s, %s, %s", res1.getBody(), res2.getBody(), res3.getBody()));
            }, e -> log.error(e.getMessage()));
        }, e -> log.error(e.getMessage()));
    }, e -> log.error(e.getMessage()));

    return dr;
}
```

`CompletableFuture`
 - 위의 결과는 Callback 중첩으로 인해 가독성이 떨어진다.
 - Java8에 추가된 `CompletableFuture`을 이용하여 Chanining하여 작성할수 있다.

```java
@GetMapping("sample2")
public DeferredResult<String> sample2() {
    DeferredResult<String> dr = new DeferredResult<>();

    //CF 는 비동기의 상태를 가지고있는
    toCF(art.getForEntity("http://localhost:8081/some-api1", String.class))
            .thenCompose(it -> toCF(art.getForEntity("http://localhost:8081/some-api1", String.class)))
            .thenCompose(it -> toCF(art.getForEntity("http://localhost:8081/some-api1", String.class)))
            //.thenCompose(it -> toCF(myService.async()))
            .thenAccept(it -> dr.setResult(String.format("%s", it)))
            .exceptionally(e -> {
                dr.setErrorResult(e.getMessage());
                return (Void) null;
            });

    return dr;
}

<T> CompletableFuture<T> toCF(ListenableFuture<T> lf) {
    CompletableFuture<T> cf = new CompletableFuture<>();
    lf.addCallback(it -> cf.complete(it), e -> cf.completeExceptionally(e));
    return cf;
}
```

## 검증 및 테스트

우선 테스트 요청을 받아줄 웹서버를 8081포트로 띄우고 아래와 같이 API를 추가한다.
```java
@Slf4j
@RequiredArgsConstructor
@RestController
public class TestController {
    DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
 
    // 응답이 느린 API
    @GetMapping("some-api")
    public String someBlockAPI() throws InterruptedException {
        Thread.sleep(2000);
        log.info("someBlockAPI call");
        return String.format("some-block-api call, timestamp : %s", LocalDateTime.now().format(dtf));
    }
}
```

테스트 요청을 수행할 Load Test Class를 생성한다.
```java
import org.springframework.util.StopWatch;
import org.springframework.web.client.RestTemplate;
 
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
 
import lombok.extern.slf4j.Slf4j;
 
 
@Slf4j
public class LoadTest {
    public static void main(String[] args) throws Exception {
        final Integer RUNNER = 100;
        AtomicInteger ai = new AtomicInteger(1);
        AtomicInteger successCount = new AtomicInteger(0);
        RestTemplate rt = new RestTemplate();
        ExecutorService es = Executors.newFixedThreadPool(RUNNER);
        CyclicBarrier cb = new CyclicBarrier(RUNNER + 1);
        for (int i = 1; i <= RUNNER; i++) {
            es.execute(() -> {
                try {
                    int idx = ai.getAndIncrement();
                    cb.await();
 
                    StopWatch sw = new StopWatch();
                    sw.start();
                    rt.getForObject("http://localhost:8080/block-api", String.class);
                    successCount.incrementAndGet();
                    sw.stop();
                    log.info("Elapsed: idx : {}, Sec : {}", idx, sw.getTotalTimeSeconds());
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
        }
        cb.await();
        StopWatch main = new StopWatch();
        main.start();
 
        es.shutdown();
        es.awaitTermination(100, TimeUnit.SECONDS);
 
        main.stop();
        log.info("Running Sec : {}, Success Count : {}", main.getTotalTimeSeconds(), successCount);
    }
}
``` 

### Blocking API를 동시에 호출하였을때 Thread의 변화를 확인
```java
@RestController
public class BlockingApiController {
    RestTemplate rt = new RestTemplate();
 
    @GetMapping("block-api")
    public String block() {
       return rt.getForObject("http://localhost:8081/some-api", String.class);
    }
}
```

Thread의 생성상태를 모니터링하기위한 Java Mission Control을 실행
```bash
> jmc
```

![JMC2](https://github.com/tries1/glenn-blog/blob/master/assets/spring/jmc-sample2.png?raw=true)


LoadTest를 수행하여 Thread 변화확인

![LoadTest1](https://github.com/tries1/glenn-blog/blob/master/assets/spring/load-test-result-1.png?raw=true)

Thread가 100개가 생성되어 모든 요청을 2초대로 처리하는것을 확인할수있다

### Tomcat Thread를 1로 제한하여 호출

Thread가 계속 생성되는건 문제가있으니 Thread를 제한을 걸고 다시 테스트 결과를 확인

application.properties에서 아래의 내용을 추가
```text
# Default 200
server.tomcat.threads.max=1
```

![LoadTest2](https://github.com/tries1/glenn-blog/blob/master/assets/spring/load-test-result-2.png?raw=true)

Thread가 과도하게 생성되진않지만, 100개의 요청이 하나씩 처리되어 처리시간도 늘어난걸 확인할수 있다.

### AsyncRestTemplate으로 변경

Blocking API대신 AsyncRestTemplate을 사용한 API를 추가하고 결과를 확인

```java
Spring 4부터 ListenableFuture지원하며 Controller에서 ListenableFuture리턴하면 Spring이 알아서 처리해준다.
AsyncRestTemplate art = new AsyncRestTemplate();
 
@GetMapping("async-api")
public ListenableFuture<ResponseEntity<String>> async() {
    return art.getForEntity("http://localhost:8081/some-api", String.class);
}
```

그러나 Thread가 순간적으로 생성되는것을 확인할수 있다.

![LoadTest3](https://github.com/tries1/glenn-blog/blob/master/assets/spring/load-test-result-3.png?raw=true)

### AsyncRestTemplate RequestFactory를 Netty로 변경

AsyncRestTemplate와 Netty를 사용하여 Thread의 변화 확인  
build.gradle에 netty라이브러리 추가

```text
implementation 'io.netty:netty-all:4.1.52.Final'
```

AsyncRestTemplate RequestFactory를 Netty로 변경
```java
AsyncRestTemplate art = new AsyncRestTemplate(new Netty4ClientHttpRequestFactory(new NioEventLoopGroup(1)));
```

LoadTest를 다시수행하여 결과를 확인하면 1개의 Thread로 100개의 요청을 동시에 처리하는것을 볼수있다.

