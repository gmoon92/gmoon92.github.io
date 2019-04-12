---
layout: post
title: "Spring AOP와 AspectJ"
tags: [AOP, Spring AOP, Proxy, JDK Dynamic Proxy, CGLIB]
categories: [Spring, AOP]
subtitle: "JDK Dynamic Proxy & CGLIB Proxy"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# AspectJ & JDK Dynamic Proxy & CGLIB Proxy

---

### 들어가기전

이전 포스팅 [Spring AOP 선언의 선택](https://gmun.github.io/spring/aop/2019/03/01/spring-aop-choosing.html)에서 JDK Dynamic Proxy와 CGLIB에 대한 이슈를 잠깐 짚고 넘어갔었다 . Spring AOP의 Proxy들의 구현 방법과 동작 방식에 대해 정리해보려 한다.

### 학습목표

1. JDK Dynamic Proxy의 이해
2. CGLIB의 이해

### Spring AOP의 Proxy


 Spring AOP에 대해 말하자면 Proxy 기반의 AOP 프레임워크 또는 Runtime weaving을 수행하는 AOP 프레임워크라 할 수 있다.

Runtime weaving을 사용하여 다음과 같이 JDK 동적 프록시 또는 CGLIB 프록시를 사용하여 대상 객체의 프록시를 사용하여 응용 프로그램을 실행하는 동안 여러 측면을 만들 수 있습니다.

Spring AOP의 weaving 방식을 살펴보면 알 수 있는데 Target Object에 Aspect를 구현하기 위해 Proxy가 생성되고 프로세스가

Spring AOP는 JDK Dynamic Proxy와 CGLIB 방식을 사용하여  Proxy를 생성한다.


 Spring AOP는 Proxy 기반의 Runtime weaving을 수행하는 AOP 프레임워크다. 즉 프로세스가 실행하는 중에 Target Object에 Aspect를 구현하기 위해 Spring AOP는 JDK Dynamic Proxy와 CGLIB를 사용하여 Target Object에 대한 Proxy를 생성한다.


Spring AOP는 Proxy를 기반으로 하는 AOP 프레임워크이다.

실행 과정에서 Target Object에 Aspect를 구현하기 위해 Proxy를 생성한다.

- JDK Dynamic Proxy
- CGLIB Proxy


이때 Spring AOP는 다음 두 Proxy중에 하나를 선택하여 weaving을 수행한다.

- AspectJ
- JDK Dynamic Proxy
- CGLIB(Code Generator Library)


Runtime(동적) : JDK Dynamic Proxy, CGLIB - 프록시 기반

Compile time(정적) : AspectJ    - 타깃 기반 (타깃 오브젝트를 직접 조작하는 방식)

#### 1. Proxy 기반

기본은 인터페이스의 유무에 따라 나눠짐

- y : JDK Dynamic Proxy
- n : cglib


일반적으로 Compile 시점 또는 Class Load 시점에 Binary 코드를 조작하여 weaving하는 AspectJ와 달리 Spring AOP는 Runtime weaving을 기반으로 수행하기 때문에,  응용프로그램이 실행하는 도중에 Proxy를 활용하여 다양한 Aspect를 만들수 있다.


#### 1.1. JDK Dynamic Proxy (InvocationHandler)

- 타겟 메소드가 호출될 때 Advice를 적용

#### 1.2. CGLIB(MethodInterceptor)

- 메써드가 처음 호출 되었을때 동적으로 bytecode를 생성하여 이후 호출에서는 재사용
- 클래스에 대한 Proxy가 가능

#### 2. AspectJ


### 참고


- Blog
  - [toby - Spring 3.0 (11) Aspects 모듈의 선택 라이브러리 분석](http://toby.epril.com/?p=620)

- Spring
  - [Top Spring AOP Interview Questions with Answers](https://howtodoinjava.com/interview-questions/top-spring-aop-interview-questions-with-answers/)
