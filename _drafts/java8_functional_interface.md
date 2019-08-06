---
layout: post
title:  "Java8의 Lamda와 FunctionalInterface"
date:   2019-08-06 11:00:00
author: Glenn
categories: Java
cover:  "/assets/instacode.png"
---

#### Java8에 Lamda가 추가되면서 다음과 같은 프로그래밍이 가능해졌습니다.

```java
IntStream.rangeClosed(1, 10)                // 1.. 10까지 반복
         .map(i -> i + 2)                   // 2씩 더하고
         .filter(i -> (i % 2) == 0)         // 2로 나눈값이 0인값만 선별해서
         .peek(i -> System.out.println(i))  // 대상값들을 확인하기위해 출력
         .sum();                            // 모든값을 더한다
```

타입이 명시되지 않았는데, 어떻게 타입을 알수있을까?

---

#### Java의 `타입 추론`과 `Type Erasure`

`타입 추론` : 정해지지 않은 대상의 타입을 컴파일러가 유추하는 기능
 - ex) `var name = "glenn"` // String, Scala의 타입추론
 
`Type Erasure` : 자바의 컴파일러가 컴파일 시점에 타입의 정보를 제거함
 - ex) `List<User> users;` -> `List users;` // 타입 정보를 제거

Java는 컴파일시 `Type Erasure`를 사용하여 타입정보를 알수없게됩니다.  
이 때문에 런타임시 `Type Checking`이 어려운점이 있고,  
Java8에서 Lamda를 지원하기위해 `FunctionalInterface` 추가하였습니다.
  
*Java가 `Type Erasure`를 사용하게된 이유는 Java5에 추가된*  
*Generic을 사용하기 이전의 코드들에대한 하위호환성을 유지하기 위함이라고 알고있습니다.*

---

#### FunctionalInterface는 무엇일까?

우선 FunctionalInterface중 하나인 `Function<T, R>`을 살펴보겠습니다.

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    
    //나머지 내용은 생략하였습니다.
    ...
}
```



Single Abstract Method(단일 추상 메소드)



#### 기본적인 FunctionalInterface

Java8에 추가된 FunctionalInterface는 약 40여개

Function
Supplier
Predicate
Consumer
