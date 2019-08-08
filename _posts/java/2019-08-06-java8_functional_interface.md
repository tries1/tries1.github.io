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

Java는 컴파일시 `Type Erasure`를 사용하여 타입정보를 알수없게됩니다.

이 때문에 런타임시 `Type Checking`이 어려운점이 있고, 

Java8에서 Lamda를 지원하기위해 `FunctionalInterface` 추가하였습니다.

`타입 추론` : 정해지지 않은 대상의 타입을 컴파일러가 유추하는 기능
 - ex) `var name = "glenn"` // String, Scala의 타입추론
 
`Type Erasure` : 자바의 컴파일러가 컴파일 시점에 타입의 정보를 제거함
 - ex) `List<User> users; -> List users;` // 타입 정보를 제거

  
*Java가 `Type Erasure`를 사용하게된 이유는 Java5에 추가된*  
*Generic을 사용하기 이전의 코드들에대한 하위호환성을 유지하기 위함이라고 알고있습니다.*

---

#### `FunctionalInterface`는 무엇일까?

우선 FunctionalInterface중 하나인 `Function<T, R>`을 살펴보겠습니다.

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    
    //나머지 내용은 생략하였습니다.
    ...
}
```

`FunctionalInterface` 어노테이션의 특징으로는 `Single Abstract Method`(단일 추상 메소드)이며,  
말그대로 `추상 메소드가 하나`를 가지고 있는지 확인을 해주는 용도입니다.

`추상 메소드가 하나`이며 이로인해 `Java`의 컴파일러가 `타입추론`을 할수있게 제공합니다.

*예외로 `default 메소드`, `static 메소드`는 `FunctionalInterface`에서 여러개 선언 될수있습니다.*

---

#### 기본적인 `FunctionalInterface`

Java8에 추가된 `FunctionalInterface`는 약 40여개이지만  
모든 `FunctionalInterface`를 알아둘필요 없이 기본적인것만 보도록 하겠습니다.

`Function<T, R>`
 - `T 타입`의 인자를 받아서 `R 타입`의 값을 리턴합니다.
 - `Stream API`의 `map`에서 사용
 ```java
Function<Integer, Integer> plusOneFunction = (x) -> x + 1;
plusOneFunction.apply(1);
Function<String, Integer> stringLength = (x) -> x.length();
stringLength.apply("glenn");
```
 
`Supplier<T>`
 - 인자를 받지않고 `T 타입`의 값을 리턴합니다.
 - get()이 호출되기 전까지 수행되지 않습니다.
 - `Optional`의 `orElseGet`에서 사용
 ```java
 Supplier<String> printNameSupplier = () -> "glenn";
 printNameSupplier.get();
 ```
 
`Predicate<T>`
 - `T 타입` 인자를 받아서 `boolean` 타입의 결과를 리턴합니다.
 - `Stream API`의 `filter`에서 사용
 ```java
 Predicate<String> isNotEmpty = (x) -> !x.isEmpty();
 isNotEmpty.test("glenn");
 ```
 
`Consumer<T>`
 - `T 타입` 인자를 받아서 결과를 리턴하지않습니다.(void)(이름 그대로 소비합니다.)
 - `Stream API`의 `forEach`에서 사용
 ```java
 Consumer<String> namePrint = (name) -> System.out.println(name);
 namePrint.accept("glenn");
 ```

---

#### 그 밖의 `FunctionalInterface`

위에서 모든 `FunctionalInterface`를 알필요가 없다고 말씀드렸는데요.  
그 이유는 위에서 보신 기본적인 `FunctionalInterface`알게 되었다면 나머지 `FunctionalInterface`는  
쉽게 이해하실수 있을겁니다.

`BiFunction<T, U, R>`
 - `Function<T, R>`에서 전달하는 인자가 하나 추가된 `FunctionalInterface`입니다.
 - `T 타입`, `U 타입`의 인자를 받아서 `R 타입`의 값을 리턴합니다.
 
```java
BiFunction<Integer, Integer, Integer> sumAndplusOneFunction = (x, y) -> x + y + 1;
sumAndplusOneFunction.apply(2, 2);
```

`BiPredicate<T, U>`
 - `Predicate<T>`에서 전달하는 인자가 하나 추가된 `FunctionalInterface`입니다.
 - `T 타입`, `U 타입`의 인자를 받아서 `boolean`의 결과를 리턴합니다.
```java
BiPredicate<Boolean, Boolean> isAllTrue = (x, y) -> x && y;
isAllTrue.test(true, false);
```
---

이 외에도 많은 `FunctionalInterface`가 있지만, 직접 `java.util.function`에서 확인해보시면

큰 어려움없이 이해하실듯합니다.
