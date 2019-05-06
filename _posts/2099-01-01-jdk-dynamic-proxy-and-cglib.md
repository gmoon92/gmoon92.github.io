---
layout: post
title: "JDK Dynamic Proxy와 CGLIB에 대해"
tags: [AOP, Spring AOP, Proxy, JDK Dynamic Proxy, CGLIB]
categories: [Spring, AOP]
subtitle: "AspectJ & JDK Dynamic Proxy & CGLIB Proxy"
feature-img: "md/img/thumbnail/aop-weaving.png"
thumbnail: "md/img/thumbnail/aop-weaving.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# JDK Dynamic Proxy vs CGLIB Proxy

---

### 들어가기전

Spring에선 AspectJ와 Spring AOP를 통해 AOP를 구현할 수 있다.

이 AOP들은 유사하지만, 분명히 다른 AOP 방식이기 때문에 구분해야 한다. 하지만 개발자들이 @AspectJ의 어노테션을 사용하여 Spring AOP를 구현하다 보니 많은 개발자가 AspectJ와 Spring AOP에 대한 차이점에 대해 혼선을 빚는 경우를 종종 보았다. 본 포스팅에선 AspectJ와 Spring AOP에 대해 정리해볼 시간을 가지려 한다.

### 학습목표

1. Spring AOP의 JDK Dynamic Proxy와 CGLIB의 차이의 이해
2. AOP의 Weaving 방식의 차이에 대한 이해

### Spring AOP의 종류

- JDK Dynamic Proxy
- CGLIB(Code Generator Library)


Runtime(동적) : JDK Dynamic Proxy, CGLIB - 프록시 기반

Compile time(정적) : AspectJ    - 타깃 기반 (타깃 오브젝트를 직접 조작하는 방식)

#### 1. Proxy 기반

기본은 인터페이스의 유무에 따라 나눠짐

- y : JDK Dynamic Proxy
- n : cglib

일반적으로 Compile 시점 또는 Class Load 시점에 Binary 코드를 조작하여 weaving하는 AspectJ와 달리 Spring AOP는 Runtime weaving을 기반으로 수행하기 때문에,  응용프로그램이 실행하는 도중에 Proxy를 활용하여 다양한 Aspect를 만들수 있다.

`java.lang.ClassLoader` 자체는 동적으로 클래스를 로드하기 위해 인터페이스나 추상화 클래스가 필요하다. 따라서
클래스 로더는 런타임 시점에 클래스 파일을 찾아 로드하는 책임이 있는 클래스


> [What is a Java ClassLoader?](https://stackoverflow.com/questions/2424604/what-is-a-java-classloader)

interface의 매개변수는 프록시 생성에 있어 ㅊ는



#### 1.1. JDK Dynamic Proxy

- 타겟 메소드가 호출될 때 Advice를 적용

#### 1.2. CGLIB

- 메써드가 처음 호출 되었을때 동적으로 bytecode를 생성하여 이후 호출에서는 재사용
- 클래스에 대한 Proxy가 가능

#### 2. AspectJ


### 마무리


또한, 이전 포스팅 [Spring AOP 선언의 선택](https://gmun.github.io/spring/aop/2019/03/01/spring-aop-choosing.html)에서 AOP Proxy의 Self Invocation에 대한 이슈를 잠깐 짚고 넘어갔었는데, 이 이슈와 더불어 본 포스팅에선 AspectJ와 Spring AOP에 대해 정리해볼 시간을 가지려 한다.

---

### 참고

- [Spring 4.0.1.RELEASE - AOP Proxies](https://docs.spring.io/spring/docs/4.0.1.RELEASE/spring-framework-reference/htmlsingle/#aop-understanding-aop-proxies)
- [Spring Blog - CGLIB](https://www.baeldung.com/cglib)
- [Spring Blog - AspectJ](https://www.baeldung.com/aspectj)
- [Spring Blog - Spring AOP vs AspectJ](https://www.baeldung.com/spring-aop-vs-aspectj)
