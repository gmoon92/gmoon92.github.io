---
layout: post
title: "[AOP] Aspect Oriented Programming 개념"
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

# AOP의 첫 단계

### 들어가기전

  AOP는 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍이라 불린다. 본 포스팅에선 AOP의 전반적인 기초적인 개념에 대해 상세히 다룰 예정이다.

 본 포스팅을 시작으로 AOP의 포스팅을 크게 3가지로 나눠 작성할 계획이다. 어찌 됐든 마지막은 포스팅은 스프링의 AOP에 대해 작성할 예정이니, 본 포스팅은 스프링 AOP를 들어가기 전 초석을 다지는 시간이 되었으면 한다.

 여기서 우리는 다음과 같은 학습목표가 있다.

### 학습목표

1. AOP 등장배경
2. AOP 개념과 용어
3. 기존 자바에서 AOP 구현 방식

### 등장배경

 AOP(Aspect Oriented Programming)는 컴퓨터 프로그래밍의 패러다임의 일종이다.

이러한 컴퓨터 프로그래밍들이 제시되어지는 근본적인 이유는 기존의 프로그래밍의 단점을 보완하고자 등장한다.  단편적인 예를들어 절차지향적 프로그래밍를 보완하고자 OOP를 나왔던 것처럼 말이다.

_절차적 프로그래밍 -> OOP -> AOP_

본론으로 돌아와서 AOP는 OOP를 보완하고자 제시된 프로그래밍의 패러다임이다. 자바 개발자라면 완벽하게만 보였던 OOP가 도대체 어떤 한계가 있길래 AOP라는 새로운 프로그래밍이 등장했을까?

#### OOP의 한계 - 극단적인 추상화의 한계

 OOP(Object oriented Programming)은 객체와 클래스에 초점을 맞춘 프로그래밍 기법이다. 이를 토대로 정의된 객체를 재사용 또는 상속하거나 추상화로 유연한 기능 확장하여 프로그래밍을 빠르게 할 수 있다.


 하지만



#### Crosscuting Concerns



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


[AOP 슬라이드](https://slideplayer.com/slide/9380068/)


[AOP 정부 프레임워크 DOC](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj)

- 전반적인 개념
[블로그](http://closer27.github.io/backend/2017/08/03/spring-aop/)
[AOP 구현 세가지 방법 비교](https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ)
[AOP의 구조 + 어노테이션](https://hunit.tistory.com/188)
[해외 블로그](https://dzone.com/articles/implementing-aop-with-spring-boot-and-aspectj)
[블로그 - 스프링이 제공하는 aop](https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/)

- aspect
[스프링 DOC - AOP aspect](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
[스프링 블로그 - aspectj](https://www.baeldung.com/aspectj)

- pointcut
[스프링 블로그 - pointcut](https://www.baeldung.com/spring-aop-pointcut-tutorial)
[블로그](http://blog.naver.com/PostView.nhn?blogId=chocolleto&logNo=30086024618&categoryNo=29&viewDate=&currentPage=1&listtype=0)
[블로그](http://closer27.github.io/backend/2017/08/03/spring-aop/)

- adivce
[해외 블로그 - @After](https://howtodoinjava.com/spring-aop/aspectj-after-annotation-example/)

- 실제 사용법
[스프링 부트에서 aspectJ 형식으로 코드 참고](http://jsonobject.tistory.com/247)


- 동영상

[스터디 스프링5 입문 - AOP 프로그래밍 1 #10](https://www.youtube.com/watch?v=wrHTMsKrKkA&index=6&list=WL&t=0s)
[스터디 스프링5 입문 - AOP 프로그래밍 2 #11](https://www.youtube.com/watch?v=9Gdv6fhhaB0&index=5&list=WL&t=0s)
[스터디 코드로배우는스프링 38 Spring의 AOP](https://www.youtube.com/watch?v=4-JcM7y1M_8&index=7&list=WL&t=0s)
[What is AOP - Aspect Oriented Programming](https://www.youtube.com/watch?v=DuFPj8MlAVo&index=8&list=WL&t=0s)
[신입SW인력을 위한 실전 자바(Java) 스프링(Spring) 동영상과정 제 09강 AOP-I](https://www.youtube.com/watch?v=2F8K9BLgvjE&index=9&list=WL&t=0s)


[Filter, Interceptor, AOP의 흐름](https://doublesprogramming.tistory.com/133)
[Spring Filter, Interceptor AOP 차이 및 정리 ](http://goddaehee.tistory.com/154)
[filter, interceptor, aop의 차이와 그 목적](http://hayunstudy.tistory.com/53)
