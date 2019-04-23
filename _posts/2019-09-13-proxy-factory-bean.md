---
layout: post
title: "Spring은 어떻게 동적으로 AOP를 적용할까"
tags: [AOP, Spring, SpringAOP, XML, AspectJ]
categories: [Spring, AOP]
subtitle: "아드리안 콜리어의 Dynamic Weaving"
feature-img: "md/img/thumbnail/spring-aop-choosing.png"
thumbnail: "md/img/thumbnail/spring-aop-choosing.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
display: "false"
priority: 1.0
---

<!--more-->

# 아드리안 콜리어의 Dynamic Weaving

---

### 들어가기전

[이전 포스팅](https://gmun.github.io/spring/aop/oop/2019/02/09/from-oop-to-aop.html)에서 OOP의 디자인 패턴들을 통해 관심사의 분리가 어렵다는 걸 알 수 있었고 AspectJ를 통해 관심사의 분리에 대한 문제를 근본적으로 해결할 수 있었다. 이러한 이유엔 AspectJ는 기본적으로 Target의 바이트 코드를 조작하여 Target에 Advice를 적용하는 Compile-Weaving임으로 객체의 관계를 무시할 수 있었다. 이 AspectJ는 AOP 프레임워크의 대명사라 할 수 있는데, 아드리안 콜리어(Adrian Colyer)는 AspectJ 프로젝트팀의 리더이면서 한 때는 Spring CTO까지 지냈던 사람이다.

따라서 Spring AOP가 굳이 많은 AOP 프레임워크 중에 AspectJ5 라이브러리의 기반으로 구현되어있다는 부분 역시 그의 영향이 미쳤다는 걸 알 수 있다. 하지만 Spring AOP는 기존의 AspectJ의 Weaving 방식과 달리 자바에서 기본적으로 제공하는 `java.lang.reflect.Proxy`를 활용하여 Weaving 하는데 이 대목에서 아드리안 콜리어는 어떻게 객체의 관계를 해결하고 있는지 궁금해졌다.

물론 동적 Weaving을 도입한 이유에 대해 먼저 고민해봐야겠지만, 본 포스팅에선 FactoryBean과 ProxyFactoryBean을 학습을 해보며, 아드리안 콜리어는 어떻게 객체의 관계를 해결했는지 생각해보는 시간을 가지려 한다.

### 학습목표

- 팩토리 메소드 디자인 패턴의 이해
- FactoryBean의 이해
- ProxyFactoryBean의 이해

### 왜 관심사의 분리를 해결하지 못했을까?

이전 포스팅에서 다뤘던 디자인 패턴들로 관심사의 분리하는 데 한계가 있었다. 그 이유에 대해선 크게 두 가지로 정리해볼 수 있다.

1. 관심사의 분리를 하기 위해선 구체적인 구조가 파악되야 한다.
2. 어찌됐든 기존 구조를 변형시켜 해결해야 한다.

OOP로 Weaving을 하기 위해선 적용할 Target의 전체적인 구조가 파악되어야 가능했다. 결과적으로 전체적인 구조가 파악됐더라도 기존의 구조를 변형시킨다는 점은 근본적으로 관심사의 분리를 해결하지 못한 부분이라 말할 수 있다.

제일 쉽게 부가기능을 확장하기 위해선 상속이라는 개념이 제일 베스트이다.
따라서 구체적인 대상을 모를 경우엔 해결할 수 있는 디자인 패턴으로 고려해야한다.

### 팩토리 메소드 패턴과 프록시

팩토리 메소드 패턴은 객체의 정확한 유형과 종속성을 미리 알지 못하는 경우에 유용하게 사용할 수 있는 디자인 패턴이다.


결과적으로 Spring 프레임워크가 인식을 하기 위해선

그러나 표준 구성 요소 대신 하위 클래스를 사용해야한다는 것을 프레임 워크가 어떻게 인식합니까?

해결책은 프레임 워크에서 구성 요소를 구성하는 코드를 단일 팩토리 메서드로 줄이고 구성 요소 자체를 확장하는 것 외에도 누구나이 메서드를 재정의 할 수있게하는 것입니다.



### FactoryBean

### ProxyFactoryBean

### 자동 프록싱

### 프록시가 왜 적용되는데?

### 속도는 당연히 느릴텐데?

### AOP와의 관계

### 마무리

프록시 패턴은 interface를 가지고 부가기능을 추가하기 위한 디자인 패턴이고 인터페이스를 기반으로 하는 디자인 패턴이다 보니  하나의 기능을 추가할 경우 여러 기능을 수정하거나 중복된 부가기능이 존재할 수 있다는 단점이 생기죠
이 단점을 보완하고자 reflection을 사용하여 부가기능을 집어넣는 jdk proxy 방식으로 대체 했는데 인터페이스 없이 깔끔하게 부가기능 객체를 구현할 수 있었지요
헌데 이 방식은 순수 클래스 정보만으로 스프링의 빈으로 활용할 수 없고 따라서 팩토리 빈이나 프록시 팩토리 빈으로 프록시를 생성하여 주입하는데
이 방식엔 부가기능을 추가하는데 핸들러를 포함하여 부수적인 기능들이 추가 개발해야 되서
어떻게하면 autoproxying을 할 수 있을까라고 고민해서 나온게 spring aop
따라서 스프링 aop는 빈 객체를 자동으로 proxy 객체로 생성해주는데
빈을 호출되는 과정에서 인터페이스의 유무에 따라

---

### 참고

- [Spring blog - Transactions, Caching and AOP: understanding proxy usage in Spring](https://spring.io/blog/2012/05/23/transactions-caching-and-aop-understanding-proxy-usage-in-spring)
- [The Aspect Blog](http://www.aspectprogrammer.org/blogs/adrian/2006/01/typed_advice_in.html)
- [The Aspect Blog - Spring 2.0 (Typed Advice in Spring 2.0)](http://www.aspectprogrammer.org/blogs/adrian/2006/01/typed_advice_in.html)
- [Debunking myths: proxies impact performance](https://spring.io/blog/2007/07/19/debunking-myths-proxies-impact-performance)
