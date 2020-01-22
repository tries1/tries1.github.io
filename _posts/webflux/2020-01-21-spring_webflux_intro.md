---
layout: post
title:  "Spring Webflux 소개"
date:   2019-07-31 11:00:00
author: Glenn
categories: Spring
cover:  "/assets/instacode.png"
comments: true
---

## Spring Webflux
Spring Web에서 비동기 & 논블록킹 Reactive programming을 활용한 웹개발 

### Reactive Stack VS Servlet Stack

![Servlet Stack, Reative Stack 비교 이미지](https://github.com/tries1/glenn-blog/blob/master/assets/spring/servlet_vs_reactive.png?raw=true)

1. Springboot Webflux (Reative Stack)를 보면 기본 내장 컨테이너가 Tomcat -> Netty를 사용합니다.
2. Reactive programming을 지원하기위해 Reactor 포함
3. RestTemplate -> WebClient로 변경하여 비동기 Call이 가능(Servlet stack에서 해당 기능만 쓰기도함)
4. Reactive Repositories를 지원
 - Reactive 방식을 지원하는 Mongo, Redis, Cassandra등을 사용하여 Repository에 논블록킹 연결이 가능
 - 기존 JDBC 논블록킹 방식을 지원하지않기때문에 [R2DBC](https://spring.io/projects/spring-data-r2dbc)를 사용해야함

---


### Servlet Stack(Spring MVC)의 문제

![Thread Pool Hell 비교 이미지](https://github.com/tries1/glenn-blog/blob/master/assets/spring/thread_pool_hell.png?raw=true)

Thread Pool Hell을 검색하면 쉽게 볼수있는 이미지입니다.  

기존의 방식은 사용자 Request하나당 서버 하나의 Thread를 사용하는 방식으로  
Thread Pool을 사용해 Thread를 재사용하는 방식이였습니다.  

이 방식의 문제는 짧은시간에 요청이 많아졌을때 재사용가능한 Thread가 부족하게되고  
부족한 Thread를 Client의 대기시간이 늘어나게되고, 서버의 자원이 부족해지며, 결국엔 응답실패로 이어지는 문제입니다.  

*결국 서버는 대기만하다가 처리량이 떨어지면서 제대로된 기능을 하지못하게됩니다.*

---

### Spring Webflux의 해결방법은?
비동기 논블록킹, 콜백헬의 문제를 해결할수있도록 Reactive programming 방식의 개발을 지원  
Reactive streams interface를 구현한 [Reactor](https://projectreactor.io/)를 사용하여 문제를 해결 

RestTemplate -> WebClient를 이용하여 Rest API 비동기 Call지원 

Sprongboot2.X의 경우 webflux를 이용하였을때 내장 컨테이너가 Netty를 사용함(비동기 방식에 Tomcat보다 적합)

Netty의 경우 Default Worker Thread의 갯수는 CPU Core * 2 입니다.  
(*로직중 블록킹되는 부분이 있으면 안되는 이유*)

현재 많은 서비스들이 [MSA](https://ko.wikipedia.org/wiki/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4)를 적용하면서  
API 호출이 많아짐에따라 효율적이고 좋은성능을 낼수있는 Webflux를 적극적으로 도입하고있는 추세입니다.  
