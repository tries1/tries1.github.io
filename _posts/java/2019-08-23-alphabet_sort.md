---
layout: post
title:  "Java8 알파벳순서대로 정렬하기"
date:   2019-08-23 15:00:00
author: Glenn
categories: Java
cover:  "/assets/instacode.png"
comments: true
---

#### Java8의 Stream API를 사용한 알파벳순서대로 정렬하기입니다.

*`sorted()`* 

```java
String alpha = "kashdfuihSDFKFCaaaWOYHaVBWIACmzxnGgvdfahgtgNdfsfhuaiwhVEHwetvavxISCDPAJ";
Arrays.stream(alpha.split(""))
.sorted()
.forEach(System.out::print);

//Result
//AABCCCDDEFFGHHIIJKNOPSSVVWWYaaaaaaaadddeffffggghhhhhiikmnssttuuvvvwwxxz
```

`naturalOrder`가 적용된 `사전에 정의된 순서`대로 정렬이됩니다.

---

*`String.CASE_INSENSITIVE_ORDER`* 

```java
String alpha = "kashdfuihSDFKFCaaaWOYHaVBWIACmzxnGgvdfahgtgNdfsfhuaiwhVEHwetvavxISCDPAJ";
Arrays.stream(alpha.split(""))
.sorted(String.CASE_INSENSITIVE_ORDER)
.forEach(System.out::print);

//Result
//aaaaaAaaaABCCCdDddDEefFFfffGggghhHhhhHiIiIJkKmnNOPsSsSttuuVvVvvWWwwxxYz
```

알바펫 순서대로 정렬이 되었지만, 대소문자가 섞여있습니다.

---

*`compareToIgnoreCase()`* 

```java
String alpha = "kashdfuihSDFKFCaaaWOYHaVBWIACmzxnGgvdfahgtgNdfsfhuaiwhVEHwetvavxISCDPAJ";
Arrays.stream(alpha.split(""))
.sorted((o1, o2) -> {
    int res = o1.compareToIgnoreCase(o2);
    return (res == 0) ? o1.compareTo(o2) : res;
})
.forEach(System.out::print);

//Result
//AAaaaaaaaaBCCCDDdddEeFFffffGgggHHhhhhhIIiiJKkmNnOPSSssttuuVVvvvWWwwxxYz
```

대소문자 구분없이 비교후, 같을경우 `compareTo()` 사용하여 `naturalOrder` 적용  
다를경우 `compareToIgnoreCase()`가 적용

*소문자를 먼저 출력하고 싶다면 `o1.compareTo(o2)` -> `o2.compareTo(o1)`로 변경하면 됩니다.*

---

추가로 정렬된 알파벳을 다시 `String`으로 얻을려면 다음과같이   
`collect()`를 사용하여 합쳐주면됩니다. 
```java
String alphaSorted = Arrays.stream(alpha.split(""))
.sorted((o1, o2) -> {
    int res = o1.compareToIgnoreCase(o2);
    return (res == 0) ? o1.compareTo(o2) : res;
})
.collect(Collectors.joining());
```
