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

### AOP 개념 - 용어와 동작

AOP의 메커니즘은 프로그램을 핵심 관심사와 횡단 관심사로 분리하고 분류된 관심사는 각각의 모듈성을 가져야 한다는 게 AOP의 핵심이고 목표이다.

따라서 AOP의 개발 방식은 핵심 관심사를 객체로 횡단 관심사는 aspect라는 모듈로 모듈화하여 각각의 다른 영역으로 개발한다.

- 핵심 관심사 → Object 기반으로 모듈화
- 횡단 관심사 → Aspect 기반으로 모듈화

여기서 핵심은 서로 다른 모듈화 방식을 통해 도출된 각각의 모듈들이 최종적으로 어떻게 서로 교차하여 동작하는지 알아야 한다. 이러한 일련의 과정에서 다소 생소한 AOP의 용어들이 나온다.

![img](/md/img/aop/aop-terms.png)

- Aspect
- Advice
- Introduction(inter-type)
- Pointcut
- Target Object
- Joinpoint
- AOP Proxy
- Weaving

이러한 AOP의 용어들은 Spring AOP에 국한되어진 용어가 아닌 여러 AOP 프레임워크에서 쓰이는 통상적인 용어들이다. 따라서 AOP 용어가 정리하지 못한 채 개발부터 하게 된다면 많은 어려움이 있다.

#### AOP 용어

먼저 AOP 용어들은 다음과 같다.

<img src="/md/img/aop/aop-terms-explanation.png" style="max-height: 620px; margin: 0; padding: 0;" alt="img">

아무래도 이 용어들을 단번에 이해하기엔 부족하다. 이 용어들을 관심사를 기준으로 하나하나 풀어가 보자.

#### 용어와 동작 - 핵심 관심사 영역

다음 그림은 `핵심 관심사(Core Concerns)` 영역이다.

<img src="/md/img/aop/module-of-core.png" style="max-height: 400px;" alt="img">

- Target Object
- JoinPoint

##### Target Object

먼저 `Target Object`는 횡단 기능(`Advice`)이 적용될 객체(Object)를 뜻한다. 이 객체는 핵심 모듈(비즈니스 클래스)이라 할 수 있다. Spring AOP에선 Advice를 받는 객체라 하여 `Adviced Object`라는 용어로 쓰이기도 한다.

Spring AOP에선 실제 적용할 객체 대신 `Runtime Proxy`를 사용하여 구현되기 때문에, Target Object는 항상 `Proxy Object`다.

##### JoinPoint

`JoinPoint`는 Target Object안에서 횡단 기능(`Advice`)이 적용될 수 있는 여러 위치를 뜻한다.

![img](/md/img/aop/joinpoint.png)

이 위치는 프로그램이 실행될 때 (1) 예외가 발생하거나 (2) 필드(attribute)가 수정되는 시점 또는 (3) 객체가 생성(constructor)되는 시점 그리고 (4) 메소드가 호출되는 시점 등 그 외 프로그램이 실행될 때 발생할 수 있는 모든 시점은 횡단 기능이 적용될 수 있는 위치들이다.

일반적으로 AspectJ는 모든 JoinPoint에 접근이 가능하지만 Spring AOP는 기본적으로 메소드 interceptor를 기반으로 하고 있어서 JoinPoint는 항상 메소드 단위다.

#### 용어와 동작 - 횡단 관심사 영역

##### Aspect

횡단 관심사 영역에서 가장 먼저 살펴볼 용어는 `Aspect`이다.

AOP는 횡단 관심사를 Aspect라는 독특한 모듈을 기반으로 모듈화를 한다. 이러한 횡단 관심사 모듈화를 `Aspect`라 한다. Java에선 사실상 표준으로 가장 널리 사용되는 `AspectJ`이라는 확장 기능을 통해 Aspect를 구현할 수 있다.

>AsepctJ의 모듈은 `*.aj`라는 확장자를 가진 독특한 파일로 구현된다. (참고 - [Eclipse AspectJ](https://www.eclipse.org/aspectj/))

AspectJ는 기본적으로 이클립스에서 확장 기능을 추가하고 그 외 별도의 설정을 해야 한다. 또한, aj는 기존 Java 문법과 비슷하면서도 사뭇 달라서 별도의 학습이 필요하다. (참고 - [Eclipse DOC - Starting-AspectJ](https://www.eclipse.org/aspectj/doc/next/progguide/starting-aspectj.html), [블로그 - Eclipse에서 AspectJ 시작하기](https://busy.org/@nhj12311/aop-aspectj-java-aop-5))

반면 Spring AOP에서 제공하는 어노테이션(@AspectJ)을 통한 구현 방식은 AspectJ 보다 쉬운 설정과 기본적으로 클래스로 구현하기 때문에 쉽게 접근할 수 있다는 장점이 있다.

Spring AOP에선 기본적으로 두 가지 방식을 통해 Aspect를 구현할 수 있다.

1. [XML(스키마 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-schema)
2. [@AspectJ(어노테이션 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)

이 Aspect 모듈은 핵심 모듈에 횡단 코드를 적용하기 위한 최종 목적을 띄고, 횡단 모듈의 관리 유용성을 증가시키기 위한 횡단 관심사의 집합체다. 이 때문에 Aspect에는 횡단 코드와 이 코드가 어디서 언제 적용할지 구현해야 한다.

<img src="/md/img/aop/module-of-aspect.png" style="max-height: 400px;" alt="img">

_Aspect = Advice + Pointcut + Introduction(inter-type)_

- Advice
- Pointcut
- Introduction(Inter-type)

##### Advice

`Advice`는 JoinPoint에 적용할 `횡단 코드`를 의미한다.

![img](/md/img/aop/advice.png)

Spring을 포함한 많은 AOP 프레임워크는 [Interceptors](https://docs.oracle.com/javaee/6/tutorial/doc/gkeed.html)로서 Advice을 모델링하고 , JoinPoit 주변의 인터셉터의 결합된 상태의 체인을 유지하고 실제 런타임 시 결합된 코드가 실행된다.

Spring AOP에서 JoinPoint는 항상 메소드 실행을 바라보기 때문에 `org.aspectj.lang.JoinPoint` Type의 매개 변수를 선언하여 JoinPoint 정보를 Advice에서 사용할 수 있다.

> - [interceptors-sample-code](https://github.com/javaee-samples/javaee7-samples/tree/master/cdi/interceptors)
> - [javaee.github.io](https://javaee.github.io/tutorial/interceptors.html)
> - [spring-doc-org.springframework.aop.interceptor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aop/interceptor/ExposeInvocationInterceptor.html)

또한, Spring AOP의 Advice는 JoinPoint와 횡단 코드의 각기 다른 결합점을 제어할 수 있도록 다양한 Advice를 제공하고 있다.

<img src="/md/img/aop/spring-aop-advice.png" style="max-height: 500px;" alt="img">

- @Before : JoinPoint 이전에 실행
  - 단 Exception을 throw 하지 않는 한 실행 흐름이 JoinPoint로 진행되는 것을 방지하는 기능은 없다.
- @AfterReturning : JoinPoint가 정상적으로 완료된 후 실행
  - 예를 들어 메소드가 Exception을 발생시키지 않고 리턴하는 경우
- @AfterThrowing : Exception을 throw하여 메소드가 종료된 경우 실행
- @After(finally) : JoinPoint의 상태(Exception, 정상)와 무관하고 JoinPoint가 실행된 후 실행
- @Around : Before와 After가 합쳐진 Advice
  -  메소드 호출 전과 후에 실행 또한 JoinPoint로 진행할지 또는 자체 반환 값을 반환하거나 Exception를 throw하여 특정 메소드를 호출할 수 있다.

##### Pointcut

Pointcut은 여러 JoinPoint 중 실제적으로 Advice할 JoinPoint이다.

![img](/md/img/aop/joinpoint-pointcut.png)

따라서 Advice는 여러 JoinPoint중에서 Pointcut의 표현식에 명시된 JoinPoint에서 실행된다. 예를 들어 여러 실행 포인트 중에서 특정 이름의 메소드에서 Advice를 하거나 제외하여 실행시킬 수 있다.

이러한 Pointcut 표현식과 일치하는 JoinPoint를 실행한다는 개념은 AOP의 핵심이다. 또한, Spring AOP에선 Pointcut과 Advice를 합쳐 `Advisor`라 불린다. Spring AOP은 기본적으로 AspectJ Pointcut 언어를 사용한다.

> - [Join Points and Pointcuts of Ecplipse DOC](https://www.eclipse.org/aspectj/doc/next/progguide/language-joinPoints.html)
> - [Pointcuts of Ecplipse DOC](https://www.eclipse.org/aspectj/doc/next/progguide/semantics-Pointcuts.html)

##### Introduction(inter-type)

Introduction(inter-type)은 Aspect 모듈 내부의 선언된 클래스 또는 인터페이스, 메소드 그 외 모든 필드를 뜻한다. 이 Introduction의 주된 목적은 기존 클래스에 새로운 인터페이스(및 해당 구현 객체)를 추가하기 위함이다.

이는 OOP에서 말하는 상속이나 확장과는 다른 방식으로 Advice 또는 Aspect를 이용해서 기존 클래스에 없는 인터페이스를 동적으로 추가할 수 있다.

특히 Spring AOP를 사용하면 Proxy된 객체에 새로운 인터페이스를 도입할 수 있다. 또한, bean이 이 인터페이스를 구현하도록 쉽게 캐싱할 수 있다. 이 예제는 [박재성(자바지기)님의 Introduction](http://www.javajigi.net/pages/viewpage.action?pageId=1084)를 참고하자.

##### AOP Proxy

Proxy는 "대신 일을 하는 사람"이라는 사전적 의미를 가지고 있다. 이와 마찬가지로 AOP Proxy는 Aspect를 대신 수행하기 위해 AOP 프레임워크에 의해 생성된 객체(Object)이다.

![img](/md/img/aop/proxy1.png)

일반적으로 Spring을 포함한 많은 AOP 프레임워크에선 핵심 관심 코드에 직접적인 Aspect를 하지 않고 `Proxy Object`를 활용하여 Aspect를 한다.

결과적으로 횡단 관심 객체와 핵심 관심 객체의 느슨한 결합 구조를 만들고, 필요 여부에 따라 부가 기능을 탈 부착하기 용이하게 해준다.

- 직접적인 참조가 아닌 Proxy를 사용하여 동적으로 참조
- 부가 기능의 탈부착이 용이

Spring을 포함하여 대부분 AOP 프레임워크에서 Proxy를 사용하여 동적으로 Advice하기 위해 Java에서 제공해주는 `java.lang.reflect.Proxy`을 사용하여 Proxy 객체를 동적으로 생성해준다.

![img](/md/img/aop/proxy2.png)

구체적으로 `JDK Dynamic Proxy`와 `CGLIB Proxy`의 방식이 존재한다. 물론 Spring AOP에선 두 가지 구현 방식을 제공하고 있다.

- JDK Dynamic Proxy
- CGLIB Proxy

 이 둘의 차이점은 추후 자세하게 포스팅할 예정이다.

#### 용어와 동작 - 관심사의 교차

AOP는 특정 JoinPoint에 Advice하여 핵심 기능과 횡단 기능이 교차하여 새롭게 생성된 객체를 프로세스에 적용하는 일련의 모든 과정을 `Weaving`라 한다.

##### Weaving

이 Weaving은 수행 시점에 따라 CTW, LTW, RTW으로 분류할 수 있다.

- CTW : Compile-Time Weaving (AspectJ Compiler)
- LTW : Load-Time Weaving (AspectJ Compiler)
- RTW : Run-Time Weaving (Spring AOP)

특히 Java 환경에서 AOP를 한다면 가장 먼저 AspectJ와 Spring AOP를 떠올릴 수 있는데, 이 둘은 서로 독립적인 대상이므로 항상 비교가 되고 Weaving에서도 그 차이를 알 수 있다.

먼저 AspectJ는 바이트 코드 기반으로 기존 클래스 코드를 조작하여 AspectJ Compiler(ACJ)에 의해 Aspect를 Weaving하는 방식을 취하고 있다.

<img src="/md/img/aop/aspect-cycle.png" style="max-height: 400px;" alt="img">

따라서 CTW, LTW 방식을 기본적으로 사용하고 있다.

반면 Spring AOP는 Dynamic Proxy 기반으로 기본적으로 CTW, LTW를 사용하지 않고 RTW를 사용한다. Spring AOP의 Weaving의 방식은 AspectJ에 비해 가볍지만, 대부분 기능을 구현할 수 있다.

![img](/md/img/aop/proxy3.png)

이는 프로그램의 퍼포먼스에도 차이가 있다. 이를 입증하듯 성능 비교를 한 글들이 많이 있는데 단편적으로 다음 [링크](https://www.nurkiewicz.com/2009/10/yesterday-i-had-pleasure-to-participate.html)에서도 확인할 수 있다.

### 마무리

여기까지 AOP의 개념적인 부분과 Spring AOP에 대한 부분도 간략히 포스팅 해보았다.

AOP의 용어적인 부분은 나름대로 관심사의 영역을 나눠 작성했지만, 아무래도 처음에 접한 분들이라면 햇갈릴 수 있다고 생각한다.

또한, 본 포스팅은 간략한 개념적인 부분이라 실제적으로 코드에 대한 부분이 부족했다. 다음 포스팅에선 이를 해소시킬 예시 코드와 부분적으로 자세히 풀어갈 예정이다.

---

### 참고

- Blog
  - [Spring-DOC : 11. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
  - [Spring-DOC : Chapter 6. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/2.0.x/reference/aop.html)
  - [Baeldung : Intro to AspectJ](https://www.baeldung.com/aspectj)
  - [Baeldung : Introduction to Pointcut](https://www.baeldung.com/spring-aop-Pointcut-tutorial)
  - [egovframework : aop-aspect](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj)
  - [stackoverflow](https://stackoverflow.com/questions/29650355/why-in-spring-aop-the-object-are-wrapped-into-a-jdk-proxy-that-implements-interf)
  - [spring-3-part-6-spring-aop](http://ojitha.blogspot.com/2013/03/spring-3-part-6-spring-aop.html)
  - [Aspect-Oriented Programming vs. Object-Oriented Programming](https://study.com/academy/lesson/aspect-oriented-programming-vs-object-oriented-programming.html)
  - [the-basics-of-aop](https://blog.jayway.com/2015/09/07/the-basics-of-aop/)
  - [Spring AOP AspectJ @After Annotation Example](https://howtodoinjava.com/spring-aop/aspectj-after-annotation-example/)
  - [Implementing AOP With Spring Boot and AspectJ](https://dzone.com/articles/implementing-aop-with-spring-boot-and-aspectj)
  - [스프링 부트에서 aspectJ 형식으로 코드 참고](http://jsonobject.tistory.com/247)
  - [AOP 슬라이드](https://slideplayer.com/slide/9380068/)

- 그 외
  - [Spring Filter, Interceptor AOP 차이 및 정리 ](http://goddaehee.tistory.com/154)
  - [Filter, Interceptor, AOP의 흐름](https://doublesprogramming.tistory.com/133)
  - [filter, interceptor, aop의 차이와 그 목적](http://hayunstudy.tistory.com/53)
