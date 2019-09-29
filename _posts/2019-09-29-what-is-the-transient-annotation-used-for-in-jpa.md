---
layout: post
title: "JPA에서 @Transient 애노테이션이 존재하는 이유"
tags: [JPA, Hibernate, Annotation, @Transient]
categories: [JPA]
subtitle: "@Transient 제대로 이해하고 사용하자"
feature-img: "md/img/thumbnail/why-used-jpa.png"
thumbnail: "md/img/thumbnail/why-used-jpa.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# @Transient
# 해당 데이터를 테이블의 컬럼과 매핑 시키지 않는다.

---

### 들어가기전

@Transient는 엔티티 객체의 데이터와 테이블의 컬럼(column)과 매핑하고 있는 관계를 제외하기 위해 사용합니다. 예를 들기 위해 회원(Member) 엔티티를 간단히 구성해봅시다.

``` java
@Entity
public class Member{
    @Id
    private Long id; // PK
    private String userId; // 사용자 아이디
    private String password; // 비밀번호
    private String confirmPassword; // 비밀번호 재입력
}
```

다음 엔티티 객체에서 **`confirmPassword`**는 회원가입 화면에서 흔히 볼 수 있는 **`비밀번호 재입력`** 데이터 필드입니다.

이 필드는 비밀번호를 입력하고 재차 비밀번호를 제대로 입력했는지 확인하는 용도로 사용됩니다. 아무래도 **`비밀번호 재입력`** 필드는 비즈니스 로직에서만 필요한 데이터일 뿐, 굳이 회원 테이블에서 컬럼으로 구성하여 **<U>관리할 필요가 없는 데이터</U>**입니다.

이럴 때 아래 코드처럼 confirmPassword 필드에 [1]@Transient 애노테이션을 선언해주면 됩니다.

``` java
@Entity
public class Member{
    @Id
    private Long id;
    private String userId;
    private String password;
    javax.persistence.@Transient // [1] @Transient 선언
    private String confirmPassword; // 비밀번호 재입력 매핑 제외
}
```

하지만 이러한 컬럼 매핑 레퍼런스 애노테이션들은 **자칫 잘못 이해하고 사용한다면 문제가 될 수 있습니다.** "설마 문제가 될까?" 라는 분들을 위해, **<U>다음 코드에서 문제가 되는 부분을 찾아봅시다.</U>**

> 참고로 컬럼 매핑 레퍼런스 애노테이션에는 @Column, @Enumerate, @Temporal ... 등, javax.persistence 패키지에 포함된  JPA의 표준 애노테이션이 존재합니다.

``` java
@Entity
public class Member{
    @Id
    private Long id;
    private String userId;
    private String password;
    private String confirmPassword;
    
    @Transient
    public String getComfirmPassword(){ return this.confirmPassword; }
}
```

혹시 문제가 된 부분을 알아보셨나요? 

바로 @Transient 애노테이션이 선언된 위치입니다.

``` java
@Transient // <-- 문제가 되는 부분
public String getComfirmPassword(){ return this.confirmPassword; }
```

그렇다면 도대체 왜, 다음 애노테이션의 위치가 어떤 문제가 되는 걸까요? 아니면 필드가 아닌 메서드에 애노테이션을 선언되었기 때문일까요?

본 포스팅에선 다음 의문을 해결하기 위해, @Transient 애노테이션을 사용함에서 주의해야 할 사항을 소개하려 합니다. 더불어 문제의 원인에 대한 이해를 돕기 위해 JPA의 개념과 함께 간단한 예제 코드로 구성하였습니다.

### 학습 목표

1. @Transient 애노테이션의 이해
2. @Transient 애노테이션의 사용법
3. 두 가지 엘리멘트 타입을 지원하는 이유
	- ElementType.METHOD
	- ElementType.FIELD
4. JPA의 엔티티 영속 상태 접근 방식에 대한 이해
	- 프로퍼티 접근 방식 (getter/setter Method)
	- 필드 접근 방식 (Instance Fields)

### 1. @Transient 이해하기 

@Transient는 JPA의 표준이라 할 수 있는 **javax.persistence** 패키지에 포함되어있는 컬럼 매핑 레퍼런스 애노테이션입니다. 또한, 앞서 소개해드렸던 것처럼 @Transient는 **컬럼를 제외하기 위해 사용합니다.**

틀린 말은 아니지만, 이 애노테이션을 제대로 이해하기 위해선, 단순히 ~~"컬럼을 제외한다."~~ 라기보단 **영속 대상에서 제외**시키기 위해 사용한다고 이해하셔야 합니다.

``` java
package javax.persistence;
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Transient {}
```

- ElementType.METHOD
- ElementType.FILED

다음 코드를 보면 @Transient는 메서드와 필드에 선언할 수 있는 애노테이션입니다. 또한, @Entity 클래스뿐만 아니라 @MappedSuperclass, @Embeddable 클래스의 필드나 getter 메서드에 선언할 수 있습니다.

>Specifies that the property or field is not persistent. It is used to annotate a property or field of an entity class, mapped superclass, or embeddable class. - [Oracle Doc - @Transient](https://docs.oracle.com/javaee/7/api/javax/persistence/Transient.html)

``` java
@Transient // <-- 문제가 되는 부분
public String getComfirmPassword(){ return this.confirmPassword; }
```

**<U>하지만 문제가 된다던 코드</U>** 역시 getter 메서드에 선언되어있었습니다. 앞서 설명대로라면 메서드에 선언된 다음 코드는 문제가 되지 않아야만 합니다. 눈치챈 분들도 있으시겠지만, <u>다음 문제의 원인은 getter 메서드에 선언되었기 때문이 아닙니다.</u> 아직까지 문제의 원인을 모르시는 분이라면, 거두절미하고 **"영속 대상에서 제외한다"**는 의미를 다시 생각해보셔야 합니다.

### 1.1. 영속 대상에서 제외

_영속 대상에서 제외시키기 위해 사용되는 애노테이션_

무엇보다 영속(persistence)이라는 개념은 JPA의 가장 근간이 되는 개념입니다. 다시 말해 JPA에선 영속성 컨텍스트(Persistence context)라는 논리적인 패러타임의 구현체라 할 수 있는 엔티티 매니저(Entity Manager)가 존재하고, 이 엔티티 매니저에서 @Entity 클래스의 객체를 관리하게 됩니다. 

![D-Zone : JPA Entity Lifecycle](https://i0.wp.com/www.javabullets.com/wp-content/uploads/2017/08/entityManager_javabullets.png?w=1357&ssl=1)

다음 다이어그램을 보면 엔티티 객체의 상태가 영속 상태(managed, persistent state)일 때, 비로소 엔티티 매니저에 의해 관리됩니다. 영속 상태의 엔티티 객체는 엔티티 매니저에 의해 **`[1]`**변화에 대한 자동 감지(Dirty Checking), **`[2]`**CRUD SQL 자동 생성 작업 및 그외 일련의 모든 JPA의 내부적인 동작 프로세스에서 활용됩니다.

하지만 영속 대상에서 제외된다면, 더는 해당 필드나 메서드는 엔티티 매니저의 관리 대상에서 제외됨을 의미합니다. 즉 해당 필드에 대해 @Transient 애노테이션을 선언하게 되면 앞서 설명한 **`[1,2]` 작업들을 수행하지 않습니다.** 이러한 결과를 토대로 **"테이블의 컬럼과 매핑을 하지 않는다."**라고 이해하셔도 무방합니다.

> 엔티티 라이프 사이클에 대한 자세한 개념은 다음 링크를 참조해주시기 바랍니다.
> - [https://dzone.com/articles/jpa-entity-lifecycle](https://dzone.com/articles/jpa-entity-lifecycle)
> - [https://www.baeldung.com/hibernate-entity-lifecycle](https://www.baeldung.com/hibernate-entity-lifecycle)

### 2. 간단한 예시와 사용법

여기까지 영속에 대한 개념을 간단히 살펴보았습니다.

다음으로 @Transient 사용법과 이에 대한 활용법에 대해 생각해봅시다.

``` java
@Entity
public class Product {
    @Id
    private Long id;
    private String name;
    private BigDecimal price;
    private boolean isEvent;
    // ^-- 영속 제외 대상
    
    public void runEventProcess(){
        if(isEvent){
            // ... 이벤트 로직 수행
        }
    }
}
```

다음과 같이 상품 엔티티 객체가 존재한다고 가정해봅시다.

여기서 살펴봐야 할 점은 `isEvent`라는 필드입니다. 이 필드는 Hooking 목적으로 특정 날짜나 시간이 되면 활성화(true)되어, 상품 가격에 대한 할인율 적용과 같은 상품에 대한 이벤트 로직을 적용 시키기 위함입니다.

이처럼 특정 필드에 대해 클래스에서만 사용되고, 테이블 컬럼으로 관리하고 싶지 않을 경우가 있습니다. 하지만 JPA는 @Entity 클래스에 포함된 모든 필드에 대해 테이블의 컬럼과 자동으로 매핑 시키는 작업을 수행해주기 때문에, 다음과 같은 엔티티에 대한 로그를 살펴보실 수 있습니다.

``` sql
Hibernate: 
    create table product (
       id bigint not null,
       is_event boolean not null, <-- 영속 제외 대상
       name varchar(255),
       price decimal(19,2),
       primary key (id)
    )
```

### 2.1. @Transient 두 가지 방식

@Transient 애노테이션은 두 가지 방식을 통해 선언할 수 있도록 제공하고 있습니다.

- ElementType.METHOD
- ElementType.FIELD

### 2.1.1. Field 방식

첫 번째로 필드 방식은 영속 대상에서 제외하고 싶은 isEvent 필드에 @Transient 애노테이션을 선언시키면 됩니다.

``` java
@Entity
public class Product{
    @Id
    private Long id;
    private String name;
    private BigDecimal price;
    @Transient
    private boolean isEvent;
    // ^-- 해당 필드 영속 제외 대상
}
```

``` sql
Hibernate: 
    create table product (
       id bigint not null,
       name varchar(255),
       price decimal(19,2),
       primary key (id)
    )
```

다음 로그를 보시면 JPA의 DDL 자동 생성 과정에서 **<U>isEvent 컬럼을 제외</U>**하고 Product 테이블을 구성하는걸 확인할 수 있습니다. 또한, JPA에 의해 자동으로 생성되었던 SELECT/UPDATE/INSERT 쿼리문에서도 해당 isEvent 컬럼 자체가 제외되어 수행됩니다.

### 2.1.2. Method 방식

이 외에도 @Transient는 필드에 적용하는 방법 외에도 메서드에도 선언시킬 수 있습니다.

``` java
@Entity
public class Product{
	private Long id;
	private String isEvent;
	
	@Id @GeneratedValue
	public Long getId(){ return this.id; }
	public void setId(Long id){ this.id = id; }
	// ^-- @GeneratedValue는 JPA의 내부적인 프로세스에 의해
	//     setter 메서드를 통해 데이터를 셋팅하기 때문에 구성함
	@Transient // <-- 해당 메서드 영속 제외 대상
	public String getIsEventProduct(){ return this.isEvent; }
}
```

다음 코드는 필드 방식과 마찬가지로 같은 결과를 수행하게 됩니다. **주의할 점은** setter 메서드가 아닌, **getter 메서드에 애노테이션을 선언**해줘야 합니다.

### 3. 메서드/필드 방식을 지원하는 JPA 애노테이션의 주의사항

지금까지 @Transient의 메서드/필드 방식의 사용법과 JPA의 영속 상태의 엔티티에 대한 개념을 간단히 알아보았습니다. 하지만 앞서 작성된 글로는 처음에 제시하였던 문제의 원인에 대한 답을 찾을 수 없을 것입니다.

무엇보다 본 포스팅을 작성한 가장 큰 이유는 실무에서 메서드에 선언된 @Transient 애노테이션을 보았기 때문이었습니다. 앞서 처음 문제가 된 코드 역시 당시 상황을 재현하기 위해 구현한 샘플 코드였고 다음 코드를 통해 문제에 대한 원인을 본격적으로 파헤쳐보도록 하겠습니다.

### 3.1. 개발 의도와는 다르게 동작하는 JPA

문제가 되는 다음 코드에서 개발자의 의도는 다음과 같습니다.

``` java
@Entity
public class Member{
	@Id
	private Long id;
	private String userId;
	private String password;
	private String confirmPassword;
	
	@Transient // <-- 문제가 되는 부분
	public String getComfirmPassword(){ return this.confirmPassword; }
}
```

- confirmPassword 필드를 영속 대상에서 제외한다.
- getComfirmPassword() 메서드에 @Transient 애노테이션 선언
- 기대하는 결과 → 테이블 컬럼 생성 및 CRUD SQL문 대상 컬럼 제외

하지만 개발자의 의도와는 달리, 아래 로그를 보면 **실제 코드는 다르게 동작됩니다.**

``` sql
Hibernate: 
    create table member (
       id bigint not null,
       user_id varchar(255),
       password varchar(255),
       confirm_password varchar(255), <-- 하지만 영속 대상에서 제외되지 않음
       primary key (id)
    )
```

분명 @Transient 애노테이션은 필드와 메서드에 선언할 수 있는 애노테이션임은 분명한데, 왜 이러한 결과가 나오게 된 걸까요? 이러한 결과를 이해하기 위해선,  JPA가 엔티티 객체에 접근하는 방식에 대해 이해해야 합니다.

### 3.2. 문제의 원인 파악 - 엔티티 접근 방식에 대한 이해

JPA 스펙에 따르자면, JPA는 **두 가지 방식**을 통해 영속 상태(managed, persistent state)인 엔티티 객체의 데이터에 접근할 수 있습니다.

1. 프로퍼티 방식 (getter/setter Method, JavaBeans 스타일 Property)
2. 필드 방식 (Instance Fileds)

> - [JPA 1.0(JSR 220)](https://jcp.org/en/jsr/detail?id=220)
> - [JPA 2.0(JSR 317)](https://jcp.org/en/jsr/detail?id=317)
> - [JPA 2.1(JSR 338)](https://jcp.org/en/jsr/detail?id=338)

일반적으론 아래 코드처럼 엔티티 객체를 개발할 때 [1] 필드 방식을 주로 사용하겠지만, 상황에 따라 [2] 메서드 방식을 선택하여 개발할 수 있습니다. 이는 JPA가 [1, 2] 두 가지 방식을 통해 엔티티 객체의 데이터에 접근할 수 있도록 지원하고 있기 때문입니다.

``` java
// [1] 필드 방식
@Entity
public class Member{
	@Id
	private Long id;
	private String name;
	...
}

// [2] 메서드 방식
@Entity
public class Member{
	private Long id;
	private String name;
	
	@Id
	public Long getId(){return this.id;}
	public void setId(Long id){this.id = id;}
	
	public String getName(){return this.name;}
	public void setName(String name){this.name = name;}
}
```

_@Target({ElementType.METHOD, ElementType.FIELD})_

따라서 개발자가 JPA의 두 가지 접근 방식을 중 선택하여 개발할 수 있도록, **필드 레벨에 선언시킬 수 있는 모든 JPA의 애노테이션**들은 기본적으로 Property(getter/setter method) 방식을 지원하기 위해 **메서드 레벨을 지원**하고 있습니다. 

여기서 중요한 핵심은 설계 방식에 따라 JPA는 엔티티 객체의  **접근 방식이 다르게 결정된다는 것입니다.** 기본적으로 엔티티 매니저의 1차 캐시는 Map<@Id, @Entity> 형태로 설계되어 key에 해당하는 @Id와 value에 해당하는 엔티티 객체를 저장하여 관리하게 됩니다.

_JPA의 엔티티의 접근 방식 = @Id 위치_

결과적으로 JPA의 **엔티티의 접근 방식은 @Id 애노테이션의 위치**에 의해 결정되며, 엔티티의 모든 필드 또는 상속된 엔티티의 계층에 대해서도 일관성 있게 적용해줘야 합니다.

``` java
@Entity
public class Member{
	@Id // 필드 방식
	private Long id;
	private String userId;
	private String password;
	private String confirmPassword;
	
	@Transient // JPA에서 인식 불가 → 동작 안함
	public String getComfirmPassword(){ return this.confirmPassword; }
}
```

따라서 다음 문제가 된 엔티티의 구조에선 **@Id 애노테이션의 위치**가 필드에 있으므로 JPA는 **필드 접근 방식**을 따르게 됩니다. 이러한 이유로 confirmPassword 필드는 **영속 대상에서 제외되지 않습니다.**

### 3.2.1. 엔티티 접근 방식의 중요성

이해를 돕기 위해 또 다른 코드를 살펴봅시다.

``` java
@Entity
public class Product {
    private Long id;
    private String name;
    private BigDecimal price;
    private boolean isEvent = true;

    @Id // [1] Property 접근 방식
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
  	// [2] getter 메서드 기준으로 컬럼 생성
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
 }
```

다음 상품 엔티티 객체를 분석해보면 다음과 같습니다.

- [1] @Id 애노테이션 위치 → getter 메서드
	- Property 접근 방식
- [2] JPA는 엔티티를 접근할 때 Property(getter/setter 메서드)방식 기준으로 데이터를 생성한다.
	- 영속 관리 대상 : id, name
	- 영속 제외 대상 : price, isEvent

``` sql
Hibernate: 
    create table product (
       id bigint not null, -- getId()
       name varchar(255),  -- getName()
       primary key (id)		 -- @Id Long id
    )
```

이러한 결과를 토대로 다시 정리해보자면, 엔티티의 접근방식에 따라 애노테이션을 선언해줘야 합니다. 참고로 초기 버전인 [JPA 1.0](https://jcp.org/en/jsr/detail?id=220)에 따르면 **엔티티 접근 방식을 혼합하여 사용할 수 없다**고 정의하고 있습니다. 

> [JPA 2.0](https://jcp.org/en/jsr/detail?id=338) 버전 부터 엔티티의 접근 방식을 혼용할 수 있도록 @Access 애노테이션을 지원하지만 이는 별개의 문제입니다.

``` java
@Entity
public class Product {
		@Id
    private Long id;
    private String name;
    
    // [1] 기능성 메서드 정의
    @Transient // [2] 불필요한 애노테이션 선언
		public Set<GrantedAuthority> getAuthorities() {
      Set<GrantedAuthority> authorities = new LinkedHashSet<GrantedAuthority>();
      authorities.add(role);
      return authorities;
  	}
 }
```

다음 코드를 보시면 주로 @Transient 애노테이션은 기능성 메서드를 구현하기 위해 활용합니다.

따라서 @Transient 애노테이션을 선언하여 [1] 기능성 메서드를 구성했지만 [2] 실제로 동작할 때는 필드 방식에 따라 동작하기 때문에 의미가 없습니다. 더불어 getter 메서드에 아무리 컬럼 매핑 레퍼런스 애노테이션을 선언한들 의미가 없습니다.

또한, 해당 필드에 autorities가 정의되지 않기 때문에 애초에 컬럼 생성과 같은 우려했던 상황이 발생하지 않습니다.

### 4. 정리

@Transient를 요약하면 다음과 같습니다.

1. @Transient는 영속 대상에서 제외한다.
2. JPA 컬럼 매핑 레퍼런스 애노테이션은 Filed, Property 방식을 지원하기 위해 필드와 메서드에 선언할 수 있다.
3. 컬럼 매핑 레퍼런스 애노테이션을 사용할 때 JPA의 엔티티 접근 방식을 살펴보자.
	- @Id 애노테이션의 위치를 보자.(@Access 애노테이션으로 접근 방식을 재정의하지 않는 이상)

학습 과정에서 @Transient를 활용한다면 엔티티 객체는 ORM(Object Relation Mapping)의 역할을 넘어서 도메인 객체(Domain Object)로써 활용할 수 있다고 생각이 들었습니다. 이러한 생각이 들었던 이유는 다음과 같습니다.

- src/main
  - com.moong.api
    - domain
      - Member.java <-- [1] @Entity 클래스

다음 패키지 구조처럼, 일반적으로 실무에선 [1] 엔티티 객체를 **`*.domain`** 패키지 하위에 구성하고 있을 것입니다. 하지만 해당 @Entity 객체는 도메인 객체라기보단 단순히 데이터베이스의 테이블과 값을 매핑만 해주는 역할을 하고 있었습니다. 아무래도 DAO 역할을 그대로 수행하고 있는 것이지요. 

여기서 저는 고민이 생겼습니다.

제가 이해하기엔 도메인 객체란 사용자의 요구사항을 담고 있는, 즉 비즈니스를 담고 있는 온전한 객체라 생각합니다. 하지만 실무에서 엔티티를 단순한 DAO 역할로만 활용하고 사용자의 요구사항을 **`비즈니스 계층으로 나눠 관리`**하고 있었습니다. 해당 비즈니스를 구성하기 위해 [2] **`*Manager`**라는 인터페이스를 구성하고, 이를 [3] 구현한 Repository 클래스를 정의하여 비즈니스 로직을 개발하고 있습니다.

- src/main
  - com.moong.api
    - domain
      - Member.java <-- [1] @Entity 클래스 
		- service
			- MemberManager <-- [2] 인터페이스, 비즈니스 요구사항 정의서
			- MemberManagerImpl  <-- [3] @Repository 인터페이스 구현체

여기서 **<U>계층을 나눠 관리한다는 점을 잘못됐다고 말하는 것은 아닙니다.</U>** 다만 인터페이스 대부분은 구현체와 일대일 관계였습니다. 이는 설계상의 이점보단 관례상 인터페이스를 구현하고 있다고 느꼈습니다. 물론, 인터페이스를 미리 구성하여 추후 변경에 있을 상황을 유연하게 대처할 수 있을지는 모릅니다. 그렇다 하여 인터페이스를 꼭 구성해야 하는 걸까요? 

또한, 단순한 비즈니스 로직이라면 @Entity 클래스에 구성하는 게 옳지 않을까 생각합니다. 하지만 아직은 경험이 부족하여, 어떻게 @Entity 객체를 도메인 객체로 활용할 수 있을지는 감이 잡히질 않습니다. 이에 대한 답을 미래에 저에게 맡기며 글을 마치려 합니다.

---

### 번외, 테스트할 때 영속 대상이 제외된 칼럼이 조회된다면?

@DataJpaTest 애노테이션에 @Transactional 애노테이션이 포함되어 있기 때문에 1차 캐시가 초기화가 되지 않는 현상이 일어날 수 있습니다. 따라서 정확한 테스트 결과를 확인하고 싶다면 1차 캐시를 초기화하시고 확인하시면 됩니다.

- [1] 1차 캐시 초기화
	- em.clear();

``` java
@RunWith(SpringRunner.class)
@DataJpaTest
public class ProductTest {
    @Autowired private ProductRepository productRepository;
    @Autowired private EntityManager em;
  
    @Before
    public void init() {
        productRepository.saveAll(
                Arrays.asList(
                          Product.builder().id(1L).name("RV").isEvent(true).build()
                        , Product.builder().id(2L).name("RC").isEvent(true).build()
                        , Product.builder().id(3L).name("RM").isEvent(true).build()
                        , Product.builder().id(4L).name("RS").isEvent(true).build()
                )
        );
        em.flush();
        em.clear(); // [1] 1차 캐시 초기화
    }

    @Test
    public void findALLTest() {
        System.out.format( "finALL : %s"
                          , productRepository.findAll().stream().findFirst()
        );
    }

}
```
``` java
Hibernate: 
    /* select
        generatedAlias0 
    from
        Product as generatedAlias0 */ select
            product0_.id as id1_2_,
            product0_.name as name2_2_,
            product0_.price as price3_2_ 
        from
            product product0_
```

### 공부할 거리

- JPA의 Field 방식과 Property 방식의 설계의 장단점, 고려해야될 상황
- JPA의 getter/setter 메서드의 내부 동작 프로세스
- 자바의 transient 키워드
  - https://stackoverflow.com/questions/2154622/why-does-jpa-have-a-transient-annotation
- 그외 컬럼 매핑 레퍼런스 애노테이션
  - @Column : 컬럼을 매핑한다.
  -  @Enumerated : enum 타입을 매핑한다.
  - @Temporal : 날짜 타입 매핑한다.
  - @Lob : BLOB, CLOB 타입을 매핑한다.
  - @Transient : 해당 필드를 테이블 컬럼과 매핑 시키지 않는다.
  - @Access : JPA가 엔티티 접근하는 방식을 지정한다.

---

### 참고

- [Oracle Doc - @Transient](https://docs.oracle.com/javaee/7/api/javax/persistence/Transient.html)
- [JPA 1.0(JSR 220)](https://jcp.org/en/jsr/detail?id=220)
- [JPA 2.0(JSR 317)](https://jcp.org/en/jsr/detail?id=317)
- [JPA 2.1(JSR 338)](https://jcp.org/en/jsr/detail?id=338)
- [DZone - The JPA Entity Lifecycle](https://dzone.com/articles/jpa-entity-lifecycle)
- [Baeldung - Hibernate Entity Lifecycle](https://www.baeldung.com/hibernate-entity-lifecycle)
- [JPA Impleamentation Pattern Field access VS Property Access](https://xebia.com/blog/jpa-implementation-patterns-field-access-vs-property-access/)
- [StackOverflow -Why JPA Transient Annotation hava method in target? ](https://stackoverflow.com/questions/38589027/why-jpa-transient-annotation-have-method-in-target?answertab=votes#tab-top)
- [StackOverflow - What does transient annotation mean for methods](https://stackoverflow.com/questions/21477034/what-does-transient-annotation-mean-for-methods)
- 자바 ORM 표준 JPA 프로그래밍
