---
layout: post
title: "AOP : Aspect Oriented Programming 개념"
tags: [AOP]
categories: [Spring, AOP]
subtitle: "Spring AOP를 학습하기 전 AOP 개념 정리"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# Spring AOP를 학습하기 전 AOP 개념 정리

---

### 들어가기전

  본 포스팅에선 AOP의 기초적인 개념에 대해 상세히 다룰 예정이다. 추후 Spring AOP를 하기 위함으로 기존 AOP의 개념과 덧붙여 Spring AOP의 개념을 간략히 정리하였다.

### 학습목표

1. AOP 등장배경
2. AOP 개념과 용어

### AOP이란

AOP는 컴퓨터 패러다임의 일종으로 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍이라 불린다.

일반적으로 Asepect를 관점으로 해석하지만, 사전적 의미를 찾아보면 또 다른 의미를 발견할 수 있다.

>- 문법 형식의 하나로써 반복(反復) 등을 나타내는 동사의 형태.
>   - 반복을 나타내는 동사 → 반복된 코드

다음과 같은 의미를 빗대어 프로그램 측면에서 해석하자면 반복되어 나타나는 중복된 코드가 이에 해당한다고 볼 수 있다.

![img](/md/img/aop/cross-cutting-concerns.png)

일반적으로 이러한 코드는 비즈니스 기능과 관계없는 부가적으로 발생하는 중복된 코드일 가능성이 크고, 이 코드들을 횡단 관심사(Cross-cutting Concerns)라 표현한다.

#### 관심 분리(Separation of Concerns)

 AOP의 메커니즘은 프로그램을 관심사(Concerns) 기준으로 크게 핵심 관심사와 횡단 관심사로 분류하고 있다.

![img](/md/img/aop/application-concerns.png)

- 핵심 관심사(Core Concerns)
- 횡단 관심사(Cross-cutting Concerns)

핵심 관심사는 프로그램의 핵심 가치와 목적이 그대로 드러난 관심 영역을 뜻한다. 해당 프로그램의 비즈니스 기능이 그러하다. 반면 횡단 관심사는 비즈니스 기능과는 다른 관심 영역을 뜻한다.

프로그램 관점에서의 대표적인 횡단 관심사는 다음과 같다.

<img src="/md/img/aop/concerns-relation.png" style="max-height: 450px;" alt="img">

- Security
- Profiling
- Logging
- Transaction Management

이처럼 보안, 프로파일링, 로그, 트랜잭션은 비즈니스 기능은 아니지만, 요구 상황에 따라서 다수의 비즈니스 기능에 포함되는 부가 기능들이다.

![img](/md/img/aop/cross-cutting-concerns2.png)

횡단 관심사는 비즈니스 기능과는 별개의 영역이지만 필연적으로 대다수의 비즈니스 기능에 분포되어 있다.

 이러한 관계는 비즈니스 기능 상에서 많은 부분에 서로 엉켜져 있어 가독성을 떨어트리고 자칫 중복된 코드가 생겨날 가능성이 크다. 이는 비즈니스 기능의 모듈성을 감소시키는 가장 큰 요인이다.

- 비즈니스 기능 모듈화 감소
- 유지보수의 어려움

이러한 문제들은 Wikipedia에서도 찾아볼 수 있다.

>[In computing, aspect-oriented programming (AOP) is a programming paradigm that aims to increase modularity by allowing the separation of cross-cuttingting concerns. ... - Wikipedia AOP](https://en.wikipedia.org/wiki/Aspect-oriented_programming)

Wikipedia에 정의된 글을 보면 AOP는 "횡단 관심사의 분리를 허용함으로써 모듈성을 증가"라는 목표를 두고 있다.

_횡단 관심사와 핵심 관심사를 분리 → 모듈성 증가_

이러한 AOP의 목표에 대해 생각하자면 횡단 관심사를 관리를 수월하기 위해 모듈화가 필요하고 동시에 AOP라는 새로운 프로그래밍이 등장했다고 해석할 수 있다.

### 등장배경

 일반적으로 새로운 컴퓨터 프로그래밍들이 제시되는 근본적인 이유는 기존의 프로그래밍 단점을 보완하는 데에 있다. 절차적 프로그래밍을 보완하고자 OOP가 등장했던 것처럼 말이다.

_절차적 프로그래밍 → 객체 지향 프로그래밍(OOP) → 관점 지향 프로그래밍(AOP)_

본론으로 돌아와서, AOP는 OOP를 보완하고자 등장한 패러다임이다. 물론 OOP를 통해 횡단 관심사를 분리할 수 있지만, OOP는 횡단 관심사 모듈화의 한계가 있다.

#### OOP의 극단적인 추상화

 OOP(Object oriented Programming)는 객체와 클래스에 초점을 맞춘 프로그래밍 기법이다. 이러한 OOP의 가장 큰 장점은 상속과 추상화를 통해 기능의 분리를 하여 유연한 기능의 확장을 할 수 있다는 점이다.

따라서 횡단 관심사를 `추상화`와 `템플릿 메소드 패턴(디자인 패턴)`을 통해 기존 클래스에서 횡단 관심사와 핵심 관심사를 분리하고 각각의 독립적인 모듈로 분리하여 관리할 수 있다.

예를 들어 트랜잭션 기능(횡단 관심사)과 비즈니스 기능(핵심 관심사)이 하나의 클래스에 공존하고 있는 UserService 클래스가 있다고 가정하자. 이때 가장 우선으로 해야 할 작업은 기존 클래스를 특수화(Specialization)하여 횡단 관심사와 핵심 관심사를 분리하는 작업을 해야 한다.

<img src="/md/img/aop/oop-concerns-div1.png" style="max-height: none;" alt="img">

- UserService : 추상화 비즈니스 모듈
- UserServiceImple : 비즈니스 모듈
- UserServiceTX : 트랜잭션 모듈

설계적 측면으로 하나의 인터페이스가 존재하고 이를 구현한 두 개의 클래스가 도출된다.

<img src="/md/img/aop/oop-concerns-div2.png" style="max-height: none;" alt="img">

이 과정에서 기존 UserService 클래스는 추상화 클래스로 정의하고 핵심이 되는 비즈니스 메소드를 추상화 메소드로 정의한다.

<img src="/md/img/aop/oop-concerns-div3.png" style="max-height: none;" alt="img">

그다음 인터페이스(UserService)의 구현체인 `UserServiceImple` 클래스에서 비즈니스 기능을 구현한다.

분리된 핵심 관심사를 UserServiceImple 클래스로 모듈화하여 관리하기 때문에 코드 베이스엔 비즈니스 기능만 남아 있어 코드가 직관적이며 동시에 유지보수가 쉬워진다.

<img src="/md/img/aop/oop-concerns-div4.png" style="max-height: none;" alt="img">

마지막으로 비즈니스 기능에 트랜잭션 기능을 적용하는 작업을 해야 한다.

다음 그림의 빨간색 영역을 보면 횡단 트랜잭션 기능(횡단 관심사)을 적용할 비즈니스 기능(`upgradeLevels()`)에 트랜잭션 기능을 추가했다. 모든 비즈니스 기능은 유연한 확장을 위해 UserService 인터페이스에 위임한다.

![img](/md/img/aop/oop-concerns-result1.png)

전반적인 기능을 담당하는 UserService 인터페이스의 메소드(`UserService.upgradeLevels()` )는 구현 객체인 UserServiceImple 메소드에 위임해줘야 하므로 기존의 의존 관계를 다시 정의해줘야 한다.

_의존 관계 : UserService.class → UserServiceTX.class → UserServiceImple.class_

따라서 UserService에 UserServiceTX를 의존성 주입(DI)을 하고 UserServiceTX엔 UserServiceImple을 DI를 해주어 의존성 관계를 형성해주면 된다. 의존 관계를 재정의함으로써 기존 프로세스에 따라 비즈니스 기능 호출할 시 트랜잭션 기능이 결합한 기능이 호출된다.

하지만 다음과 같은 상황이 닥친다면 문제가 발생한다.

 1. 특정 메소드만 적용
 2. 다른 횡단 관심사를 추가로 적용

이에 대응하기 위해선 이에 맞는 각기 다른 추상화 클래스가 필요하다. 따라서 추상화의 본질적인 장점과는 다르게 많은 추상화 클래스가 생기게 되고 오히려 이를 관리하는데 큰 비용이 든다. 결과적으로 OOP는 객체의 관점으로 횡단 관심사를 분리하기 때문에 많은 추상화 클래스가 생성되고 이를 관리하는데 어려움이 따른다.

![img](/md/img/aop/oop-concerns-result2.png)

- 추상화 클래스 관리의 어려움
- 복잡한 의존 관계

#### AOP의 필요성

앞서 보았던 OOP의 문제점들을 보안하고자 등장한게 바로 AOP이다.

AOP는 분리된 횡단 관심사를 `Aspect`라는 모듈 형태로 만들어서 설계하고 개발을 한다.

Aspect 모듈에는 부가 기능(횡단 관심사)을 내포하고 있으며 자체적으로 부가 기능을 여러 객체의 핵심 기능에 교차로 적용을 시켜주기 때문에 추상화를 통해 분리하는 작업도 필요가 없어짐으로 횡단 관심사 모듈을 효율적으로 관리할 수 있게 된다.

<img src="/md/img/aop/aspect-modules.png" style="max-height: 500px;" alt="img">

>[... It does so by adding additional behavior to existing code (an advice) without modifying the code itself  ... - Wikipedia AOP](https://en.wikipedia.org/wiki/Aspect-oriented_programming)

무엇보다 Aspect 모듈의 가장 큰 장점은 핵심 기능에 부가 기능의 코드가 남아 있지 않아도 된다는 점이다. 이러한 이유엔 대부분의 AOP 프레임워크들이 [Interceptors](https://docs.oracle.com/javaee/6/tutorial/doc/gkeed.html)를 통해 핵심 기능에 부가 기능을 결합하는 방식을 사용하기 때문이다.

- 횡단 관심사의 모듈화
- 효율적인 횡단 모듈 관리

여담이지만 OOP보다는 AOP가 좋은 프로그래밍이 아닌 서로 다른 프로그래밍이라는 것을 인지해야 한다. 따라서 개발자는 상황에 맞게 프로그래밍을 선택해야 한다.

- OOP : 핵심 관심사의 모듈화
- AOP : 인프라 혹은 횡단 관심사의 모듈화

본론으로 들어와서, 앞서 설명한 Aspect가 어떻게 핵심 기능에 적용되는지 알아야 한다.

### AOP 개념 - 동작과 용어

AOP의 메커니즘은 프로그램을 핵심 관심사와 횡단 관심사로 분리하고 분류된 관심사는 각각의 모듈성을 가져야 한다는 게 AOP의 핵심이고 목표이다.

따라서 AOP의 개발 방식은 핵심 관심사를 객체로 횡단 관심사는 aspect라는 모듈로 모듈화하여 각각의 다른 영역으로 개발한다.

- 핵심 관심사 → Object로 모듈화 (*.class)
- 횡단 관심사 → Aspect로 모듈화 (*.aj of AspectJ AOP)

여기서 핵심은 서로 다른 모듈화 방식을 통해 도출된 각각의 모듈들이 최종적으로 어떻게 서로 교차하여 동작하는지 알아야 한다. 이러한 일련의 과정에서 다소 생소한 AOP의 용어들이 나온다.

이러한 AOP 용어가 정리하지 못한 채 개발부터 하게 된다면 많은 어려움이 있다.

#### AOP 용어

AOP의 용어엔 다음과 같다.

<img src="/md/img/aop/aop-terms.png" style="max-height: 400px;" alt="img">

- Aspect
- Advice
- Introduction(inter-type)
- Pointcut
- Target Object
- Joinpoint
- Weaving
- AOP Proxy

이 용어들은 Spring AOP에 국한되어진 용어가 아닌 여러 AOP 프레임워크에서 쓰이는 통상적인 용어들이다. 우선 용어들은 AOP의 동작 방식을 살펴보면서 하나하나 풀어가 보자.

<img src="/md/img/aop/aspect-cycle.png" style="max-height: 400px;" alt="img">

먼저 그림에서 `핵심 관심사(Core Concerns)` 영역을 보면 다음 용어를 볼 수 있다.

- Target Object
- JoinPoint

`Target Object`는 횡단 기능(`Advice`)이 적용될 객체(Object)를 뜻한다.

> Spring AOP에선 Adviced Object라 한다. Spring AOP에선 Runtime Proxy를 사용하여 구현되기 때문에, Target Object는 항상 `Proxy Object`다.

`JoinPoint`는 Target Object안에서 횡단 기능(`Advice`)이 적용될 수 있는 여러 시점을 뜻한다.

이 시점은 프로그램이 실행될 때 (1)예외가 발생되거나 (2)필드(attribute)가 수정되는 시점 또는 (3)객체가 생성(constructor)되는 시점 그리고 (4)메소드가 호출되는 시점 등 프로그램이 실행될때 발생할 수 있는 시점들은 횡단 기능이 적용될 수 있는 시점들이다.

> Spring AOP에서 JoinPoint는 항상 메소드 실행을 나타낸다. `org.aspectj.lang.JoinPoint` Type의 매개 변수를 선언하여 JoinPoint 정보를 Advice에서 사용할 수 있다.

#### 횡단 관심사(Cross-cutting Concerns)

#### 1.1 Aspect(Advisor)

Aspect는 횡단 관심사의 모듈화이다.

횡단 모듈에 필요한 Advice,  Introduction, Pointcut이 내제되어 있다.

_Aspect = Advice + Introduction(inter-type) + Pointcut_

- Advice
- Introduction(inter-type)
- Pointcut


> Spring AOP에선 Aspect를 구현하는 두 가지 방식을 제시하고 있다.
> 1. [XML(스키마 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-schema)
> 2. [@AspectJ(어노테이션 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)


#### 1.2 Advice

Advice는 실제적으로 적용시킬 횡단 기능을 구현한 구현체라 할 수 있다.

이러한 Advice는 기존 핵심 기능에 횡단 기능의 각기 다른 결합점을 제어할 수 있도록 다양한 Advice를 제공하고 있다.

Spring을 포함한 많은 AOP 프레임 워크는 [Interceptors](https://docs.oracle.com/javaee/6/tutorial/doc/gkeed.html)로서 Advice을 모델링하고 , JoinPoit 주변의 인터셉터의 결합된 상태의 체인을 유지하고 실제 런타임 시 체인의 순서를 실행시킨다.

- Before : JoinPoint 이전에 실행
  - 단 예외를 throw 하지 않는 한 실행 흐름이 JoinPoint로 진행되는 것을 방지하는 기능은 없다.
- After returning : JoinPoint가 정상적으로 완료된 후 실행
  - 예를 들어 메소드가 예외를 발생시키지 않고 리턴하는 경우
- After throwing : Exception을 throw하여 메소드가 종료 된 경우 실행
- After(finally) : JoinPoint의 상태(Exception, 정상)와 무관하고 JoinPoint가 실행된 후 실행
- Around : Before와 After가 합쳐진 Advice
  -  메소드 호출 전과 후에 실행 또한 JoinPoint로 진행할지 또는 자체 반환 값을 반환하거나 Exception를 throw하여 권고 된 메소드 실행을 바로 가기할지 여부를 선택하는 작업도 할 수 있다.

> - [interceptors-sample-code](https://github.com/javaee-samples/javaee7-samples/tree/master/cdi/interceptors)
> - [javaee.github.io](https://javaee.github.io/tutorial/interceptors.html)
> - [spring-doc-org.springframework.aop.interceptor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aop/interceptor/ExposeInvocationInterceptor.html)

#### 1.3 Introduction(inter-type)

 Introduction(inter-type)은 Aspect 모듈 내부의 선언된 메소드 또는 필드를 뜻한다.

 ``` java
 public aspect SampleAscpect{
   private String attribute;
   public void method1(){ .... }
   ...
 }
 ```
 Spring AOP를 사용하면 프록시 된 객체에 새로운 인터페이스 (및 해당 구현)를 도입 할 수 있다.  예를 들어 Introduction를 사용한다면 bean이 IsModified 인터페이스를 구현하도록 쉽게 캐싱할 수 있다.

> - [자바지기-Introduction](http://www.javajigi.net/pages/viewpage.action?pageId=1084)

#### 1.4 Pointcut

Pointcut은 여러 JoinPoint 중 실제적으로 Advice할 지점이다.

따라서 Advice는 여러 JointPoint중에서 Pointcut의 표현식에 명시된 JointPoint에서 실행된다. 예를들어 여러 실행 포인트 중에서 특정 이름의 메소드에서 Advice를 하거나 제외하여 실행 시킬 수 있다.

이러한 Pointcut 표현식과 일치하는 JoinPoint를 실행한다는 개념은 AOP의 핵심이다.

>Spring AOP은 기본적으로 AspectJ Pointcut 언어를 사용한다.
> - [Join Points and Pointcuts of Ecplipse DOC](https://www.eclipse.org/aspectj/doc/next/progguide/language-joinPoints.html)
> - [Pointcuts of Ecplipse DOC](https://www.eclipse.org/aspectj/doc/next/progguide/semantics-Pointcuts.html)




#### 관심사의 교차

#### 3.1 Weaving

AOP는 특정 JoinPoint에 Advice하여 핵심 기능과 Aspect가 연결된 객체를 만든다. 이러한 일련의 과정을 Weaving이라 한다.

Weaving은 수행 시점에 따라 Compile Weaving, Runtime Weaving으로 나뉜다.

- Compile Weaving : 예 AspectJ 컴파일러 사용
- Runtime Weaving : Spring AOP, 순수 자바 AOP 프레임워크

#### 3.2 AOP proxy

AOP proxy는 Aspect를 구현하기 위해 AOP 프레임워크에 의해 생성된 Object이다.

Spring에선 다음과 같은 AOP proxy를 제공하고 있다.

- JDK dynamic proxy
- CGLIB proxy.

Spring 프레임 워크에서 AOP 프록시는 JDK 동적 프록시 또는 CGLIB 프록시가 될 것이다. 프록시 생성은 Spring 2.0에서 소개 된 aspect 선언의 스키마 기반 및 @AspectJ 스타일의 사용자에게는 투명합니다.

### 마무리


---

### 참고

- Reference
  - [Spring-DOC : 11. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
  - [Spring-DOC : Chapter 6. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/2.0.x/reference/aop.html)
  - [Baeldung : Intro to AspectJ](https://www.baeldung.com/aspectj)
  - [Baeldung : Introduction to Pointcut](https://www.baeldung.com/spring-aop-Pointcut-tutorial)
  - [egovframework : aop-aspect](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj)
  - [stackoverflow : ](https://stackoverflow.com/questions/29650355/why-in-spring-aop-the-object-are-wrapped-into-a-jdk-proxy-that-implements-interf)
---

- 해외
  - [spring-3-part-6-spring-aop](http://ojitha.blogspot.com/2013/03/spring-3-part-6-spring-aop.html)
  - [Aspect-Oriented Programming vs. Object-Oriented Programming](https://study.com/academy/lesson/aspect-oriented-programming-vs-object-oriented-programming.html)
  - [the-basics-of-aop](https://blog.jayway.com/2015/09/07/the-basics-of-aop/)
  - [Spring AOP AspectJ @After Annotation Example](https://howtodoinjava.com/spring-aop/aspectj-after-annotation-example/)
  - [Implementing AOP With Spring Boot and AspectJ](https://dzone.com/articles/implementing-aop-with-spring-boot-and-aspectj)

---

- 전반적인 개념
  - [AOP 개념](https://devjms.tistory.com/70)
  - [AOP 웹공학](http://www.jidum.com/jidums/view.do?jidumId=312)
  - [Day 2 - 스프링 AOP(Aspect Oriented Programming)](http://closer27.github.io/backend/2017/08/03/spring-aop/)
  - [AOP의 구조 + 어노테이션](https://hunit.tistory.com/188)
  - [Spring-AOP, Proxy 란?](https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/)
  - [3. 스프링 AOP (AspectJ의 Pointcut 표현식) ](http://blog.naver.com/PostView.nhn?blogId=chocolleto&logNo=30086024618&categoryNo=29&viewDate=&currentPage=1&listtype=0)
  - [스프링 부트에서 aspectJ 형식으로 코드 참고](http://jsonobject.tistory.com/247)

---

- 동영상
  - [What is AOP - Aspect Oriented Programming](https://www.youtube.com/watch?v=DuFPj8MlAVo&index=8&list=WL&t=0s)
  - [스터디 스프링5 입문 - AOP 프로그래밍 1 #10](https://www.youtube.com/watch?v=wrHTMsKrKkA&index=6&list=WL&t=0s)
  - [스터디 스프링5 입문 - AOP 프로그래밍 2 #11](https://www.youtube.com/watch?v=9Gdv6fhhaB0&index=5&list=WL&t=0s)
  - [스터디 코드로배우는스프링 38 Spring의 AOP](https://www.youtube.com/watch?v=4-JcM7y1M_8&index=7&list=WL&t=0s)
  - [신입SW인력을 위한 실전 자바(Java) 스프링(Spring) 동영상과정 제 09강 AOP-I](https://www.youtube.com/watch?v=2F8K9BLgvjE&index=9&list=WL&t=0s)

---

- slideplayer
  - [AOP 슬라이드](https://slideplayer.com/slide/9380068/)

---

- 그외
  - [Spring Filter, Interceptor AOP 차이 및 정리 ](http://goddaehee.tistory.com/154)
  - [Filter, Interceptor, AOP의 흐름](https://doublesprogramming.tistory.com/133)
  - [filter, interceptor, aop의 차이와 그 목적](http://hayunstudy.tistory.com/53)
