---
layout: post
title: "Spring AOP와 AspectJ"
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

# AspectJ vs JDK Dynamic Proxy vs CGLIB Proxy

---

### 들어가기전

Spring에선 AspectJ와 Spring AOP를 통해 AOP를 구현할 수 있다.

이 AOP들은 유사하지만, 분명히 다른 AOP 방식이기 때문에 구분해야 한다. 하지만 개발자들이 @AspectJ의 어노테션을 사용하여 Spring AOP를 구현하다 보니 많은 개발자가 AspectJ와 Spring AOP에 대한 차이점에 대해 혼선을 빚는 경우를 종종 보았다. 본 포스팅에선 AspectJ와 Spring AOP에 대해 정리해볼 시간을 가지려 한다.

### 학습목표

1. Spring AOP의 JDK Dynamic Proxy와 CGLIB의 차이의 이해
2. AOP의 Weaving 방식의 차이에 대한 이해

#### 1. 방향성의 차이

우선 AspectJ와 Spring AOP는 서로 다른 목표를 가지고있다.

AspectJ는 이미 알고 있다시피 Java 진영에서 대표적인 AOP 프레임워크이다. AspectJ의 성숙한 AOP의 기술을 통해 모든 조인 포인트 및 객체에 AOP를 적용할 수 있다. 하지만 AspectJ의 개발 난이도가 높고 복잡하다.

반면, Spring AOP는 AOP의 기술보다는 편의성을 중점으로 두고 있다. 개발자가 비즈니스 개발에 집중할 수 있도록 간단한 AOP 구현 방식들을 제공하고, 이를 토대로 간편하게 IoC 컨테이너에 AOP를 적용할 수 있게 도와준다. 하지만 Spring AOP는 Spring Ioc 컨테이너에서 관리되는 Bean에만 AOP를 적용할 수 있고 프록시 기반으로 인한 성능 문제 그리고 조인 포인트의 한계를 가지고 있어서 완벽한 AOP의 기술을 제공한다곤 말할 수 없다.

#### 2 Weaving의 차이

Spring AOP와 AspectJ는 다른 방식의 Weaving 방식을 취하고 있다.

먼저 AspectJ는 세 가지의 Weaving 방식을 지원한다.

- CTW(Compile-time weaving) : AJC(AspectJ Compiler)를 이용해서 소스코드를 컴파일 할때 Weaving
- PCW(Post-compile weaving) : 이미 컴파일된 바이너리 클래스에 Weaving
- LTW(Load-time weaving) : Class Loader가 클래스를 로딩할 때 Weaving (Weaving Agent 필요합니다)

#### 2.1. AspectJ Weaving

AspectJ에는 AJC (AspectJ Compiler)라는 컴파일러가 있는데 Java Compiler를 확장한 형태의 컴파일러이다. AJC를 통해 java파일을 컴파일 하며, 컴파일 과정에서 바이트 코드 조작을 통해 Advisor 코드를 직접 삽입하여 위빙을 수행한다. 장점으로는 3가지 위빙 중에서는 가장 빠른 퍼포먼스를 보여준다. (JVM 상에 올라갈 때 메소드 내에 이미 advise 코드가 삽입 되어있기 때문) 하지만 컴파일 과정에서 lombok과 같이 컴파일 과정에서 코드를 조작하는 플러그인과 충돌이 발생할 가능성이 아주 높다. (거의 같이 사용 불가)

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
- [Spring Blog - Proxies impact performance](https://spring.io/blog/2007/07/19/debunking-myths-proxies-impact-performance)
- [Toby - Spring 3.0 (11) Aspects 모듈의 선택 라이브러리 분석](http://toby.epril.com/?p=620)
- [kezhuw - Spring AspectJ LTW](http://blog.kezhuw.name/2017/08/31/spring-aspectj-load-time-weaving/)
- [Credera - Spring JDK Proxies vs CGLIB vs AspectJ](https://www.credera.com/blog/technology-insights/open-source-technology-insights/aspect-oriented-programming-in-spring-boot-part-2-spring-jdk-proxies-vs-cglib-vs-aspectj/)
- [doanduyhai - Anti AspectJ](https://doanduyhai.wordpress.com/2011/12/12/advanced-aspectj-part-ii-inter-type-declaration/)
- [howtodoinjava - Top Spring AOP Interview Questions with Answers](https://howtodoinjava.com/interview-questions/top-spring-aop-interview-questions-with-answers/)
- [nurkiewicz - CTW vs LTW Performance in Spring](https://www.nurkiewicz.com/2009/10/yesterday-i-had-pleasure-to-participate.html)
