---
layout: post
title: "Spring AOP와 AspectJ"
tags: [AOP, Spring]
categories: [Spring, AOP]
subtitle: "Spring AOP와 AspectJ의 비교"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# Spring AOP와 AspectJ의 비교

---

### 들어가기전


### 학습목표

1. s

### Spring AOP

#### Introduction(inter-type)

 예를 들어 Introduction를 사용한다면 bean이 IsModified 인터페이스를 구현하도록 쉽게 캐싱할 수 있다.

 > - [자바지기-Introduction](http://www.javajigi.net/pages/viewpage.action?pageId=1084)

#### Aspect(Advisor)

_Aspect = Advice + Pointcut_

Aspect는 싱글톤 형태의 객체로 존재하는 횡단 관심사의 모듈화이다.

> Spring AOP에선 Aspect를 구현하는 두 가지 방식을 제시하고 있다.
> 1. [XML(스키마 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-schema)
> 2. [@AspectJ(어노테이션 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)


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

- Blog
  - [toby - Spring 3.0 (11) Aspects 모듈의 선택 라이브러리 분석](http://toby.epril.com/?p=620)
