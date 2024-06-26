---
layout: post
title: "JPA 성능 N+1 문제와 해결 방법"
tags: [Spring, JPA, Hibernate, Spring Data JPA, N+1, JPA 튜닝]
categories: [Spring, JPA, Hibernate, N+1]
subtitle: "JPA 매커니즘과 Proxy 그리고 N+1 문제"
feature-img: "md/img/thumbnail/hibernate.png"
thumbnail: "md/img/thumbnail/hibernate.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily

priority: 1.0
---

<!--more-->

# JPA 성능 - N+1 문제와 해결 방법

---

# 들어가기전

JPA의 ORM 기술은 양날의 검과 같다.

오늘날 ORM 기술은 비즈니스 로직에 좀 더 집중할 수 있는 환경을 개발자에게 제공해주고 있다. 하지만 어떤 이들은 쿼리를 많이 발생시켜 애플리케이션의 성능 저하를 발생시킨다는 의견이 존재한다. 흔히 N+1 문제다.

이는 JPA의 동작 방식을 이해한다면, JPA 어노테이션과 다양한 방법을 통해 해결할 수 있다. 필자는 일대일 연관 관계를 통해 JPA의 전반적인 메커니즘과 N+1 문제에 대해 소개하려 한다.

> N+1 해결 방법으론 QuickPerf 테스팅 라이브러리를 [QuickPerf 라이브러리 소개 with JPA N+1](https://gmoon92.github.io/spring/jpa/hibernate/n+1/testing/quickperf/2021/12/05/quickperf.html) 참고하자.

# 학습 목표

1. 글로벌 패치 전략
2. Proxy
3. N+1 문제와 해결 방법

# 환경

- Spring boot 2.4.1
- Hibernate 5.4.25.Final
- Querydsl 4.4.0
- h2 1.4.200

----

## 1. @OneToOne 모델링

우선 사용자 - 사용자 옵션의 도메인 모델을 구성해보자.

![member-domain-relations](/md/img/hibernate/n-plus-one/member-domain-relations.png)

- 식별 관계
- 대상 테이블에서 외래키 관리

다음 조건을 충족하기 위해선 양방향으로 매핑한다.

> 기본적으로 JPA의 일대일 관계에선 대상 테이블(MemberOption)에서 외래키를 저장하는 단방향 매핑을 지원하지 않는다. 따라서 대상 테이블에서 외래키를 관리하는 구조라면 양방향으로 매핑한다.

## 1.1. 사용자 도메인

```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(name = "name")
    private String name;
    
    @OneToOne(mappedBy = "member", cascade = CascadeType.ALL)
    private MemberOption memberOption;
}
```

## 1.2. 사용자 옵션 도메인

```java
@Entity
public class MemberOption {
    
    @Id
    @Column("member_id")
    private Long memberId;
    
    @MapsId
    @OneToOne
    @PrimaryKeyJoinColumn(name = "member_id")
    // ^-- 식별 관계 (JPA 2.0 이상)
    private Member member;
    
    @Column(name = "enabled")
    private boolean enabled;
    
    @Column(name = "retired")
    private boolean retired;
}
```
> The PrimaryKeyJoinColumn annotation specifies a primary key column that is used as a foreign key to join to another table. - [Difference between @JoinColumn and @PrimaryKeyJoinColumn](https://stackoverflow.com/questions/3417097/jpa-difference-between-joincolumn-and-primarykeyjoincolumn)

여기까지 도메인 설계는 끝이 났다. 그다음으로 글로벌 패치 전략을 고민해봐야 한다. 참고로 일대일 연관관계의 기본 글로벌 패치 전략은 EAGER이다.

|FetchType|장점|단점|
|---|---|---|
|LAZY|1. 빠른 클래스 로드 시간<br/>2. 적은 메모리 소비|1. N+1 : lazily-initialized 될 경우 성능에 영향을 미칠 수 있다.<br/>2. LazyInitializationException : 어떤 경우에는 특별한 주의를 기울여 lazily-initialized 된 개체를 처리해야 하거나 예외가 발생할 수 있다.|
|EAGER|1. lazily-initialized과 관련된 성능 영향 없음|1. 긴 클래스 로드 시간<br/>2. OOM heap : 불필요한 데이터를 너무 많이 메모리에 로드되면 성능에 영향을 미칠 수 있다.|

## 2. 글로벌 패치 전략

먼저 Lazy 패치 전략 동작 방식에 대해 살펴보자. 글로벌 패치 전략은 연관 관계 어노테이션의 fetch 속성을 통해 정의할 수 있다.

## 2.1. @OneToOne - Lazy Loading

```java
public class Member {
    // ...
    @OneToOne(mappedBy = "member"
         , cascade = CascadeType.ALL
         , fetch = FetchType.LAZY) // Lazy loading
    private MemberOption memberOption;
 
}
```

Lazy 패치 전략이 정상적으로 동작하는지 확인하기 위해선, Member 엔티티 객체를 조회할 때 MemberOption가 Proxy 객체로 반환되면 Lazy 패치 전략으로 정상 동작한다고 생각하면 된다. Proxy 여부는 `Hibernate.isInitialized`를 통해 확인해볼 수 있다.

```java
class MemberRepositoryTest{
    // ...
    @Test
    void testOneToOneLazy() {
        EntityManager em = getEntityManager();
        Member member = em.find(Member.class, 1L);
        MemberOption memberOption = member.getMemberOption();
        log.debug("memberOption class : {}", memberOption.getClass());
        assertEquals(false, Hibernate.isInitialized(memberOption), "isProxyObject");
    }
 
}
```

### 2.1.1. @OneToOne - Lazy Loading 검증 실패

그러나 테스트 결과를 보면 Lazy 패치 설정이 동작되지 않는다.

![test-one-to-one-lazy1](/md/img/hibernate/n-plus-one/test-one-to-one-lazy1.png)

- MemberOption 클래스가 Proxy가 아니다.
- MemberOption 필드에 접근하지 않았음에도 MemberOption를 조회 쿼리가 발생한다.

이 부분은 애플리케이션의 성능 저하를 발생시킬 수 있기에, 왜 @OneToOne 관계에서 Lazy 패치 설정이 동작 안됐는지 원인과 해결방법에 대해 인지해야 한다.

### 2.1.2. @OneToOne - Lazy Loading 검증 실패 원인
### 2.1.2.1. Lazy Loading - 객체 그래프 탐색 & Proxy

그렇다면 왜 Lazy 글로벌 패치 설정이 동작하지 않았을까?

JPA에선 객체 그래프 탐색이라는 논리적인 개념이 존재한다. 특정 도메인 객체를 조회할 때 연관 관계를 맺은 각각의 도메인의 글로벌 패치 전략을 판단하다. 이때 Lazy 전략으로 설정된 도메인에 대해선 내부적으로 default 생성자를 통해 Proxy 객체를 생성하고([참고 CGLIB](https://github.com/cglib/cglib/wiki)) 조회할 도메인 객체는 생성된 Proxy 객체를 참조하게 된다.

즉 nullable한 도메인에 대해선 Proxy 객체 생성을 보장할 수 없다. 따라서 Lazy 패치 전략이더라도 내부적으로 객체간 참조를 보장하기 위해 쿼리를 발생시켜 연관된 객체에 데이터를 매핑시키는 동작 방식을 취하게 된다.

이를 토대로 앞서 본 테스트 결과를 분석하면 다음과 같다.

- MemberOption 클래스가 Proxy가 아니다.
    - nullable한 OneToOne 관계라면, Proxy 객체를 감싸지 않고 내부적으로 객체를 반환한다. 단. Collection(*ToMany)은 다른 방식을 취한다.
- MemberOption 필드에 접근하지 않았음에도 MemberOption를 조회 쿼리가 발생한다.
    - 하이버네이트는 객체 그래프 탐색하는 시점에 Lazy로 판단하여 쿼리 생성 시 Member 쿼리만 생성하게 된다. 하지만 MemberOption은 Proxy 객체가 아니므로 MemberOption 쿼리가 발생하여 참조 받을 수 있도록 쿼리를 발생시키게 된다.


### 2.1.3. Lazy Loading - Proxy를 보장하자

결과적으로 Proxy 생성을 보장하기 위해선 non-null 관계가 형성되어야 한다. JPA의 연관관계 관련된 어노테이션에는 `optional`이라는 속성이 존재한다. 다음 속성을 통해 Proxy 생성을 보장하면 된다.

```java
public class Member {
    // ...
    @OneToOne(mappedBy = "member"
         , cascade = CascadeType.ALL
         , fetch = FetchType.LAZY // Lazy loading
         , optional = false) // [1] non-null relationship
    private MemberOption memberOption;
}
```
- optional=true : (default) nullable, 외부 조인
- optional=false : non-null, 내부 조인

![test-one-to-one-lazy2](/md/img/hibernate/n-plus-one/test-one-to-one-lazy2.png)

[1] `optional=false` 속성을 지정해주면 하이버네이트는 내부적으로 MemberOption에 대한 null을 허용하지 않음으로 비로소 lazy loading이 된다.

> 참고) OneToOne lazy-loading 조건 : null을 허용하지 않거나 또는 단방향 관계 설정

## 2.2. 반드시 Lazy 패치 설정을 해야되나?

Lazy 패치에 대해 충분한 이해가 있다면 웬만하면 Lazy 패치로 잡는게 좋지만, 예외적으로 비즈니스에 따라 Eager 패치를 고려할 필요가 있다.

예를 들어 `사용자 정보 변경`라는 사용자 인터페이스에 사용자의 이름, 이메일, 핸드폰 번호와 같은 기본적인 정보와 사용자 계정에 대한 활성화/비활성화 할 수 있는 옵션을 제공한다고 하자.

이런 경우엔 Lazy 보단 Eager 패치가 더 바람직할 수 있다. 테스트 코드 결과를 통해 Lazy 패치의 문제점을 살펴보자.

```java
@Test
void testOneToOneLazy() {
    EntityManager em = getEntityManager();
    Member member = em.find(Member.class, 1L);
    MemberOption memberOption = member.getMemberOption();

    // Proxy 확인
    log.debug("memberOption class : {}", memberOption.getClass());
    assertEquals(false, Hibernate.isInitialized(memberOption), "isProxyObject");
    
    member.setName("gmoon"); // [1] 사용자 이름 변경
    log.debug("사용자 이름 변경");
    
    log.debug("option class : {}", memberOption.getClass());
    memberOption.enabled();
    member.setMemberOption(memberOption); // [2] 계정 활성화
    log.debug("계정 활성화");
}
```
![test-one-to-one-lazy3](/md/img/hibernate/n-plus-one/test-one-to-one-lazy3.png)

1. [1] Member를 조회 쿼리가 발생한다.
2. Lazy 패치 전략으로 MemberOption은 Proxy 객체로 반환된다.
3. [2] `MemberOption.enabled` 필드에 접근하는 순간 MemberOption 조회 쿼리가 발생한다.

[1] Member 조회 쿼리 1번과 [2] MemberOption에 대한 조회 쿼리 1번 즉 2번의 쿼리가 발생하게 된다. 드디어 N+1 문제에 직면하게 되었다.

> OneToOne N+1 문제는 하나의 쿼리가 발생하지만, Collection 연관 관계(*ToMany)에선 대상 도메인의 수 만큼 쿼리가 발생하기에 애플리케이션 성능에 치명적이다.

다음처럼 필수적인 식별 관계의 @OneToOne 글로벌 패치 전략은 LAZY 보단 EAGER로 설정하는 게 오히려 바람직할 수 있다.

## 2.3. @OneToOne - Eager Loading

EAGER 패치 전략에 대해 살펴보자.

```java
@Entity
public class Member {
    //...
    @OneToOne(mappedBy = "member"
            , cascade = CascadeType.ALL
            , optional = false) // [1] non-null relationship
    private MemberOption memberOption;
}
```

![test-one-to-one-eager1](/md/img/hibernate/n-plus-one/test-one-to-one-eager1.png)

Eager 패치를 통해 Member 도메인을 조회할 때 MemberOption 조인되어 하나의 조회 쿼리만 발생됐다. 더이상 Lazy-loading에서 발생하던 N+1 문제가 발생하지 않는다.

그렇다고 Eager 패치로 설정하라는 말은 아니다. Eager 패치는 불필요한 객체 참조로 인해 메모리가 많이 차지한다는 단점이 존재한다.

기본적으로 JPA 컨셉은 Proxy 매커니즘을 지향한다. 또한, 한번 글로벌 패치 설정 정하면 운영 상황에서 바꾸기 어렵다. 따라서 초기에 Lazy 패치로 설정하며 도메인 모델링을 하는 것을 추천한다.

> [Bealdung - Eager/Lazy Loading In Hibernate](https://www.baeldung.com/hibernate-lazy-eager-loading) - The idea of disabling proxies or lazy loading is considered a bad practice in Hibernate. It can result in a lot of data being fetched from a database and stored in a memory, irrespective of the need for it.


## 3. Spring Data JPA - 편리함은 늘 경계해야 된다.

지금까지 글로벌 패치에서 발생하는 N+1 문제에 대해서 살펴봤다.

요즘엔 EntityManager에서 직접 엔티티를 제어하기보다는, 기본적인 기능들을 제공해주는 Spring Data Repository를 사용하는 경우가 더 많다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> { ... }
```

편리함은 늘 경계할 필요가 있다. 마찬가지로 Spring Data JPA를 사용하다 보면 문제가 발생할 수 있다. 예를 들어 Spring Data Repository에서 기본적으로 제공하는 findAll 메서드가 대표적이다. findAll 메서드는 Eager/Lazy 든 N+1 이 발생시키기 때문이다.

```java
@Test
void testSpringDataJpaFindAll_n_plus_one() {
    memberRepository.findAll();
}
```
![test-spring-data-jpa-find-all-n-plus-one1](/md/img/hibernate/n-plus-one/test-spring-data-jpa-find-all-n-plus-one1.png)

findAll 메서드에서 글로벌 패치가 제대로 동작하지 않는 이유는 Spring 내부에서 Criteria를 사용하여 JPQL 쿼리를 생성하게 된다. JPQL을 생성할 때 *Repository&lt;T, ID&gt;에 설정된 엔티티 타입을 ROOT 엔티티로 설정하여 반환한다. 반환된 조회 쿼리는 다음과 같다.

```java
// select m from Member as m 
entityManager.createQuery("select m " 
                          + "from Member as m", Member.class)
                .getResultList();
```
> 참고) @see org.springframework.data.jpa.repository.support.SimpleJpaRepository#getQuery(Specification, Class, Sort)

findAll 메서드로 생성된 JPQL은 대상 엔티티의 모든 필드에 접근하게 하고 엔티티에 지정된 글로벌 패치 설정과 무관하게 필드 값에 값을 채우기 위해 MemberOption 조회 쿼리가 발생하게 된다.

## 4. N+1 해결 방법
## 4.1. JPQL - Fetch Join

엔티티와 함께 조회할 속성을 지정해 줄 수 있도록 JPQL 문법에선 `join fetch` 문법을 사용하여 N+1 문제를 해결할 수 있다.

```java
// JPQL join fetch
em.createQuery("select m "
               + "from Member as m "
               + "inner join fetch m.memberOption mo", Member.class)
        .getResultList();
```
![test-jpql-fetch-join1](/md/img/hibernate/n-plus-one/test-jpql-fetch-join1.png)

## 4.2. Spring Data JPA - @Query

Spring Data JPA에선 @Query 어노테이션을 사용하여 JPQL 문법을 사용할 수 있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m inner join fetch m.memberOption mo")
    List<Member> findAllOfQueryAnnotation();
}
```

그러나 페치 조인 같은 경우엔 중복된 쿼리가 발생한다는 단점이 존재한다.

```text
// 1. 이름 조회 - Member만 조회
select m from Member m where m.name = ?
 
// 2. 이름 조회 - MemberOption과 함께 조회
select m from Member m 
join fetch m.memberOption where m.name = ?
 
// 3. 이름 조회 - Team과 함께 조회
select m from Member m 
join fetch m.team where m.name = ? 
```

다음 JPQL 모두 사용자 이름을 조회하지만, 상황에 따라 각기 다른 JPQL를 사용해야 한다. JPA 2.1에 추가된 엔티티 그래프 기능을 사용한다면 엔티티를 조회하는 시점에 함께 조회할 엔티티를 선택할 수 있다.

## 4.3. 엔티티 그래프 - @EntityGraph & @NamedEntityGraph

엔티티 그래프란 엔티티 조회 시점에 연관된 엔티티들을 함께 조회하는 기능을 의미한다.

> 참고) hibernate version 5.2 이상 or Spring Data JPA 2.0.0.RELEASE 이상

엔티티 그래프를 사용하게 된다면 ROOT 엔티티만 JPQL로 작성하면 된다.

```text
select m from Member m where m.name = ?
```

엔티티 그래프 종류엔 정적/동작에 따라 2 가지로 분류할 수 있다.

1. @NamedEntityGraph(정적)
2. @EntityGraph(동적)

엔티티 그래프 기능을 사용할 엔티티 객체에 @NamedEntityGraph 어노테이션을 설정한다.

```java
@Entity
@NamedEntityGraph(name = "Member.withMemberOption"
        , attributeNodes = { @NamedAttributeNode(value = "memberOption") })
public class Member {

//    ...
    
    @OneToOne(mappedBy = "member", cascade = CascadeType.ALL, optional = false, fetch = FetchType.LAZY)
    private MemberOption memberOption;
}
```
- name : 엔티티 그래프 이름
- attributeNodes : 함께 조회할 속성


### 4.3.1. EntityManager - @NamedEntityGraph

엔티티 객체에 설정된 엔티티 그래프는 EntityManager를 통해 다음과 같이 조회할 수 있다.

```java
EntityManager em = getEntityManager();
EntityGraph graph = em.getEntityGraph("Member.withMemberOption");

// 단일건 조회
Map<String, Object> hints = new HashMap<>();
hints.put("javax.persistence.fetchgraph", graph);
long primaryKey = 1L;
Member member = em.find(Member.class, primaryKey, hints);

Query query = em.createQuery("select m from Member m"
                    + " inner join fetch m.memberOption");
//                      ^-- JPQL은 글로벌 패치를 고려하지 않고 
//                      항상 외부 조인을 사용하기 때문에 inner join fetch 명시한다.
query.setHint("javax.persistence.fetchgraph", graph);
List<Member> list = query.getResultList();
```
> 참고로 `Member -> MemberOption -> ChildEntity` 엔티티 구조에서 Member 엔티티를 조회할 때 MemberOption -> ChildEntity도 엔티티 그래프에 포함하려면 @NamedAttributeNode의 subgraph 속성을 사용하면 된다. 자세한 내용은 본 포스팅에선 다루지 않겠다.

### 4.3.2. EntityManager - @EntityGraph

엔티티에 지정된 @NamedEntityGraph를 사용하지 않고, 조회하는 시점에 @EntityGraph 어노테이션을 사용하여 동적으로 엔티티 그래프 생성할 수도 있다.

```java
EntityManager em = getEntityManager();
EntityGraph<Member> graph = em.createEntityGraph(Member.class); // [1]
graph.addAttributeNodes("memberOption"); // [2]

Map<String, Object> hints = new HashMap<>();
hints.put("javax.persistence.fetchgraph", graph);
Member member = em.find(Member.class, 1L, hints);
```

1. createEntityGraph 메서드를 사용하여 동적 엔티티 그래프 생성
2. addAttributeNodes 메서드를 사용하여 함께 조회할 속성 정의

## 4.4. Spring Data JPA - @EntityGraph & @NamedEntityGraph

Spring Data JPA에선 다음과 같다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @EntityGraph(value = "Member.withMemberOption"
               , attributePaths = "memberOption"
               , type = EntityGraph.EntityGraphType.FETCH)
    Member findByName(String name);

    @EntityGraph(value = "Member.withMemberOption")
    @Query("select m from Member m") // [1] 엔티티 root 설정
    List<Member> findAllOfEntityGraph();
}
```
- value : 엔티티 그래프 이름 (`@NamedEntityGraph(name="Member.withMemberOption")`)
- attributePaths : 함께 조회할 속성
- type : 엔티티 그래프 힌트
    - EntityGraph.EntityGraphType.FETCH (default)
        - javax.persistence.fetchgraph : 엔티티 그래프에서 정의한 속성만 함께 조회
    - EntityGraph.EntityGraphType.LOAD
        - javax.persistence.loadgraph : 엔티티 그래프에서 정의한 속성 + EAGER 패치로 설정된 연관관계도 포함

엔티티 그래프는 항상 조회하는 엔티티의 ROOT에서 시작해야 한다. 이점은 Spring Data JPA에서 엔티티 그래프 기능을 사용할 때 유의해야 한다.

Spring Data JPA는 내부적으로 result type이 단일 엔티티인 경우에만 ROOT 엔티티를 추적할 수 있고, 추적된 ROOT 엔티티 기준으로 엔티티 그래프 기능을 적용해준다. Collection과 같은 result type에선 ROOT 엔티티를 추적할 수 없으므로 [1] @Query 어노테이션을 사용하여 ROOT 엔티티를 지정해줘야 한다.

![test-spring-data-jpa-entity-graph-non-root](/md/img/hibernate/n-plus-one/test-spring-data-jpa-entity-graph-non-root.png)

> 참고) Spring Data JPA에선 @Fetch 모드를 무시한다. 이와 관련된 내용은 아래 링크를 참고하자. <br/>
> I think that Spring Data ignores the FetchMode. I always use the @NamedEntityGraph and @EntityGraph annotations when working with Spring Data JPA... - [stackoverflow - How does the FetchMode work in Spring Data JPA](https://stackoverflow.com/questions/29602386/how-does-the-fetchmode-work-in-spring-data-jpa)

## 4.5. QueryDsl - fetchJoin

QueryDsl의 fetchJoin에 대해 살펴보자.

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    public List<Member> getList() {
        QMember qMember = QMember.member;
        return jpaQueryFactory.select(qMember)
                .from(qMember)
                .fetch();
    }
}
```
![test-query-dsl-n-plus-one1](/md/img/hibernate/n-plus-one/test-query-dsl-n-plus-one1.png)

다음 코드에서도 마찬가지로 N+1 문제가 발생한다. 이러한 이유엔 QueryDsl은 SQL, JPQL을 코드로 작성할 수 있도록 도와주는 빌더 API이기 때문이다.

결국, JPQL를 기반으로 하므로, QueryDsl에서도 fetchJoin으로 함께 조회할 속성을 정의해줘야 한다.

```java
return jpaQueryFactory.select(qMember)
        .from(qMember)
        .innerJoin(qMember.memberOption, qMemberOption).fetchJoin()
        .fetch();
```

QueryDsl에선 join 절 뒤에 fetchJoin() 메서드를 지정해주면 된다.

### 4.5.1. QueryDsl 주의 사항 - Cross Join

아래 코드처럼 join을 명시하지 않고 qMember.memberOption.enabled 처럼 작성하는 경우, MemberOption을 크로스 조인하여 실행된다.

````java
@Override
public List<MemberVO.Data> getList() {
    QMember qMember = QMember.member;
    
    return jpaQueryFactory
            .select(new QMemberVO_Data(qMember.id
                                     , qMember.name
                                     , qMember.memberOption.enabled))
            .from(qMember)
            .fetch();
}
````
![test-query-dsl-cross-join](/md/img/hibernate/n-plus-one/test-query-dsl-cross-join.png)

### 4.6. QueryDsl - @QueryProjection

N+1과 무관하게 되도록이면 비즈니스 로직에 꼭 필요한 데이터들만 projection을 사용하여 조회할 수 있도록 하는것이 바람직하다.

projection을 지정하여 조회하는 방식은 대표적으로 3 가지 방식을 주로 사용한다.

- @QueryProjection
- Projections.bean
- Projections.constructor

필자는 Projections을 사용하는 방식보다는 @QueryProjection 어노테이션을 사용하는 방식을 더 추천한다.

> 1. Projections.bean : setter 필요 </br>
> 2. Projections.constructor : 생성자 파라미터 순서가 변경되면 queryDsl도 같이 변경

```java
public class MemberVO {

    @QueryProjection
    public static class Data {
        private String id;
        
        private String name;
        
        private boolean enabled;
    }
}
```
```java
jpaQueryFactory.select(new QMemberVO_Data(qMember.id
                                        , qMember.name
                                        , qMemberOption.enabled))
        .from(qMember)
        .innerJoin(qMember.memberOption, qMemberOption)
        .fetch();
```

### 4.6.1. QueryDsl Projection 사용시 주의 사항

Projection과 fetchJoin은 같이 사용할 수 없다.

```java
return jpaQueryFactory
        .select(new QMemberVO_Data(qMember.id
                                 , qMember.name
                                 , qMemberOption.enabled))
                .from(qMember)
                .innerJoin(qMember.memberOption, qMemberOption).fetchJoin()
                .fetch();
```
![test-query-dsl-projection-with-fetchjoin](/md/img/hibernate/n-plus-one/test-query-dsl-projection-with-fetchjoin.png)
```text
Caused by: org.hibernate.QueryException: query specified join fetching, but the owner of the fetched association was not present in the select list [FromElement{explicit,not a collection join,fetch join,fetch non-lazy properties,
```

fetchJoin은 엔티티 객체의 엔티티 그래프를 참조할때 사용한다. 즉 VO 객체에선 불가능하다. Projection을 사용하여 VO를 반환하는 구조라면 fetchJoin을 제거하여 작성하면된다.

```java
return jpaQueryFactory
        .select(new QMemberVO_Data(qMember.id
                                , qMember.name
                                , qMemberOption.enabled))
    .from(qMember)
    .innerJoin(qMember.memberOption, qMemberOption)
    .fetch();
```

---

## 마무리

코로나로 인해 회사가 커지면서 운영 중인 서버 사용량이 기하급수적으로 늘어났다.

2020년도엔 서버 장애와 긴급 이슈에 대응하며 서비스 안정화를 위해 바쁘게 보냈던 거 같다. 특히 RabbitMQ, DynamoDb 등 트래픽 부하 분산을 위한 인프라가 구축되어가면서 다행히도 안정된 수준에서 운영할 수 있었다. 이 과정에서 병목 현상이 일어나는 많은 로직을 개선했었는데, 그중 대표적으로 일대일 N+1 문제로 인해 개선했던 점이 기억에 남는다. (~~종종 타 부서에선 JPA는 많은 쿼리를 발생시킨다며 JPA에 대한 선입견과 부정적인 의견들도 있었다.~~)

일반적으로 N+1 문제는 *ToMany 관계에서 주로 일어난다고 생각하는 것 같다. 따라서 *ToMany 관계에선 주의 깊게 로직을 구현하지만 비교적 OneToOne 관계에서도 N+1 문제가 발생한다는 점을 안일하게 생각했던 건 아니었는지... 일대일 연관관계와 N+1 문제에 대해 정리한 이유이기도 하다.

끝으로 본 포스팅 내용을 정리하며 마무리하겠다. Lazy 패치는 메모리 성능과 특정 도메인을 조회 시 불필요한 컬럼이 포함하지 않기 때문에 비즈니스를 파악하기 수월하다는 장점이 존재하지만, 자칫 잘못 사용하는 경우 LazyInitializationException 와 N+1 문제를 직면하여 오히려 성능상의 이슈가 발생할 수 있다. 또한, JPQL의 N+1 문제와 해결 방법에 대해 인지하고 N+1 문제가 발생한다면 각각 상황에 맞게 비즈니스 로직을 개선해야 한다. 언제나 N+1 문제가 발생할 수 있다는 점을 인지하며 단순한 로직을 구현할 때도 여러 측면에서 생각하며 개발해 볼 필요가 있다.

- 글로벌 패치 조인 : Proxy
- OneToOne Lazy-loading
- 엔티티 그래프 : 객체 그래프 탐색
- JPQL : fetch join

## TODO

- JPA 튜닝
- JPA hint
- fetch join 문제점
    - fetch join - pageable
    - [bottom-to-top-blog - fetch join 과 limit 을 같이 쓸 때 주의하자.](https://bottom-to-top.tistory.com/45)

## 참고

- [Baeldung - JPA OneToOne](https://www.baeldung.com/jpa-one-to-one)
- [Baeldung - EAGER/LAZY Loading In Hibernate](https://www.baeldung.com/hibernate-LAZY-EAGER-loading)
- [Spring Data JPA DOC - Entity Graph](https://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/#jpa.entity-graph)
- [Spring Data JPA DOC - Custom Implementations for Spring Data Repositories](https://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/#repositories.custom-implementations)
- [QueryDsl - Reference](http://www.querydsl.com/static/querydsl/4.4.0/reference/html_single/)
- [QueryDsl - JPAQueryBase](http://www.querydsl.com/static/querydsl/4.0.4/apidocs/com/querydsl/jpa/JPAQueryBase.html)
- [Hibernate5 - User Giude](https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html)
- [Hibernate5 - Reference](https://docs.jboss.org/hibernate/stable/orm/javadocs/)
- [Hibernate3 - QueryHql](https://docs.jboss.org/hibernate/core/3.3/reference/en/html/queryhql.html)
- [Wiki - JPA OneToOne](https://kwonnam.pe.kr/wiki/java/jpa/one-to-one)
- [Wiki - Primary Keys through OneToOne and ManyToOne Relationships](https://en.wikibooks.org/wiki/Java_Persistence/Identity_and_Sequencing#Primary_Keys_through_OneToOne_and_ManyToOne_Relationships)
- [blog - Difference between @JoinColumn and @PrimaryKeyJoinColumn](https://thorben-janssen.com/hibernate-tip-difference-between-joincolumn-and-primarykeyjoincolumn/)
