---
layout: post
title: "AOP : Aspect Oriented Programming 개념"
tags: [AOP]
categories: [Spring, AOP]
subtitle: "Spring AOP를 들어가기전 AOP 용어와 개념"
feature-img: "md/img/thumbnail/java-script-logo.png"
thumbnail: "md/img/thumbnail/java-script-logo.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# AOP의 이해와 원리

---

### 들어가기전

  AOP(Aspect Oriented Programming)는 컴퓨터 프로그래밍의 패러다임의 일종이다. 본 포스팅에선 AOP에 대한 기초적인 개념에 대해 상세히 다룰 예정이다.

### 학습목표

1. AOP 등장배경
2. AOP 개념과 용어
3. 기존 자바에서 AOP 구현 방식

### AOP

AOP는 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍이라 불린다.

일반적으로 Asepect를 관점으로 해석하지만, 사전적 의미를 찾아보면 또 다른 의미를 발견할 수 있다.

>- 문법 형식의 하나로써 반복(反復) 등을 나타내는 동사의 형태.
>   - 반복을 나타내는 동사 → 반복된 코드

다음과 같은 의미를 빗대어 프로그램 측면에서 해석하자면 반복되는 코드가 이에 해당한다고 볼 수 있다. 이러한 코드를 횡단 관심사(Crosscutting Concerns)라 표현한다. Concerns에 대해 좀 더 깊게 들어가자면 프로그램에는 크게 핵심 관심사와 횡단 관심사로 나뉠 수 있다.

#### 분류된 관심사

- 핵심 관심사(Core Concerms)
- 횡단 관심사(Crosscutting Concerns)

핵심 관심사는 프로그램의 핵심 가치와 목적이 그대로 드러난 관심 영역을 뜻한다.  해당 프로그램의 비즈니스 로직이 그러하다.

반면 횡단 관심사는 비즈니스 로직과는 다른 관심 영역을 뜻한다. 프로그램 관점에서의 대표적인 횡단 관심사는 다음과 같다.

- Security
- Profiling
- Logging
- Transaction Management

정리하자면 횡단 관심사란 비즈니스 로직과는 별개의 영역으로 대다수 기능에서 발생하는 중복된 공통 로직을 의미한다.

 또 다른 측면으로 생각해보자면 핵심 관심사와 횡단 관심사로 분류할 수 있다는 의미는 횡단 관심사 또한 모듈화도 가능하다는 의미이다.

#### AOP이란

본론으로 돌아와서 AOP는 이러한 "횡단 관심사를 주로 프로그래밍한다."라고 짐작할 수 있다. AOP의 정확한 정의를 알기 위해 Wikipedia의 힘을 빌려보자.

>[In computing, aspect-oriented programming (AOP) is a programming paradigm that aims to increase modularity by allowing the separation of cross-cutting concerns. <br/>  ...  <br/>  It does so by adding additional behavior to existing code (an advice) without modifying the code itself <br/>  ... - Wikipedia AOP](https://en.wikipedia.org/wiki/Aspect-oriented_programming)

Wikipedia에 정의된 글을 보면 AOP는 "횡단 관심사의 분리를 허용함으로써 모듈성을 증가"라는 목표를 두고 있다. 따라서 AOP는 핵심 관심사와 횡단 관심사를 분리함으로써 비즈니스 로직의 모듈성을 증가할 수 있다는 의미이다.

_횡단 관심사와 핵심 관심사를 분리 → 모듈성 증가_

이러한 AOP의 가장 큰 핵심은 비즈니스 로직에 별도의 코드 추가 없이 횡단 관심사를 해결할 수 있다는 점이다.

여기까지 AOP를 개념을 간략하게 훑어보았다. 횡단 관심사가 무엇인지, 어떻게 AOP를 구현하는지에 대한 좀 더 깊게 AOP를 학습하기 위해선 먼저 AOP가 왜 무엇 때문에 등장했는지에 대해 이해하는 과정이 선행되어야 한다.

### 등장배경

 일반적으로 새로운 컴퓨터 프로그래밍들이 제시되는 근본적인 이유는 기존의 프로그래밍 단점을 보완하는 데에 있다. 절차적 프로그래밍을 보완하고자 OOP가 등장했던 것처럼 말이다.

_절차적 프로그래밍 → 객체 지향 프로그래밍(OOP) → 관점 지향 프로그래밍(AOP)_

본론으로 돌아와서, AOP는 OOP를 보완하고자 등장한 패러다임이다.  이 점은 매우 흥미로웠다. 완벽하게만 보였던 OOP에 도대체 어떤 한계가 있었기에 AOP라는 새로운 패러다임이 등장했었을까?

#### OOP의 한계 - 극단적인 추상화의 한계

 OOP(Object oriented Programming)은 객체와 클래스에 초점을 맞춘 프로그래밍 기법이다. 이를 토대로 정의된 객체를 재사용 또는 상속하거나 추상화로 유연한 기능 확장하여 프로그래밍을 빠르게 할 수 있다.

#### Crosscutting Concerns

- 다른 클래스들을 한번에 고치기 위해 추상화 또는 상속 방법으로 클래스 정의
- 이에 따라 관리의 복잡성이 증가
- 횡단 관심사(crosscutting concerns)를 모듈화

### 2. AOP의 개념과 용어정리

- 용어
   - Aspect
   - Join point
   - Advice(interceptor를 )
       - Before
       - After returning
       - After throwing
       - Around
   - pointcut
   - introduction
   - target
   - *AOP proxy
   - Weaving

- 그림으로 한번에 정리


### 3 기존 자바의 AOP 구현 방식

- JDK Dynamic Proxy
- CGLIB
- AspectJ


   - Runtime(동적)      : Jdk Dynamic Proxy, CGLIB - 프록시 기반
   - Compile time(정적) : AspectJ    - 타깃 기반 (타깃 오브젝트를 직접 조작하는 방식)

   Jdk Dynamic Proxy (InvocationHandler)
       - 타겟 메소드가 호출될 때 Advice를 적용

   CGLIB(MethodInterceptor)
       - 메써드가 처음 호출 되었을때 동적으로 bytecode를 생성하여 이후 호출에서는 재사용
       - 클래스에 대한 Proxy가 가능

   프록시 기반
       기본은 인터페이스의 유무에 따라 나눠짐

       y : jdk Dynamic Proxy
       n : cglib


참고[https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ]


4. 스프링 proxy

https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/
6. 스프링은 어떤 방식으로 AOP를제공하고 있을까 ?

   JDK Dynamic Proxy
   CGLIB



6-1. 스프링 AOP의 구현방식
    - XML
    - @AspectJ
    - 비교 분석 장단점



6-2. 스프링 포인트컷
   - 기본
   - 어노테이션

6-3. 어드바이스 라이프 사이클


6-4. weaving 3가지 방식

### 마무리


---

### 참고

[블로그 - Spring 횡단관심(Crosscutting Concerns), 핵심관심(Core Concerns) ](http://winmargo.tistory.com/89)

[해외 - Aspect-Oriented Programming vs. Object-Oriented Programming](https://study.com/academy/lesson/aspect-oriented-programming-vs-object-oriented-programming.html)

[AOP 슬라이드](https://slideplayer.com/slide/9380068/)

[AOP 정부 프레임워크 DOC](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj)

- 전반적인 개념
[블로그 - 스프링 AOP(Aspect Oriented Programming)](http://closer27.github.io/backend/2017/08/03/spring-aop/)
[AOP 구현 세가지 방법 비교](https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ)
[AOP의 구조 + 어노테이션](https://hunit.tistory.com/188)
[해외 블로그 - Implementing AOP With Spring Boot and AspectJ](https://dzone.com/articles/implementing-aop-with-spring-boot-and-aspectj)
[블로그 - 스프링이 제공하는 aop](https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/)

- aspect
[스프링 DOC - AOP aspect](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
[스프링 블로그 - aspectj](https://www.baeldung.com/aspectj)
[스프링 DOC - AOP](https://docs.spring.io/spring/docs/2.0.x/reference/aop.html)

- pointcut
[스프링 블로그 - pointcut](https://www.baeldung.com/spring-aop-pointcut-tutorial)
[블로그 - 3. 스프링 AOP (AspectJ의 Pointcut 표현식) ](http://blog.naver.com/PostView.nhn?blogId=chocolleto&logNo=30086024618&categoryNo=29&viewDate=&currentPage=1&listtype=0)

- adivce
[해외 블로그 - @After](https://howtodoinjava.com/spring-aop/aspectj-after-annotation-example/)

- 실제 사용법
[스프링 부트에서 aspectJ 형식으로 코드 참고](http://jsonobject.tistory.com/247)

---

- 동영상

[스터디 스프링5 입문 - AOP 프로그래밍 1 #10](https://www.youtube.com/watch?v=wrHTMsKrKkA&index=6&list=WL&t=0s)
[스터디 스프링5 입문 - AOP 프로그래밍 2 #11](https://www.youtube.com/watch?v=9Gdv6fhhaB0&index=5&list=WL&t=0s)
[스터디 코드로배우는스프링 38 Spring의 AOP](https://www.youtube.com/watch?v=4-JcM7y1M_8&index=7&list=WL&t=0s)
[What is AOP - Aspect Oriented Programming](https://www.youtube.com/watch?v=DuFPj8MlAVo&index=8&list=WL&t=0s)
[신입SW인력을 위한 실전 자바(Java) 스프링(Spring) 동영상과정 제 09강 AOP-I](https://www.youtube.com/watch?v=2F8K9BLgvjE&index=9&list=WL&t=0s)

---
블로그

[Filter, Interceptor, AOP의 흐름](https://doublesprogramming.tistory.com/133)
[Spring Filter, Interceptor AOP 차이 및 정리 ](http://goddaehee.tistory.com/154)
[filter, interceptor, aop의 차이와 그 목적](http://hayunstudy.tistory.com/53)
