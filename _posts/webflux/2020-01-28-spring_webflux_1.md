---
layout: post
title:  "Spring Webflux 1(Reactor 살펴보기)"
date:   2020-01-28 12:00:00
author: Glenn
categories: Spring
cover:  "/assets/instacode.png"
comments: true
---

## [Reactor](https://projectreactor.io/)
Spring Webflux에서 비동기, 논블록킹 어플리케이션 프로그래밍을 위해 지원되는 라이브러리이며  
Spring과 함께 **Pivotal**에서 관리되고있으며, [Reactive streams interface](https://www.reactive-streams.org/)를 구현하였습니다.

*Reactor를 처음 시작하면 다양한 생성자와, Operators 때문에 익숙해지는데 시간이 걸리게됩니다.*  
*그러나, Reactive Programming에 점차 익숙해지고 좀더 활용하게되면*  
*비동기 논블록킹 프로그래밍을 간결하게 처리할수 있게됩니다.*

---

### 2가지 [Publisher](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/package-summary.html) 
#### [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)
**0 - 1**개의 데이터를 다룰때 사용

#### [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)
**0 - N**개의 데이터를 다룰때 사용

---

### Publisher 생성하기
#### create (Cold Publisher)

create(Consumer<MonoSink<T>> callback) lower level의 메소드로  
직접적으로 데이터의 방출 및 에러신호 내보낼수있습니다.

*Cold Publisher이며 subscribe(구독) 하지않으면 데이터를 방출하지 않습니다.*  
*[Hot, Cold Publisher는 추후에 정리하도록 하겠습니다.](https://projectreactor.io/docs/core/release/reference/#reactor.hotCold)*

Mono
```java
Mono.create(monoSink -> {
  try {
      monoSink.success(1);
  } catch (RuntimeException e) {
      monoSink.error(e);
  }
}).subscribe();

```

Flux
```java
Flux.create(fluxSink -> {
  try {
      fluxSink.next(1);
      fluxSink.next(2);
      fluxSink.next(3);
  } catch (RuntimeException e) {
      fluxSink.error(e);
  }
}).subscribe();
```

---

#### Just (Hot Publisher)
just(T)는 가장 일반적이며 해당값을 즉시 방출합니다.  
Hot Publisher이며 처음 방출된 값을 cache해놓고 다음 구독자에게 cache된 값을 방출합니다.  

*Hot Publisher이며 subscribe(구독) 하지않아도 데이터를 방출합니다.*  

Mono(T)
```java
Mono<Double> monoJust = Mono.just(Math.random());
Mono<Double> monoDefer = Mono.defer(() -> Mono.just(Math.random()));

// just는 처음에 방출한 값을 내부적으로 cache한후 재사용 
// random값을 호출했지만 모두 같은결과가 나옵니다.
System.out.println("Just >>>>>>>>>>..");
monoJust.subscribe(System.out::println);
monoJust.subscribe(System.out::println);
monoJust.subscribe(System.out::println);

// defer는 매번 새로운 값을 방출 
// defer를 사용하여 Hot Publisher를 Cold Publisher로 변경하는 방법이기도 합니다.
System.out.println("Defer >>>>>>>>>>..");
monoDefer.subscribe(System.out::println);
monoDefer.subscribe(System.out::println);
monoDefer.subscribe(System.out::println);
```

Flux(T... data)  

Hot, Cold Publisher의 성격은 동일하며,  
아래는 Flux의 just의 간단한 사용법입니다.
```java
// 1, 2, 3, 4, 5를 순서대로 방출
Flux
.just(1, 2, 3, 4, 5)
.subscribe(System.out::println);
```

---

#### defer (Cold Publisher)
defer라는 단어에서 알수있듯이 데이터의 방출을 구독전까지 *지연*시킵니다.  
뿐만 아니라 checked exception method를 호출하여 처리할수있습니다.  

*Cold Publisher이며 subscribe(구독) 하지않으면 데이터를 방출하지 않습니다.*
  
```java
// just는 exception처리를 할수없음
// compile error : unhandle exception
Mono.just(someError());

Mono.defer(() -> {
  try {
      Integer res = someError();
      return Mono.just(res);
  } catch (Exception e) {
      // 에러가 발생하면 error 신호를 방출
      return Mono.error(e);
  }
});


public Integer someError() throws Exception {
  throw new RuntimeException("에러 발생!!");
}
```

---

#### fromCallable
fromCallable(Callable<? extends T> supplier)는 defer와 유사하지만  
exception을 자동으로 Mono.error로 래핑합니다.
  
*Cold Publisher이며 subscribe(구독) 하지않으면 데이터를 방출하지 않습니다.*  

```java
//checked exception method를 호출할수있다.
Mono.fromCallable(someError()).subscribe();

public Integer someError() throws Exception {
  throw new RuntimeException("에러 발생!!");
}
```

---

**기본적인 Mono와 Flux의 생성자를 소개해드렸으며, 이외에도 많은 생성자가 있습니다.**  
