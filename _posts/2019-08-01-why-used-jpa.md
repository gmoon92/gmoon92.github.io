---
layout: post
title: "JPA를 사용해야 하는 4가지 이유"
tags: [JPA, Hibernate]
categories: [JPA]
subtitle: "왜 JPA를 사용해야 하는 걸까?"
feature-img: "md/img/thumbnail/why-used-jpa.png"
thumbnail: "md/img/thumbnail/why-used-jpa.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# 왜 JPA를 사용해야 하는 걸까?

---

### 들어가기전

> "실무에서 테이블 설계는 제대로 하면서, 객체 모델링은 왜 하지 않을까?" - 자바 ORM 표준 JPA 프로그래밍 중에서

다음 문장은 JPA를 사용해야 하는 이유인 동시에 JPA를 학습해야 하는 이유를 잘 들어낸 문장이라 생각합니다. 그렇다면 구체적으로 JPA를 사용하면 어떠한 장점이 있고 사용해야 하는지에 대해 포스팅을 작성해보려 합니다.


### 학습목표

- JPA의 장점에 대한 이해
- JPA를 학습해야 되는 이유

### 1. JPA란 무엇일까?

JPA는 Java Persistence API라 하여 Java 진영의 ORM의 기술 표준을 의미합니다.

_객체(Object) ← 매핑(Mapping) → 관계(Relational)_

ORM은 Object Relational Mapping의 약자로, Object를 의미하는 객체와 Relational를 의미하는 관계형 데이터 베이스의 데이터를 맵핑 시켜주는 기술이라 할 수 있습니다.

#### 1.1. JPA의 장점

- CRUD SQL 작성 필요 X
  - 자동 SQL 조회 및 매핑
- SQL 중심이 아닌 객체 중심의 개발
  - 생산성 ↑ 유지보수 ↑
- DB 정책 변경에 용이(Data Base 방언들을 해결)
  - MySQL → Oracle DB로 정책이 바뀌더라도 쉽게 대응할 수 있다.
    - ex) SQL를 의존하여 개발하지 않고 JPA를 활용하여 객체로 개발을 하기 때문

최종적으로 귀찮은 SQL 작성과 전달할 데이터를 객체에 자동으로 매핑해주는 작업을 JPA는 객체로써 이 모든 걸 해결해주고 있습니다.

### 2. 기존 객체 매핑에 관련된 문제들

아무래도 다수의 개발자가 JPA에 대한 관심은 무엇보다 개발의 생산성을 증가시켜주기 때문은 아닌지 생각합니다.

특히 개발 영역에 있어 RDB(관계형 데이터베이스)는 신뢰할 만한 데이터 저장소로 대부분의 개발 환경에서 RDB를 사용할 거로 생각합니다. 따라서 데이터를 DB에 저장하기 위해선 SQL를 사용해야 하는데, 특히 SQL 작성과 데이터 매핑 작업은 필수적인 과정입니다. 우선 기존에 사용했던 개발 방식을 살펴봅시다.

#### 2.1. 패러다임의 차이

![img](/md/img/why-used-jpa/dev-cycle.png)

1. SQL 작성 및 JDBC API 메소드를 활용하여 코드 작성
2. JDBC API에 의해 SQL 실행
3. DB 결과 조회
4. DAO 개발/매핑 코드 작성

과정을 살펴보자면 SQL을 작성한다든가 DB에 조회된 결과를 VO에 매핑하는 작업등, 이러한 일련의 작업 과정들은 최종적으로 개발자가 DB의 데이터를 객체로 사용하기 위해서 이뤄집니다.

근본적으로 이러한 과정이 필연적인 이유는 바로 DB와 객체간의 **패러다임의 차이**라 할 수 있습니다. RDB는 말 그대로 관계형 **데이터** 베이스입니다. 데이터를 중심으로 설계/구조화된 산물이라 할 수 있습니다. 반면 Java와 같이 OOP를 근간으로 객체를 중심으로 개발하는 방식을 따르고 있습니다.

_객체(Object) ← 개발자의 역할 → 관계(Relational)_

따라서 **개발자**가 **객체**와 **DB** 사이에서 사용할 특정 데이터를 가공해서 객체로 활용하기 위해 **변환(매핑)해주는** 작업을 해주고 있는 겁니다. 또한, 테이블을 중점으로 객체를 맞추다 보면 객체로써 활용될 수 있는 이점들을 다 포기한 채 데이터를 전달하는 용도로 사용할 수밖에 없습니다.

#### 2.2. 물리적으로는 계층 분리 성공, 하지만 논리적으론 실패

다른 사례로 예를 들자면 테이블의 칼럼 속성 또는 SQL을 수정된 상황이라 가정해봅시다.

1. 수정된 테이블의 칼럼과 관련된 SQL과 이와 관련된 모든 DAO를 찾는다.
2. 변경된 상황에 맞게 DAO에 작성된 SQL를 수정/추가 작업을 수행한다.
3. 또한, 칼럼이 추가 또는 수정되었다면 이에 맞게 VO도 수정/추가 작업을 병행한다.

다음과 같이 테이블 또는 SQL이 변경되는 상황을 대처하기 위해 3가지의 영역에서 수정/추가 작업을 병행해야 합니다.

이뿐만이 아닌 다른 테이블과 함께 조회되어야 할 상황이라면 어떻게 될까요?

``` sql
-- [1] 기존 회원만 조회하는 SQL
SELECT M.*
  FROM MEMBER M

-- [2] 팀과 함께 조회하는 SQL
SELECT M.*, T.*
  FROM MEMMBER M
  JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

[1] 기존 회원만 조회되는 SQL를 [2]와 같이 수정해야 하고 SQL이 바뀌면서, 이와 연관된 DAO도 확인해야 하고 수정을 해야 합니다. 핵심은 SQL만 수정했지만, 개발자는 관리할 포인트가 많다는 점입니다.

![img](/md/img/why-used-jpa/layer.png)

이처럼 물리적으로는 계층을 분리하여 개발하지만, 논리적으론 서로 강하게 의존되어 있기 때문에 최종적으로 각각의 영역을 신뢰할 수 없는 구조입니다.

JPA는 엔티티로 이러한 문제를 해결해줍니다.

> DataBase에서 엔티티란? <br/>
> Table = Entity = 개체라 생각해도 무방하다. <br/>
> 즉 엔티티는 표현하려는 데이터들의 집합이라 생각하면 되고, 엔티티엔 테이블간의 관계를 나타날때 사용되는 식별자(Key)와 속성(Attribute)로 구성되는 특징이 있다.

Java에선 비즈니스 요구사항을 모델링한 객체를 뜻합니다. 본론으로 돌아와서 JPA는 @Entity를 활용하여 **객체와 데이터간의 자동 매핑을** 해주고 자체적인 기술을 통해 **SQL도 자동생성해주는 기능까지 존재합니다.**

### 3. 패러다임의 불일치 해결

무엇보다 JPA가 많은 개발자에게 사랑받는 이유는 객체와 DB간의 불일치한 패러다임을 해결해주기 때문이라고 생각합니다.

- 집합적인 사고
- 데이터 중심
- 캡추다상정 개념 X
  - OOP의 특징이라 할 수 있는 캡슐화, 추상화, 다형성, 상속, 정보은닉 개념이 없다.

우선 DB는 다음과 같은 특징이 있습니다.

특히 DB에선 OOP의 특징을 잘 살려 설계를 하기엔 무리가 있습니다. 물론 마스터(슈퍼)-디테일(서브) 관계라 하여 구조적으로 상속이라는 특징을 살린 테이블 구조로 설계할 수 있습니다.

![img](/md/img/why-used-jpa/object-vs-relation.png)

예를 들어 다음 그림처럼 **`types`** 라는 특정 구분자를 두어 OOP의 상속이라는 특징을 살린 구조로 테이블을 설계할 수 있습니다. 하지만 패러다임 불일치를 해소해주기 위해 번거로운 코드가 많이 소비됩니다.

#### 3.1. JPA의 특징

이러한 패러다임 불일치를 해결해주는 JPA의 특징으론 다음과 같습니다.

- 연관 관계, 참조와 방향성
- 객체지향 모델링
- 객체 그래프 탐색
- 객체 비교

#### 3.1.1. 연관 관계, 참조와 방향성

우선 객체는 참조(레퍼런스)에 의해 객체간의 관계를 성립합니다.

``` java
public Class Member{
    ...
    Team team; // 참조

    // 수정자 메소드로 관계를 성립
    public void setTeam(Team team){
        this.team = team;
    }
}
```

반면 DB는 일반적으로 외래키(FK)를 사용하여 관계를 성립합니다.

``` sql
SELECT *
  FROM MEMBER M
  JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
--                  ^-- FK를 사용하여 관계 성립
```

_참조(Object) vs 외래키(Relational)_

여기서 중요한 점은 객체는 관계를 성립하기 위해 참조(레퍼런스)를 사용하고, 참조는 방향성이 존재합니다.

![img](/md/img/why-used-jpa/object-reference.png)

즉 참조가 있는 Member 객체에서만 Team을 조회할 수 있습니다. 이 점은 외래키만 있으면 어느 테이블에서 조회할 수 있었던 DB와는 극명한 차이라 할 수 있습니다.

#### 3.1.2. 객체지향 모델링이 가능하다.

앞서 보신 그림과 같이 Member 객체에서 Team 객체와 관계를 맺기 위해, **`Team team`** 필드를 사용했다는 점입니다.

``` java
member.setTeam(team); // 회원, 팀 연관 관계 설정
jpa.persist(member);
// ^-- 회원의 연관관계를 함께 저장, JPA가 변환 작업을 수행
```

이는 DB처럼 특정 id에 해당되는 외래키가 아닌 객체 그대로 사용할 수 있고, 패러다임 불일치를 해결하고자 개발자가 중간에서 변환 작업을 수행했던 과정들을 JPA가 자동으로 해결해주고 있습니다.

#### 3.1.3. 객체 그래프 탐색

객체 그래프 탐색은 JPA의 가장 중요한 특징이라 할 수 있습니다.

객체 그래프 탐색이란 특정 객체를 조회할 때 참조를 사용하여, 연관되는 객체를 찾는걸 의미합니다. 예를 들어 다음과 같이 회원과 팀을 조회하는 쿼리가 존재한다고 가정해봅시다.

``` sql
SELECT M.*, T.*
  FROM MEMBER M
  JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

이때 쿼리는 실행하는 SQL문에 의해 객체 그래프 탐색의 범위가 정해집니다.

이는 `2.2. 물리적으로는 계층 분리 성공, 하지만 논리적으론 실패` 파트에서 설명드렸던 신뢰할 수 없는 구조라 할 수 있습니다. 아무래도 DAO를 직접 확인하여 SQL문을 확인하기 전까지 객체가 어디까지 탐색이 가능한지 모르기 때문입니다.

JPA에선 이를 해결하기 위해 2가지 로딩 기법을 제공해주고 있습니다.

- Lazy Loading → 지연로딩
- Eager Loading → 즉시로딩

우선 지연 로딩이란 참조하고 있는 객체를 사용하기 전까지 SQL를 지연시킨다는 특징을 지니고 있습니다.

``` java
Member member = jpa.find(Member.class, memberId);
// select * from member 조회

Team team = member.getTeam();
// select * from team 조회
```

지연로딩은 team 객체를 사용할때 SQL이 조회됩니다. 즉시(Eager) 로딩은 이와 반대로 Member 클래스와 Team이 함께 조회됩니다. 결과적으로 JPA가 자동으로 SQL문을 생성해주기 때문에 @Entity로 지정된 Member 객체만 집중할 수 있게 됩니다.

#### 객체 비교

마지막으로 비교라는 특징에 대해 알아보겠습니다.

우선 DB는 기본키(PK)로 조회된 Row 데이터를 비교합니다. 반면 객체는 동일성(identity), 동등성(equality)라는 두 가지 방식을 통해 비교할 수 있습니다.

- 동일성( == ) : 인스턴스의 주소값 비교
- 동등성( equals() ) : 객체의 내부값 비교

여기서 핵심은 조회된 동등성은 해결할 수 있지만 동일성을 보장하기 쉽지 않습니다. JPA는 같은 트랜잭션일때 동일성이 보장되는 객체를 반환해줍니다.

``` java
@Transactional
public void findMember(Long memberId){
    Member member1 = jpa.find(Member.class, memberId);
    Member member2 = jpa.find(Member.class, memberId);

    System.out.println(member1 == member2); // true
}
```

### 마무리

JPA를 만나기 전 저는 실무에서 DAO와 VO 같은 객체는 단순히 데이터베이스에 데이터를 전달하거나 받는 목적으로 개발했습니다. 물론 번거롭고 귀찮은 과정이지만 당연히 필요한 과정이라 생각했고, 객체지향의 장점을 포기한 채 반복 작업을 개발자의 숙명이라 여겼습니다.

하지만 JPA를 공부하고 직접 개발해본 경험을 비춰봤을 때, JPA는 반복작업 이외에도 여러 문제를 개발자 대신 해결해줬습니다. 아무래도 관행적으로 소비되었던 시간을 JPA가 해결해줌으로써 비즈니스의 로직에 더 집중하여 개발할 수 있도록 도와줍니다.

물론 많은 선배 개발자분들이 실무에서 JPA를 도입하기에 앞서 JPA를 제대로 이해하고 사용해야 한다고 합니다. 이처럼 JPA의 학습에 대한 러닝 커브가 높은 편입니다. 그런데도 왜 굳이 JPA를 사용해야 하는지에 대해 작성하려 합니다. 이 글이 이제 막 JPA를 시작하는 사람들 또는 사용할지 고민하는 개발자분들을 위해 포스팅을 남기겠습니다.

### 참고

- 자바 ORM 표준 JPA 프로그래밍