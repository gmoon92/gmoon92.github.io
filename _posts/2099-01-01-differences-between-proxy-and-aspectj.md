---
layout: post
title: "Spring AOP 구현 방법과 적용하는 방식들"
tags: [AOP]
categories: [Spring, AOP]
subtitle: "JDK Dynamic Proxy & CGLIB Proxy & AspectJ"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# JDK Dynamic Proxy & CGLIB Proxy

---

### 들어가기전

  본 포스팅에선 Spring AOP에서 적용하는 방식에 따라 JDK Dynamic Proxy, CGLIB, AspectJ로 분류할 수 있는데, 이 방식들에 대해 알아보고 비교해보자.

### 학습목표

1. Spring AOP의 구현방식
2. JDK Dynamic Proxy의 이해
3. CGLIB의 이해
4. AspectJ의 이해

### AspectJ, JDK Dynamic Proxy, CGLIB

- AspectJ
- JDK Dynamic Proxy
- CGLIB(Code Generator Library)


Runtime(동적) : JDK Dynamic Proxy, CGLIB - 프록시 기반

Compile time(정적) : AspectJ    - 타깃 기반 (타깃 오브젝트를 직접 조작하는 방식)

#### 1. Proxy 기반

기본은 인터페이스의 유무에 따라 나눠짐

- y : JDK Dynamic Proxy
- n : cglib

#### 1.1. JDK Dynamic Proxy (InvocationHandler)

- 타겟 메소드가 호출될 때 Advice를 적용

#### 1.2. CGLIB(MethodInterceptor)

- 메써드가 처음 호출 되었을때 동적으로 bytecode를 생성하여 이후 호출에서는 재사용
- 클래스에 대한 Proxy가 가능

#### 2. AspectJ


참고[https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ]


무슨일일까... 이와 같은 현상은 다음 링크를 통해 답을 찾을 수 있다.

- [springboot-issues#8434](https://github.com/spring-projects/spring-boot/issues/8434)
- [springboot-issues#5423](https://github.com/spring-projects/spring-boot/issues/5423)

>We've generally found cglib proxies less likely to cause unexpected cast exceptions. - Phil Webb(Spring Framework committer and current lead of Spring Boot.)

현재 Spring Boot의 프로젝트 리더인 Phil Web은 CGLIB Proxy 방식이 예기치 않은 캐스팅 예외를 일으킬 가능성이 적다고 한다.

이러한 이슈로 Spring 4.3, Spring Boot 1.4 이상의 버전부터 proxyTargetClass 옵션이 `proxyTargetClass=true`로 기본적으로 셋팅된다. 즉 bean을 구성하기 위해 인터페이스로 구현하지 않아도 되고 인터페이스 유무와 상관없이 CGLIB Proxy가 생성됨을 의미한다.

> [Spring-Boot-1.4-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-1.4-Release-Notes)


### 참고

- [AOP 구현 세가지 방법 비교](https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ)



---



### 참고

- Reference
  - [Spring-DOC : 11. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
  - [Spring-DOC : Chapter 6. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/2.0.x/reference/aop.html)
  - [Baeldung : Intro to AspectJ](https://www.baeldung.com/aspectj)
  - [Baeldung : Introduction to Pointcut](https://www.baeldung.com/spring-aop-Pointcut-tutorial)
  - [egovframework : aop-aspect](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj)
  - [stackoverflow](https://stackoverflow.com/questions/29650355/why-in-spring-aop-the-object-are-wrapped-into-a-jdk-proxy-that-implements-interf)
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
