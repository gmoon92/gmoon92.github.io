---
layout: post
title: "Hibernate Envers를 활용한 이력관리"
tags: [Spring, JPA, Hibernate, Spring Data JPA, Envers]
categories: [Spring, Hibernate, Envers]
subtitle: "Hibernate Envers Concepts"
feature-img: "md/img/thumbnail/hibernate-envers.png"
thumbnail: "md/img/thumbnail/hibernate-envers.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily

priority: 1.0
---

<!--more-->

# 엔티티의 이력관리하는 방법
# 1. Hibernate Envers Concepts

---

# 들어가기전

어느 날 사용자의 요구사항이 들어왔습니다.

_사용자가 변경한 데이터들을 추적할 수 있는 화면을 제공해주세요._

우선 다음과 같은 요구사항을 해결하기 위해 여러 개발 방법을 논의할 수 있습니다. 그중에서 가장 단순한 방식으론 추적할 엔티티와 관련된 로직에 추적할 수 있는 로직을 추가하는 방법입니다.

``` java
public void saveMember(MemberVO vo){
    ... 기존 회원 엔티티 저장 로직
    saveMemberHistory(vo, HistoryStatus.INSERT);
}

private void saveMemberHistory(MemberVO vo, HistoryStatus status){
    ... 회원 엔티티 데이터를 추적할 히스토리용 엔티티 저장 로직
}
```

가장 단순한 방식이지만, 다음 방식을 진행하기 위해선 몇 가지 선행되어야 할 과정이 있습니다.

1. 변경된 데이터를 추적할 테이블 정리
2. 별도의 이력 테이블을 생성
3. 추적할 엔티티와 해당 엔티티를 사용하는 비즈니스 로직 파악 및 정리
4. 기존 비즈니스 로직에 부가적인 이력 코드를 추가

이 과정들은 관리 포인트를 의미하며, 이러한 관리 포인트가 증가하면 개발자의 실수를 유발할 수 있는 사이드 이펙트가 동반될 수밖에 없습니다. 또한, 비즈니스 로직 코드상에서 별개의 코드들이 추가됨으로 결과적으로 다음 개발 방식은 효율이 너무 낮습니다.

이러한 경우에 Hibernate Envers를 사용하게 되면 `사용자가 변경한 데이터들을 추적할 수 있는 화면을 제공해주세요.` 라는 요구사항에 대해 해결하는 과정에서 발생되는 사이드 이펙트를 고려하지 않고 별도로 추적할 데이터들을 관리할 수 있게 됩니다.

# 학습 목표

- Hibernate Envers의 개념
- Hibernate Envers의 동작원리

일반적으로 추적할 데이터들을 **이력 데이터**라 하는데 본 포스팅에선 Hibernate Envers에 대한 개념과 어떻게 이력 데이터를 관리하는지에 대한 이론적인 설명 중심으로 작성되었습니다.

## 1. Concepts

Hibernate는 이력 데이터를 쉽게 관리할 수 있도록 Envers라는 패키지를 제공해주고 있습니다. 따라서 Envers 패키지에 포함된 클래스와 애노테이션들은 전적으로 애플리케이션 엔티티 데이터의 이력 데이터를 쉽게 관리할 수 있도록 제공해주고 있습니다.

Hibernate Envers의 목적은 이력 데이터를 버전별로 쉽게 관리하는 데 있습니다. 예를 들어 우리가 흔히 소스 코드를 관리하기 위해 사용하고 있는 Git 또는 Subversion과 같은 VCS(Version Controller System)처럼 **"Hibernate Envers는 엔티티의 데이터 개정(Revision)을 관리하기 위해 사용한다."**라고 이해하시면 되겠습니다.

## 2. 동작 원리

우선 Hibernate Envers를 학습하는 데 도움이 되는 키워드를 크게 세 가지로 분류할 수 있습니다.

- Revision Table
- Audited Table
- Transaction

일반적으로 엔티티의 데이터 변경은 트랜잭션 단위에서 이뤄집니다. 이러한 Hibernate의 동작 방식에 따라 Hibernate Envers는 각각의 트랜잭션마다 하나의 Revision Number(개정 번호)를 가질 수 있도록 동작하게 됩니다.

> 참고로 Hibernate Envers는 생성된 Revision Number를 별도의 Revision 테이블에서 관리하게 됩니다.

Revision Number는 각각의 트랜잭션마다 새롭게 생성되기 때문에 코드상으로 어떠한 엔티티의 그룹이 변경되고 있는지 식별하는 데 활용할 수도 있습니다.

식별한다는 의미에 대해 이해를 돕기 위해 코드와 함께 예를 들어 봅시다.

``` java
@Transactional
public void changeTeam(MemberVO vo){
	// 회원의 Revision Number 1
	Team team = memberRepository.changeTeam(vo)
															.getTeam();

	// 부서의 Revision Number 1
	... 부서 이동에 의한 코드
	teamRepository.change(vo, team);
}
```

다음 코드는 부서 이동이라는 기능을 지닌 메서드입니다. 비즈니스 로직 특성상 같은 트랜잭션 안에서 회원 엔티티와 부서 엔티티의 데이터가 변경되고 있습니다. 이때 Hibernate Envers는 트랜잭션에 대해 Revision Number를 생성해주고, 생성된 Revsion Number를 해당 엔티티의 **이력관리 테이블(Audited Table)**에 저장하게 됩니다.

![img](/md/img/hibernate/envers/hibernate-envers-tables.png)

정리를 하자면 해당 비즈니스 로직에서 변경된 엔티티들에 대한 이력관리 테이블의 Revision Number(REV) 컬럼엔 같은 데이터를 가지게 됩니다. 이를 토대로 이력관리 테이블의 데이터상으로 회원과 부서 엔티티가 변경되는 로직이 존재한다고 식별할 수 있습니다. 또한, Revision Number를 글로벌하게 관리함에 따라 개발적인 이점을 취할 수 있습니다.

1. 다양한 이력 엔티티에 대해 조회가 가능해진다.
2. 데이터베이스 상에서 세부적으로 검색할 수 있다.

각각의 이력 테이블의 Revision Number(REV)가 존재하고, Revision Number는 Hibernate Envers에 의해 생성된 개정 테이블의 PK를 의미하기 때문에 다양한 이력 엔티티에 대해 조회가 가능해집니다.

마지막으로 개정 테이블에는 Revision Number 뿐만 아니라 커밋된 날짜(Revision Time Stamp, REVTSTMP)를 포함하여 관리하고 있으므로 특정 날짜에 대한 이력 데이터를 조회할 수 있고, 이력에 대한 커밋된 시점을 알 수 있습니다.

# 앞으로 진행될 내용

지금까지 Hibernate Envers에 대한 대략적인 개념과 동작 방식과 관련된 이론적인 설명이 많았습니다.

물론, 개념과 동작 원리라고 하기엔 생략된 부분들이 많습니다. 예를 들어 이력 테이블에 존재하는 REVTYPE 컬럼은 무엇을 의미하는지, Audited Table은 어떻게 생성하고 관리해야 하는지 Revision Number는 어느 시점에 생성되는지 등등 궁금증이 생기실 것입니다. 앞으로 작성될 포스트를 통해 제기된 궁금증들을 풀어갈 예정이며 보다 Hibernate Envers에 어떻게 사용할 수 있는지에 대해 설명을 하겠습니다.

- Audited Entity
- Custom Revision Entity
- EntityTrackingRevisionListener
- EventListenerRegistry

# 참고

[jboss-docs-hibernate-4.1-Envers](https://docs.jboss.org/hibernate/core/4.1/devguide/en-US/html/ch15.html#envers-tracking-modified-entities-queries)