---
layout: post
title:  "Java8 LocalDate 날짜 계산, 비교, 구하기, format 변경하기"
date:   2020-01-30 11:00:00
author: Glenn
categories: Java
cover:  "/assets/instacode.png"
comments: true
---

### Java8의 LocalDate를 사용한 날짜 계산
java8에서 날짜를 쉽게 다룰수 있도록 `LocalTime`, `LocalDate`, `LocalDateTime` API가 추가되었습니다.  


#### 현재시간 구하기
`now()`를 사용하여 현재시간은 구할수 있습니다.(장비의 설정된 시간을 따라갑니다.)

```java
LocalTime.now();
LocalDate.now();
LocalDateTime.now();
```

---

#### 특정시간 구하기
`of()`를 사용하여 특정시간으로 생성할수있습니다.

```java
LocalTime.of(10, 00);
LocalDate.of(2020, 01, 30);
LocalDateTime.of(2020, 01, 30, 10, 00, 00);

// 날짜포맷 변경
LocalTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"));
LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

---

#### 날짜포맷 변경하기
`format()`, `DateTimeFormatter`를 사용하여 원하는 포맷으로 날짜를 생성할수있습니다.

```java
LocalTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"));
LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

---

#### 날짜 계산
날짜 계산에 필요한 유용한 메서드등을 제공합니다.

```java
LocalDateTime now = LocalDateTime.now();

// 오늘이 이번달 몇일째인지
now.getDayOfMonth();

// 오늘이 1년중 몇일째인지
now.getDayOfYear();

// 오늘이 무슨요일인지
// ex) TextStyle.FULL = 금요일, TextStyle.SHORT = 금
now.getDayOfWeek().getDisplayName(TextStyle.FULL, Locale.KOREAN);
now.getDayOfWeek().getDisplayName(TextStyle.SHORT, Locale.KOREAN);

//어제 날짜는?
now.minusDays(1);
//저번달 날짜는?
now.minusMonths(1);
```

---

#### 날짜 비교
두날짜를 비교하고 결과를 리턴합니다.

```java
LocalDateTime yesterday = LocalDateTime.of(2020, 01, 29, 10, 00, 00);
LocalDateTime today = LocalDateTime.of(2020, 01, 30, 10, 00, 00);

// yesterday가 today보다 이전인가요?
yesterday.isBefore(today);

// 두날짜의 일수 차이는?
today.compareTo(yesterday);
```
