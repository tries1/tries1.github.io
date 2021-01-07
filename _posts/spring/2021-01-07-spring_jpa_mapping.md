---
layout: post
title:  "Spring Data JPA의 연관관계 매핑"
date:   2021-01-07 12:00:00
author: Glenn
categories: Spring
cover:  "/assets/instacode.png"
comments: true
---

## 연관관계 매핑 정의

객체는 참조로 연관관계를 가지고, 테이블은 외래키로 연관관계를 가진다.
테이블은 기본 양방향 관계지만, 객체는 설정된 방향에 따라서 다르다.
- 단방향 : 한객체가 다른객체를 참조
- 양방향 : 두 객체가 서로참조

- 객체에는 양방향 연관관계라는것이 없다.
	- 단반향연결이 양쪽에 되어있을뿐

## 매핑 어노테이션

@OneToOne
- 1 : 1 관계 매핑
- fetch() default EAGER

@ManyToOne
- N : 1 관계 매핑
- fetch() default EAGER

@OneToMany(mappedBy=“owner의 매핑된 field”)
- 1 : N 관계 매핑
- fetch() default LAZY
- mappedBy로 owner을 지정해야한다.

@ManyToMany(mappedBy=“owner의 매핑된 field”)
- N : N 관계 매핑
- fetch() default LAZY
- mappedBy로 owner을 지정해야한다.

@JoinColumn(name = "TEAM_ID”)
- 외래키 매핑할때 사용
- 양방향 매핑일때 사용하는데, 반대쪽 관계와 매핑되는 필드값을 주면된다.

fetch type
- EAGER : 연관관계 엔티티의 데이터를 즉시로딩
- LAZY : 연관관계 엔티티의 데이터를 호출하기전까지 로딩하지 않고, Proxy 객체를 바라본다.

## 연관관계의 주인(owner)

- 주인만이 데이터베이스 연관관계와 매핑되고, 외래키관리(등록, 수정, 삭제)할수있다, owner가 아니면 읽기만 가능.
- 주인 설정은 mappedBy로 할수있다.
- 주인은 mappedBy를 사용하지 않는다.
- owner는 외래키를 관리하고있는 엔티티가 되어야함.
- 테이블의 다대일, 일대다 관계에서는 항상 다쪽이 외래키를 가진다. 따라서 `@ManyToOne`은 `mappedBy`속성이 없다.

## 양방향 연관관계의 주의점

- 주인에는 값을 입력하지않고, 주인이 아닌곳에만 입력하면 외래키 등록이 안된다.
    - 외래키의 주인만이 외래키의 입력, 수정, 삭제를할수있다.
    - 주인이 아니면 읽기만 가능
	
- 양방향은 양쪽다 관계를 설정해주어야한다. 
    - 객체관점에서보면 한쪽만 연관관계를 지정하면, 예상치 못한 오류가 발생할수있다.
    - 주인이 아닌곳에서 연관관계를 지정해도 저장시 사용하진 않지만, 객체관점을 고려한다면 설정해줘야한다.
    - 이런 문제를 예방하기위해 연관관계 편의 메소드를 작성한다.

- 연관관계 편의 메소드
    - setter 메소드를 수정해서 연관코드가 같이수행되도록 수정하는것이 안전하다!
    - 기존 연관관계가 있다면 삭제후 새로운 연관관계를 등록해야한다!!

```java
public void setMember(Member member) {
    // 기존 관계가 있다면 삭제
    if (Objects.nonNull(this.member)) {
        this.member.getOrders().remove(this);
    }

    // 새로운 관계 등록
    this.member = member;
    
    // 반대편에도 등록
    this.member.getOrders().add(this);
}
```

## 정리
- 단방향 매핑만으로 연관관계 매핑은 완료
- 단방향을 양방향으로 만들면 반대방향에서 객체 그래프 탐색 기능이 추가된다.
- 양뱡항 연관관계는 객체 양쪽에서 모두 매핑해줘야 한다.
