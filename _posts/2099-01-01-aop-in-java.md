---
layout: post
title: "Java에서 다양한 AOP 구현 방법"
tags: [AOP]
categories: [Spring, AOP]
subtitle: "JDK Dynamic Proxy, CGLIB, AspectJ 비교 분석"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# JDK Dynamic Proxy, CGLIB Proxy, AspectJ 비교 분석

---

### 들어가기전

  본 포스팅에선 Java에서 AOP를 구현할 수 있는 JDK Dynamic Proxy, CGLIB, AspectJ의 방식에 대해 작성할 예정이다.

### 학습목표

1. AOP 등장배경
2. AOP 개념과 용어
3. 기존 자바에서 AOP 구현 방식

### JDK Dynamic Proxy, CGLIB, AspectJ

- JDK Dynamic Proxy
- CGLIB
- AspectJ

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


### 참고

- [AOP 구현 세가지 방법 비교](https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ)
