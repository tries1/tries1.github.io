---
layout: post
title:  "Spring 비동기 처리의 이해 4(Webflux with R2DBC, 게시판 만들기)"
date:   2020-12-10 12:00:00
author: Glenn
categories: Spring
cover:  "/assets/instacode.png"
comments: true
---

이전 포스트에서 H2 DB를 설정하고 dummy 데이터 초기화까지 해보았습니다. 
 [(Spring 비동기 처리의 이해 3)](https://tries1.github.io/spring/2020/12/02/spring_async3.html)  

이제 간단한 User API를 만들어보겠습니다.

## Package 추가

다음과같이 controller, service, repository, model 4개의 package를 추가합니다.

![webflux4-1](https://github.com/tries1/glenn-blog/blob/master/assets/spring/webflux4_1.webp?raw=true)

## User Model 추가 

model/User.java를 생성하고 다음과 같이 작성합니다.

```java
package com.example.webfluxboard.model;

import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.relational.core.mapping.Table;

import java.time.LocalDateTime;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
@AllArgsConstructor
@Table("users")
public class User {

    @Id
    private Long id;
    private String name;
    private Integer age;
    private String profilePictureUrl;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    @CreatedDate
    private LocalDateTime createdAt;

    @Builder
    public User(String name, Integer age, String profilePictureUrl, LocalDateTime updatedAt, LocalDateTime createdAt) {
        this.name = name;
        this.age = age;
        this.profilePictureUrl = profilePictureUrl;
        this.updatedAt = updatedAt;
        this.createdAt = createdAt;
    }
}

```

## @EnableR2dbcAuditing 추가 

User Model에 추가한 `@CreatedDate`, `@LastModifiedDate` 기능을 동작시키기 위해 Auditing기능을 사용하도록 설정해줍니다.

```java
@EnableR2dbcAuditing // 추가
public class H2R2dbcConfig extends AbstractR2dbcConfiguration {
}
```

## UserRepository 추가

repository/UserRepository.java를 생성하고 다음과 같이 작성합니다.

```java
import com.example.r2dbc.model.User;
import org.springframework.data.repository.reactive.ReactiveCrudRepository;

public interface UserRepository extends ReactiveCrudRepository<User, Long> {
}
```

## UserService 추가

service/UserService.java를 생성하고 다음과 같이 작성합니다.

```java
import com.example.r2dbc.model.User;
import com.example.r2dbc.repository.UserRepository;

import org.springframework.stereotype.Service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Slf4j
@RequiredArgsConstructor
@Service
public class UserService {
    private final UserRepository userRepository;

    public Mono<Long> count() {
        return userRepository.count();
    }

    public Flux<User> findAll() {
        return userRepository.findAll();
    }

    public Mono<User> addUser(User user) {
        return userRepository.save(user);
    }

    public Flux<User> addUsers(Flux<User> users) {
        return userRepository.saveAll(users);
    }

    public Mono<Void> deleteUser(Long id) {
        return userRepository.deleteById(id);
    }

    public Mono<Void> deleteAll() {
        return userRepository.deleteAll();
    }
}
```

## UserApiController 추가

controller/UserApiController.java를 생성하고 다음과 같이 작성합니다.

```java
package com.example.r2dbc.controller;

import com.example.r2dbc.model.User;
import com.example.r2dbc.service.UserService;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Slf4j
@RequiredArgsConstructor
@RestController
public class UserApiController {
    private final UserService userService;

    @GetMapping("users")
    public Flux<User> findAll() {
        return userService.findAll();
    }

    @PostMapping(value = "users", consumes = MediaType.APPLICATION_JSON_VALUE)
    public Mono<User> addUser(@RequestBody User user) {
        return userService.addUser(user);
    }

    @DeleteMapping(value = "users/{id}", consumes = MediaType.APPLICATION_JSON_VALUE)
    public Mono<Void> deleteUser(@PathVariable Long id) {
        return userService.deleteUser(id);
    }
}
```

## Test Code 추가

`UserRepository`, `UserService`에 대해서 기능이 정상적으로 동작하는지  
테스트 코드를 작성해보겠습니다.

`test.com.example.webfluxboard.service`  
`test.com.example.webfluxboard.repository`

2개의 package를 추가합니다.

UserRepositoryTest.java
```java
import com.example.webfluxboard.model.User;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.time.LocalDateTime;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

@SpringBootTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        User user1 = User.builder().name("test1").age(10).updatedAt(LocalDateTime.now()).createdAt(LocalDateTime.now()).build();
        User user2 = User.builder().name("test2").age(20).updatedAt(LocalDateTime.now()).createdAt(LocalDateTime.now()).build();
        User user3 = User.builder().name("test3").age(30).updatedAt(LocalDateTime.now()).createdAt(LocalDateTime.now()).build();

        userRepository.deleteAll()
        .and(userRepository.saveAll(Flux.just(user1, user2, user3)))
        .subscribe();
    }

    @AfterEach
    void tearDown() {
        userRepository.deleteAll().subscribe();
    }

    @Test
    @DisplayName("사용자 테이블 deleteAll 테스트")
    public void whenDeleteAllThen0IsExpected() {
        userRepository.deleteAll()
                .as(StepVerifier::create)
                .expectNextCount(0)
                .verifyComplete();
    }

    @Test
    @DisplayName("사용자 테이블 findAll 테스트")
    public void whenFindAllThen3AreExpected() {
        userRepository.findAll()
                .as(StepVerifier::create)
                .expectNextCount(3)
                .verifyComplete();
    }

    @Test
    @DisplayName("사용자 테이블 추가 테스트")
    public void whenInserThen1AreExpected() {
        // given
        User user = User.builder()
                .name("test")
                .age(20)
                .updatedAt(LocalDateTime.now())
                .createdAt(LocalDateTime.now())
                .build();

        // when
        userRepository.deleteAll();
        Mono<User> u = userRepository.save(user);

        // then
        StepVerifier.create(u)
                .assertNext(it -> {
                    Assertions.assertEquals(user.getName(), it.getName());
                    Assertions.assertEquals(user.getAge(), it.getAge());
                    Assertions.assertEquals(user.getCreatedAt(), it.getCreatedAt());
                    Assertions.assertEquals(user.getUpdatedAt(), it.getUpdatedAt());
                })
                .verifyComplete();
    }
}
```

UserServiceTest.java
```java
import com.example.webfluxboard.model.User;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.time.Duration;
import java.time.LocalDateTime;

import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

@SpringBootTest
class UserServiceTest {

    @Autowired
    private UserService userService;

    @BeforeEach
    void setUp() {
        User user1 = User.builder().name("test1").age(10).updatedAt(LocalDateTime.now()).createdAt(LocalDateTime.now()).build();
        User user2 = User.builder().name("test2").age(20).updatedAt(LocalDateTime.now()).createdAt(LocalDateTime.now()).build();
        User user3 = User.builder().name("test3").age(30).updatedAt(LocalDateTime.now()).createdAt(LocalDateTime.now()).build();

        userService.deleteAll()
        .and(userService.addUsers(Flux.just(user1, user2, user3)))
        .subscribe();
    }

    @AfterEach
    void tearDown() {
        userService.deleteAll().subscribe();
    }

    @Test
    void count() {
        // when & then
        userService.count().as(StepVerifier::create)
                .assertNext(count -> Assertions.assertEquals(count, 3))
                .verifyComplete();
    }

    @Test
    void findAll() {
        System.out.println("findAll");
        // when & then
        userService.findAll().as(StepVerifier::create)
                .expectNextCount(3)
                .verifyComplete();
    }

    @Test
    void addUser() {
        User user = User.builder()
                .name("test")
                .age(20)
                .updatedAt(LocalDateTime.now())
                .createdAt(LocalDateTime.now())
                .build();

        // when & then
        StepVerifier.create(userService.addUser(user))
                .assertNext(it -> {
                    Assertions.assertEquals(user.getName(), it.getName());
                    Assertions.assertEquals(user.getAge(), it.getAge());
                    Assertions.assertEquals(user.getCreatedAt(), it.getCreatedAt());
                    Assertions.assertEquals(user.getUpdatedAt(), it.getUpdatedAt());
                })
                .verifyComplete();
    }

    @Test
    void addUsers() {
        //given
        Flux<User> users = Flux.interval(Duration.ofMillis(10))
                .map(i -> User.builder()
                        .name("test" + i)
                        .age(i.intValue())
                        .updatedAt(LocalDateTime.now())
                        .createdAt(LocalDateTime.now())
                        .build()
                )
                .limitRequest(10)
                .buffer()
                .flatMap(it -> Flux.fromIterable(it));

        // when & then
        userService.addUsers(users).as(StepVerifier::create)
                .expectNextCount(10)
                .verifyComplete();

    }

    @Test
    void deleteUser() {
    }
}
```

## test profile 설정 추가
이전 시간에 H2 Profile을 구성하였기 때문에,  
test 수행시 기본 profile을 `H2`로 설정해주어야 합니다.

build.gradle을 다음과 같이 수정합니다.
```java
import org.springframework.util.StringUtils
test {
	def profile = (StringUtils.isEmpty(System.getProperty("spring.profiles.active"))) ? "h2" : System.getProperty("spring.profiles.active")
	println("""
		|==================================
		|Test profile ${profile}
		|==================================
	""".stripMargin())

	systemProperty 'spring.profiles.active', profile
	useJUnitPlatform()
}
```
이제 다음 명령어 또는
```text
./gradlew test
```

Intelli J에서 테스트 코드를 수행합니다. 

![webflux4-2](https://github.com/tries1/glenn-blog/blob/master/assets/spring/webflux4_2.webp?raw=true)
