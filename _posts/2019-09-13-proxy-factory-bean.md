---
layout: post
title: "아드리안"
tags: [AOP, Spring, SpringAOP, XML, AspectJ]
categories: [Spring, AOP]
subtitle: "Spring AOP로 메소드 시간 측정하기"
feature-img: "md/img/thumbnail/spring-aop-choosing.png"
thumbnail: "md/img/thumbnail/spring-aop-choosing.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# 아드리안 콜리어의 OOP로 해결한 AOP

---
[Adrian Colyer](https://nofluffjuststuff.com/conference/speaker/adrian_colyeramming)

### 들어가기전

프록시 패턴은 interface를 가지고 부가기능을 추가하기 위한 디자인 패턴이고 인터페이스를 기반으로 하는 디자인 패턴이다 보니  하나의 기능을 추가할 경우 여러 기능을 수정하거나 중복된 부가기능이 존재할 수 있다는 단점이 생기죠
이 단점을 보완하고자 reflection을 사용하여 부가기능을 집어넣는 jdk proxy 방식으로 대체 했는데 인터페이스 없이 깔끔하게 부가기능 객체를 구현할 수 있었지요
헌데 이 방식은 순수 클래스 정보만으로 스프링의 빈으로 활용할 수 없고 따라서 팩토리 빈이나 프록시 팩토리 빈으로 프록시를 생성하여 주입하는데
이 방식엔 부가기능을 추가하는데 핸들러를 포함하여 부수적인 기능들이 추가 개발해야 되서
어떻게하면 autoproxying을 할 수 있을까라고 고민해서 나온게 spring aop
따라서 스프링 aop는 빈 객체를 자동으로 proxy 객체로 생성해주는데
빈을 호출되는 과정에서 인터페이스의 유무에 따라


### 학습목표


### 참고

- [Spring blog - Transactions, Caching and AOP: understanding proxy usage in Spring](https://spring.io/blog/2012/05/23/transactions-caching-and-aop-understanding-proxy-usage-in-spring)
- [Debunking myths: proxies impact performance](https://spring.io/blog/2007/07/19/debunking-myths-proxies-impact-performance)
- [Refactoring.guru - proxy](https://refactoring.guru/design-patterns/proxy)
