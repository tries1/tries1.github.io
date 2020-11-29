---
layout: post
title:  "Spring 비동기 처리의 이해 2(Reactive Streams Interface 구현해보기)"
date:   2020-11-29 12:00:00
author: Glenn
categories: Spring
cover:  "/assets/instacode.png"
comments: true
---

이전 포스트에서 비동기 처리가 필요한 이유와 개념을 이해하였으니  
 [(Spring 비동기 처리의 이해)](https://tries1.github.io/spring/2020/09/21/spring_async.html)  

Spring Webflux를 사용하여 프로그램을 만들기 앞서,
Reactor를 좀더 이애하기위해 Reactive Streams Interface 직접 구현해보겠습니다.

## Reactive Streams Interface란 무엇인가?
[reactive menifesto](https://www.reactivemanifesto.org/ko)

리액티브 시스템이란?  
![reactive-menifesto](https://github.com/tries1/glenn-blog/blob/master/assets/spring/reactive-menifesto.png?raw=true)

응답이 잘 되고, 탄력적이며 유연하고 메시지 기반으로 동작하는 시스템 입니다. 우리는 이것을 리액티브 시스템(Reactive Systems)라고 부릅니다.  

- 응답성(Responsive)
  - 신속하고 일관성 있는 응답 시간을 제공하고, 신뢰할 수 있는 상한선을 설정하여 일관된 서비스 품질을 제공합니다. 
- 탄력성(Resilient)
  - 시스템이 장애 에 직면하더라도 응답성을 유지 하는 것을 탄력성이 있다고 합니다. 
- 유연성(Elastic)
  - 시스템이 작업량이 변화하더라도 응답성을 유지하는 것을 유연성이라고 합니다.
- 메시지 구동(Message Driven)
  - 명시적인 메시지 전달은 시스템에 메시지 큐를 생성하고, 모니터링하며 필요시 배압 을 적용함으로써 유연성을 부여하고, 부하 관리와 흐름제어를 가능하게 합니다

## Reactive Streams Interface 구현해보기
- Reactive streams interface
  - [https://www.reactive-streams.org/](https://www.reactive-streams.org/)
  - [https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/package-summary.html](https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/package-summary.html)
  - [https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md#specification](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md#specification)


Webflux에서는 Reactor을 사용하며  
Reactor는 Reactive Streams Interface의 구현체중 하나입니다.

Reactive Streams Interface 조금이나마 이해하기 위해  
직접 구현해보도록 하겠습니다.

구현해볼 인터페이스는 3개입니다.

- `Publisher<T>`  
  - 데이터를 제공하는 역할

```java
import org.reactivestreams.Publisher;

Publisher<Integer> pub = new Publisher<Integer>() {
    @Override
    public void subscribe(Subscriber<? super Integer> s) {}
};
```

- `Subscriber<T>`
  - 데이터를 소모하는 역할
  
```java
import org.reactivestreams.Subscriber;

Subscriber<Integer> sub = new Subscriber<Integer>() {
    @Override
    public void onSubscribe(Subscription s) {}

    @Override
    public void onNext(Object o) {}

    @Override
    public void onError(Throwable t) {}

    @Override
    public void onComplete() {}
};
```

- `Subscription`
  - Publisher에게 필요한 데이터 수를 요청하고, cancel요청을 하며, BackPressure 동작에 중요한 역할
  
```java
import org.reactivestreams.Subscription;

Subscription subscription = new Subscription() {
    @Override
    public void request(long n) {}

    @Override
    public void cancel() {}
};
```

위의 인터페이스는 서로 아래와 같은 흐름을 가지고 동작합니다.
![reactive stream interface 동작](https://github.com/tries1/glenn-blog/blob/master/assets/spring/reactive-stream-interface-behavior.png?raw=true)
출처 : [https://grokonez.com/java/java-9-flow-api-reactive-streams](https://grokonez.com/java/java-9-flow-api-reactive-streams)

먼저, 데이터를 제공하는 Publisher를 작성해보겠습니다.
```java
Publisher<Integer> pub = new Publisher<Integer>() {
    // 1. 1~10까지의 데이터를 제공
    List<Integer> integers = IntStream.rangeClosed(1, 100).boxed().collect(Collectors.toList());

    @Override
    public void subscribe(Subscriber<? super Integer> s) {
        // 2. 나머지는 Subscription에서 구현
        s.onSubscribe();
    }
};
```

그다음, 데이터를 소모하는 Subscriber를 작성해보겠습니다.
```java
Subscriber<Integer> sub = new Subscriber<Integer>() {
    @Override
    public void onSubscribe(Subscription s) {
        // 1. 받을수 있는 모든 데이터를 요청
        s.request(Long.MAX_VALUE);
    }

    @Override
    public void onNext(Integer integer) {
        System.out.println("onNext : " + integer);
    }

    @Override
    public void onError(Throwable t) {
        System.out.println("onError : " + t.getMessage());
    }

    @Override
    public void onComplete() {
        System.out.println("onComplete");
    }
};
```

마지막으로 Subscription 구현해보겠습니다.  
Publisher의 onSubscribe를 다음과 같이 변경합니다.
```java
@Override
public void subscribe(Subscriber<? super Integer> s) {
    s.onSubscribe(new Subscription() {
        @Override
        public void request(long n) {
            try {
                integers
                .stream()
                .limit(n)
                .forEach(i -> {
                    if (i > 11) {
                        throw new IllegalArgumentException("integer must be under 11");
                    }
                    s.onNext(i);
                });
            } catch (Exception e) {
                s.onError(e);
            }

            s.onComplete();
        }

        @Override
        public void cancel() {
            System.out.println("cancel");
        }
    });
}
```

이제 Publisher를 구독해보겠습니다.
```java
pub.subscribe(sub);
```

아래와 같은 결과를 볼수있습니다.
```java
onNext : 1
onNext : 2
.
.
.
onNext : 10
```

onError를 발생시키기 위해 Subscriber의 데이터 요청수를 20으로 변경해봅니다.

```java
@Override
public void onSubscribe(Subscription s) {
    s.request(20);
}
```

11이상의 데이터를 요청하면 onError를 호출하는것을 볼수있습니다.  
다음 포스트에선 Webflux R2DBC를 이용한 간단한 Rest API를 작성해보겠습니다.
