---
layout: post
title: "Spring AOP or AspectJ : AOP 선언의 고민"
tags: [AOP, Spring, AspectJ, Srpring-AOP]
categories: [Spring, AOP]
subtitle: "XML & @AspectJ & AspectJ"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# XML & @AspectJ & AspectJ

---

### 들어가기전

 프로젝트의 시간이 많이 지난 시점에서 추가 요구사항이 처리하는데 있어 난감한 경우가 많다.

 문제없이 잘 돌아가는 시스템에 기존 코드를 수정해야 하는 상황이 발생한다면 개발자로선 부담된다. 특히 추가 기능이 다수의 기존 기능에 추가되어야 하는 상황이라면 개발자에겐 최악의 시나리오다.

  물론 이러한 상황을 배제하더라도 기존 코드를 분석하는 시간과 테스트 시간 및 요구사항을 구현하는 데 있어 예기치 못한 오류가 발생하여 상당한 시간이 소요될 수 있다.

이러한 문제점들을 최소화하고 부가 기능을 추가하는 방법으로 관점(Aspect)으로 접근하는 방법이 가장 최선의 방법일 수 있다.

따라서 이러한 요구사항을 처리하는데 있어 AOP로 접근하는 방법이있다.

   주어진 요구사항을 처리하는데 있어

 Aspect가 가장 좋은 접근이라고 결정했다면,

  Spring AOP의 구현방식에 대해 알아보자.

### 학습목표

1. XML스키마 기반 방식의 이해
2. @AspectJ 어노테이션 방식의 이해
3. AspectJ 방식의 이해

### AOP 방식 선택

Spring AOP에선 기본적으로 두 가지 방식을 통해 Aspect를 구현할 수 있다.

1. [XML(스키마 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-schema)
2. [@AspectJ(어노테이션 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)

#### XML스키마 기반

먼저 XML 기반의 형식을 선호한다면 Spring이 제공하는  `aop`라는 Namespace 태그를  사용하여 aspect를 정의할 수 있다. (단. aspect와 advisor는 반드시 `<aop:config>` 요소 내부에 정의되어야 한다.)

``` xml
<bean  id="loginProfiler"  class="..." > <!-- bean def -->
  ...
</bean>

<aop:config>
  <aop:aspect  id="aspectHandler"  ref="loginProfiler" > <!-- aspect 정의 -->
      ...
  </aop:aspect>
</aop:config>
```

일반적으로 `<aop:config>` 태그를 사

#### @AspectJ

### 참고

- [AOP 구현 세가지 방법 비교](https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ)

  - [Spring-DOC : 11. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
