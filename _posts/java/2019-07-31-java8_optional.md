---
layout: post
title:  "Java8 Optional을 활용하여 Null pointer exception 예방하기 (if문 대체하기)"
date:   2019-07-31 11:00:00
author: Glenn
categories: Java
cover:  "/assets/instacode.png"
comments: true
---

#### 프로그래밍을 하다보면 자주 보게되는 예외중 하나가 NPE(NullPointerException) 입니다.
Java8에서 `Optional`이 등장하기전에는 NPE를 방지하기위해 보통 `if`문을 사용하여 처리하였습니다.

*`if`문을 이용하여 null처리* 
```java
String text = null;

if (text != null) {
    System.out.println("text is " + text);
} else {
    System.out.println("text is null!");
}

```

---

#### 이제 `Optional`을 이용하여 처리하는 방법을 확인해보겠습니다.

*기본적인 `Optional`을 생성하는 방법*
```java
//null을 허용하지 않는 Optional 생성(text가 null이면 NPE 발생!!)
Optional.of(text);

//null을 허용하는 Optional 생성
Optional.ofNullable(text);

//비어있는 Optional 생성
Optional.empty();
```

---

*`if`문을 통해 처리하던 부분을 `Optional`로 변경*
```java
String text = null;

Optional<String> optionalText = Optional.ofNullable(text);

//get 메소드는 강제로 꺼내오기때문에 null일경우 "NoSuchElementException: No value present" 에러가 발생하니 유의해야합니다.
optionalText.get(); 

//optionalText가 null일경우 "text is null!" 제공
optionalText.orElse("text is null!"); //text is null! 출력

//orElseGet이 무엇인지는 본문 하단에서 보도록 하겠습니다.
optionalText.orElseGet(() -> "text is null!"); //text is null! 출력

```

---

#### `Optional`을 조금더 활용해보기
Java8에 함께 추가된 Functional interface을 같이 사용해보겠습니다.
```java
String text = null;

//map은 Functional interface중 하나인 Function 사용합니다.
Optional.ofNullable(text)
        .map(s -> "text is " + s)//string 앞에 "text is " 추가
        .orElse("text is null!");//text가 null일경우 orElse가 호출됩니다.

```

---

*체이닝을 통해 연결해서 사용이 가능합니다.*
```java
String text = null;

Optional.ofNullable(text)
        .filter(s -> !s.isEmpty())
        .map(s -> "text is " + s)
        .map(s -> s + "!!")
        .map(String::trim)
        .orElse("text is null!");
```
`map` : Functional interface중 `Function` 사용
 - Function : `Function<T, R>` 인자 T를 받아서 R type을 return 
 
`filter` : Functional interface중 `Predicate` 사용
 - Predicate : `Predicate<T>`  인자 T를 받아서 boolean type을 return

---

#### orElse와 orElseGet의 차이

*Optional 연산중 null값을 만나게되면 orElse 또는 orElseGet을 호출하게됩니다.*

```java
String text = null;
Optional<String> optionalText = Optional.ofNullable(text);

optionalText.orElse("text is null!"); //text is null! 출력
optionalText.orElseGet(() -> "text is null!"); //text is null! 출력
```

*둘다 같은 결과가 나오지만 매우 큰 차이가 존재합니다.*

`orElse` : 미리 계산된 값?을 가져옵니다.

`orElseGet` : Functional interface중 하나인 `Supplier` 통해서 값을 가져옵니다.

**Supplier는 get메소드가 호출되어질때 값을 계산하고 결과를 가져옵니다.(lazy evaluation)**
```java
Supplier<String> stringSupplier = () -> "hello";
stringSupplier.get();
```

한가지씩 예를들어 보겠습니다.
* 검색결과가 없을때 "결과없음"이라는 고정된값(상수) String값을 return할때
  - `orElse` 사용
  - `orElse`에는 **상수를 반환하는것**이 좋습니다.(사용하지 않더라고 호출되기때문이죠)
  
* Cache에서 값을 찾지 못했을때 DB에서 값을 호출하고 싶을때
  - `orElseGet` 사용
  - cache에서 **값이 없을때** `orElseGet`이 호출이되면서 DB를 호출

---

#### 추가로 `orElse` or `orElseGet`에서는 null을 return하지 않아야합니다.

`Optional`을 사용하는 이유가 NPE예방인데 null을 return하게되면 해당 값을 사용하는 다른곳에서 NPE가 발생할수도 있기 때문입니다.
