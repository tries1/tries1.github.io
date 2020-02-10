---
layout: post
title:  "Spring Webflux를 활용하여 Bookstore 만들기"
date:   2020-02-10 12:00:00
author: Glenn
categories: Spring
cover:  "/assets/instacode.png"
comments: true
---

### Spring initializr 사용하기 
[Spring initializr](https://start.spring.io/)를 사용하면 간단하게 스프링 프로젝트를 생성할수 있습니다.  

아래 이미지와 같이 옵션을 선택합니다.  
![Book store1](https://github.com/tries1/glenn-blog/blob/master/assets/spring/webflux2_1.png?raw=true)

![Book store2](https://github.com/tries1/glenn-blog/blob/master/assets/spring/webflux2_2.png?raw=true)


---

### API 만들기
1. edu.webflux.bookstore 패키지 아래에 controller 패키지를 추가합니다.
![Book store3](https://github.com/tries1/glenn-blog/blob/master/assets/spring/webflux2_3.png?raw=true)

2. HomeController.java를 추가하고 controller Method를 추가합니다.

```java
package edu.webflux.bookstore.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Mono;

@RestController
public class HomeController {

    @GetMapping("/")
    public Mono<String> home(){
        return Mono.just("welcome book store");
    }
}
```

3. 프로젝트 Root에서 `./gradlew bootRun` 입력하여 어플리케이션을 실행합니다.

4. 브라우저에서 `http://localhost:8080` 입력하여 welcome book store가 나오는지 확인합니다.
