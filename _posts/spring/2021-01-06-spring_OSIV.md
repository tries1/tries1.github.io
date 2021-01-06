---
layout: post
title:  "Spring Data JPA의 Open Session In View (OSIV)"
date:   2021-01-06 12:00:00
author: Glenn
categories: Spring
cover:  "/assets/instacode.png"
comments: true
---

## 준영속 상태에서 FetchType.Lazy접근 문제

기본적으로 영속성 컨텍스트 전략은 트랜잭션 범위입니다.  
Spring에서 Transaction의 시작은 보통 Service로 설정되고 사용되어집니다.  
문제는 트랜잭션 범위를 벗어난 Controller, View의 경우 준영속상태에서 페치전략이 FetchType.Lazy인  
엔티티에 접근시 `LazyInitializationException`이 발생하게 됩니다.

```java
memberService.findAll().stream()
.forEach(member -> {
    // Proxy 객체를 참조하고있어서 에러 발생하지 않는다.
    member.getOrders();

    // LazyInitializationException 발생!!
    member.getOrders().get(0);
});
```

## FetchType.Lazy와 Proxy 객체

연관된 객체의 모든 데이터를 가져오는것은 비효율적이기때문에  
`@OneToMany`, `@ManyToMany`같은 연관관계 매핑 어노테이션 사용시 `FetchType.Lazy`를 사용합니다.(기본 FetchType은 Lazy)  
`FetchType.Lazy`은 실제 사용하기전까지 데이터로 로딩하지 않는 기능인데요.  
그전까지 실제 클래스와 겉모습이 비슷하고 실제 객체를 참조하고있는 Proxy 객체를 바라보고있습니다.

위의 예제를 다시보면 `member.getOrders()`는 Proxy 객체를 참조하고있어 에러가 발생하지 않습니다.  
(실제 데이터를 가져오지 않는다는 뜻입니다.)


## Controller, View와 준영속 상태

보통 Controller, View의 경우 트랜잭션이 적용되지 않습니다.  
그렇기에 영속성 컨텍스트에서 분리된 준영속상태입니다.  
이런 이유로 Controller, View에서 `FetchType.Lazy` 속성의 엔티티에 접근하지 못하게 됩니다.


## Open Session In View

이런 문제를 해결하기 위한 방법이 OSIV인데, Spring에서의 OSIV를 살펴보겠습니다.  
SpringBoot의 OSIV 기본설정은 true입니다.  

```java
// application.properties
spring.jpa.open-in-view: true
```

그래서 SpringBoot를 사용했다면 `LazyInitializationException`이 발생하는것을 경험하지 못했을수도 있습니다.  
SpringBoot는 다음과 같이 영속성 컨텍스트를 관리합니다.  

1. 요청이 들어오면 영속성 컨텍스트 시작
2. service, repository계층의 트랜잭션이 종료될때 flush(영속성 적용)를 호출
3. controller, view 계층은 엔티티를 수정해도 이미 flush가 진행되었기 때문에 더티체킹(변경감지)이 되지 않음

위처럼 동작하는데, 한가지 문제는 controller에서 엔티티 변경후 service로직을 호출하면 flush가 적용되어  
의도치않게 값이 변경될수 있습니다.  
이런경우는 엔티티를 수정하지않고, DTO를 사용해서 service에 넘겨주면 문제를 피할수 있습니다.

