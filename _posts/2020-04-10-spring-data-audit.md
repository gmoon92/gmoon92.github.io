---
layout: post
title: "Spring Data JPA의 이력 유형 데이터 관리"
tags: [Spring, JPA, Hibernate, Spring Data JPA, Audited]
categories: [Spring, Hibernate, Audited]
subtitle: "엔티티의 이력 유형 데이터 모델링"
feature-img: "md/img/thumbnail/hibernate-envers.png"
thumbnail: "md/img/thumbnail/hibernate-envers.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily

priority: 1.0
---

<!--more-->

# 엔티티의 이력 유형 데이터 모델링

---

### 들어가기전

 데이터베이스에서 이력 데이터 모델링은 실무에서 중요한 부분이라 할 수 있습니다.

![img](/md/img/hibernate/audit/member_table.png)

- 생성자
- 생성 일자
- 수정자
- 수정 일자

위의 테이블 모델링을 살펴보시면 낯설지 않게 느끼실 수 있습니다. 이러한 테이블 모델링은 기본적이면서도 중요한 부분이라 할 수 있습니다. 개발자는 생성자, 생성 일자, 수정자, 수정 일자 컬럼의 데이터를 통해, 누가 언제 테이블의 데이터를 저장하고 수정 또는 삭제 했는지에 대해 알 수 있기 때문입니다. 이러한 칼럼의 유형을 이력유형이라 하는데, 실무에서 이력 유형 컬럼들을 고려하지 않은 테이블 모델링을 마주하게 된다면 데이터를 누가 삽입/수정/삭제했는지에 대해 추적하는데 어려움이 따릅니다.
 
만약 Spring과 JPA를 사용하는 프로젝트라면 이력 유형 데이터들을 쉽게 관리할 수 있습니다. 본 포스팅에선 **엔티티의 이력유형 데이터 모델링**을 어떻게 구성하는 방법을 소개해드리겠습니다. 전체 코드는 [GitHub](https://github.com/gmoon92/github-blog-codes/tree/master/spring-data-audit)에서 확인해볼 수 있습니다.

- Spring Boot 2.2.2.RELEASE
- Hibernate 5.4.9 Final

### 학습 목표
 
- Spring Data Audit 기능
- 엔티티의 이력 유형 데이터 모델링

### 1. Spring Data JPA의 Audit 기능

Spring Data JPA에선 이력 유형 데이터를 쉽게 관리할 수 있도록 @CreatedBy, @CreatedDate, @LastModifiedBy, @LastModifiedDate 이력 유형 애노테이션을 제공하고 있습니다. 애노테이션의 네이밍 그대로 생성자, 생성 일자, 마지막 수정자, 마지막 수정 일자를 의미합니다.

- org.springframework.data.annotation.CreatedBy
- org.springframework.data.annotation.CreatedDate
- org.springframework.data.annotation.LastModifiedBy
- org.springframework.data.annotation.LastModifiedDate

이력 유형 애노테이션의 사용법은 간단합니다.

1. 이력 유형 필드에 애노테이션 명시
2. 이력 유형 리스너 활성화

### 1.1. 이력 유형 필드에 애노테이션 명시

우선 엔티티를 구성할 때, 이력 유형 필드에 애노테이션을 명시해줍니다.

``` java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String userId;

    private Stirng password;

    @CreatedBy
    private Long createdBy;

    @CreatedDate
    private LocalDateTime createdDt;

    @LastModifiedBy
    private Long modifiedBy;

    @LastModifiedDate
    private LocalDateTime modifiedDt;
}
```

그다음으로 이력 유형 애노테이션을 활성화하기 위해 부가적인 작업이 필요합니다. 아래 테스트 코드 결과를 보면 이력 유형 애노테이션이 활성화되지 않아 데이터들이 null로 저장되는 걸 확인할 수 있습니다.

``` java
@Slf4j
@DataJpaTest
@TestConstructor(autowireMode = ALL)
@RequiredArgsConstructor
class MemberRepositoryTest {

    private final memberRepository;

    @Test
    void saveAutoAuditedFields() {
        Member member = memberRepository.save(Member.builder()
                .userId("gmoon")
                .password("111111")
                .build());

        assertAll("auto init audited fields",
                () -> assertNotNull(member.getCreatedBy()),
                () -> assertNotNull(member.getCreatedDt()),
                () -> assertNotNull(member.getModifiedBy()),
                () -> assertNotNull(member.getModifiedDt())
        );
    }
}
```

![img](/md/img/hibernate/audit/test-result1.png)

### 1.2. Audit Listener 등록 및 활성화
### @EntityListeners, @EnableJpaAuditing

Spring Data JPA 애노테이션을 활성화하기 위해선, JPA 표준 규격 패키지에 포함된 javax.persistence.EntityListeners 애노테이션이 필요합니다.

``` java
package javax.persistence;

@Target({TYPE}) 
@Retention(RUNTIME)
public @interface EntityListeners {

    Class[] value();
}
```

@EntityListeners 애노테이션에 리스너 클래스를 지정하면 엔티티 상태가 변화할 때 지정된 리스너로 콜백받는 용도로 사용되며, @Entity 또는 @MappedSuperclass 애노테이션이 부착된 클래스에서 사용할 수 있습니다. 별도의 리스너 클래스를 정의하여 사용해도 되지만, 기본적으로 Spring Data에서 제공하는 AuditingEntityListener 클래스를 지정해주면 됩니다.
 
 AuditingEntityListener 클래스는 내부적으로 엔티티의 콜백 이벤트 애노테이션인 @PrePersist, @PreUpdate를 사용하여, 영속 상태 이전에 값을 삽입하거나 데이터베이스의 업데이트 질의문을 수행하기 이전에 이력 유형 데이터를 관리해줍니다. 간단히 이벤트 애노테이션들을 소개해드리자면 다음과 같습니다.

|애노테이션|설명|
|-------|---|
|@PostLoad|엔티티가 영속성 컨텍스트에서 조회된 직후 또는 refresh된 경우에 호출된다.|
|@PrePersist|EntityManager의 persist 또는 cascaded 전에 호출된다. 참고로 식별자 생성 전략을 사용한 경우 엔티티에 식별자는 존재하지 않는다.|
|@PostPersist|EntityManager의 persist 또는 cascaded 후에 호출되며 식별자가 항상 존재한다. 참고로 식별자 생성 전략이 IDENTITY면 식별자를 생성하기 위해 persist를 호출한 직후에 이벤트가 호출된다.|
|@PreRemove|EntityManager의 remove 또는 cascaded 전에 호출되며 orphanRemoval에 대해서는 flush나 commit할 시점에도 호출된다.|
|@PostRemove|EntityManager의 remove 또는 cascaded 후에 호출된다.|
|@PreUpdate|데이터베이스 UPDATE 쿼리가 실행되기 전에 호출된다.|
|@PostUpdate|데이터베이스 UPDATE 쿼리가 실행되기 후에 호출된다.|

본론으로 돌아와서 회원 클래스에 AuditingEntityListener 클래스를 지정해줍시다.

``` java
@Entity
@EntityListeners(value = { AuditingEntityListener.class })
public class Member { ... }
``` 

아래 Bean 구성 클래스에 다음과 같이 @EnableJpaAuditing 애노테이션을 명시하여 리스너를 활성화해줍니다. 이처럼 리스너를 활성화하게 되면, @CreatedDate, @LastModifiedDate 애노테이션이 지정된 날짜 이력 유형 필드들은 Spring Data에 의해 자동으로 관리됩니다.

``` java
@Configuration
@EnableJpaAuditing // Audit 리스너 활성화
public class AuditedConfig { }
```

마지막으로 @CreatedBy, @LastModifiedBy 애노테이션을 활성화 해주기 위해선, AuditorAware 인터페이스를 구현한 클래스를 Bean으로 등록해주면 됩니다.
 
``` java
@Configuration
@EnableJpaAuditing
public class AuditedConfig {

    @Bean
    public AuditorAware<Long> auditorAware() {
        return () -> Optional.ofNullable(0L);
    }
}
```

물론 Spring Security를 사용한다면, 시큐리티 컨텍스트에서 로그인한 사용자 정보를 반환해주는 코드를 작성해주면 되지만, 위 코드는 예제 코드임으로 엔티티 ID를 0L로 지정해주었습니다. 이제 마지막으로 테스트 코드를 실행하여 정상적으로 이력 유형 필드의 데이터들이 관리되는지 확인해보겠습니다.

``` java
@DataJpaTest
@TestConstructor(autowireMode = ALL)
@RequiredArgsConstructor
class MemberRepositoryTest {

    private final MemberRepository memberRepository;

    @Test
    void saveAutoAuditedFields() {
        Member member = memberRepository.save(Member.builder()
                .userId("gmoon")
                .password("111111")
                .build());

        assertAll("auto init audited fields",
                () -> assertNotNull(member.getCreatedBy()),
                () -> assertNotNull(member.getCreatedDt()),
                () -> assertNotNull(member.getModifiedBy()),
                () -> assertNotNull(member.getModifiedDt())
        );
    }

    @TestConfiguration
    @EnableJpaAuditing
    static class TestAuditedConfig {

        @Bean
        public AuditorAware<Long> auditorAware() {
            return () -> Optional.ofNullable(0L);
        }
    }
}
```
![img](/md/img/hibernate/audit/test-result2.png)

### 1.3. 이력 유형 데이터 추상화
### @MappedSuperclass 애노테이션 활용

앞서 본 예제 코드처럼 엔티티 클래스에 이력 유형 애노테이션들을 명시하여 관리할 수 있습니다.

하지만 부가적으로 여러 도메인에 공통된 이력 유형 필드에 대해 감사(audit)가 필요하다면 직접적인 클래스에 필드로 명시하는 것보다는 공통된 부분인 이력 유형 필드를 추상화하여 관리해야 합니다. 이럴 때 JPA 표준 패키지에 포함된 @MappedSuperclass 애노테이션을 활용한다면, 코드 관리 관점에서도 이력 데이터를 더욱 쉽게 관리할 수 있습니다.

우선 기존 회원 클래스에 존재했던 이력 유형 필드들을 BaseTraceEntity 추상화 클래스로 모듈화해줍니다.

``` java
@Getter
@MappedSuperclass
@EntityListeners(value = { AuditingEntityListener.class })
public abstract class BaseTraceEntity {

    @CreatedBy
    private Long createdBy;

    @CreatedDate
    private LocalDateTime createdDt;

    @LastModifiedBy
    private Long modifiedBy;

    @LastModifiedDate
    private LocalDateTime modifiedDt;

}
```

앞서 설명했던 것처럼 @EntityListeners 애노테이션은 @Entity 클래스뿐만 아니라 @MappedSuperclass 클래스에도 지정할 수 있으므로, BaseTraceEntity 클래스에 리스너를 명시해줍니다. 

마지막으로 회원 클래스에선 생성자, 생성 일자, 수정자, 수정 일자 필드를 제거하고, 구성한 BaseTraceEntity 클래스를 상속만 해주면 됩니다.

``` java
@Entity
public class Member extends BaseTraceEntity{

    @Id
    @GeneratedValue
    private Long id;

    private String userId;

    private String password;
}
```

### 마무리

다음과 같이 Spring Data와 JPA를 활용하여 이력 유형 데이터를 모델링하는 방법에 대해 소개해드렸습니다.

1. 이력 유형 애노테이션 활용 : @MappedSuperclass를 활용하여 추상화
2. 리스너 활성화 : @EntityListeners, @EnableJpaAuditing
3. AuditorAware 인터페이스 구현 및 Bean 등록

이처럼 간단한 방법을 통해 엔티티를 누가 저장했는지, 누가 수정했는지에 대해 추적할 수 있습니다. 단 엔티티의 삭제 이력 데이터에 대해선 다음과 같은 방법으론 추적할 수 없습니다. 다음 포스팅에선 삭제된 이력 데이터를 관리하는 방법을 소개해드리겠습니다.

### 참고

- [jboss-docs-hibernate-4.1-Envers](https://docs.jboss.org/hibernate/core/4.1/devguide/en-US/html/ch15.html#envers-tracking-modified-entities-queries)
- [Spring DOC - Spring data](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/annotation/package-frame.html)
- [Baeldung - Spring data annotations](https://www.baeldung.com/spring-data-annotations)
- [Hibernate-core - listeners](https://docs.jboss.org/hibernate/core/4.0/hem/en-US/html/listeners.html)