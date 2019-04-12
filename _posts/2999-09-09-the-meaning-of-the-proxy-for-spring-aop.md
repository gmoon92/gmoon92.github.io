---
layout: post
title: "Proxy를 선택한 Spring AOP"
tags: [AOP, Spring, SpringAOP, Proxy]
categories: [Spring, AOP]
subtitle: "난 프록시가 뭔지 모르는데..."
feature-img: "md/img/thumbnail/spring-aop-choosing.png"
thumbnail: "md/img/thumbnail/spring-aop-choosing.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# 난 프록시가 뭔지 모르는데...

---

### 들어가기전

사실 전에 작성한 [Spring AOP 선언의 선택](https://gmun.github.io/spring/aop/2019/03/01/spring-aop-choosing.html)의 포스팅에서  **"** *난 프록시가 뭔지 모르는데... 잘 읽었어요.* **"** 라는 피드백을 많이 받았다.

나 역시도, Proxy는 Spring AOP를 공부하면서 가장 이해하기 어려웠던 개념이었다. Spring AOP의 레퍼런스만 봐도 Runtime weaving, autoproxying, JDK Dynamic Proxy 등 수 많은 용어와 개념들을 학습하는데 있어 Proxy에 대한 개념은 필수적이다.

이는 Spring AOP는 Proxy를 기반으로 동작하는 AOP 프레임워크이기 때문인데, 제대로 Proxy에 대해 이해하지 못하면 Spring AOP를 이해할 수 없다.

이처럼 Proxy와 Spring AOP는 떼려야 뗄 수 없는 관계다. 본 포스팅을 통해 Proxy에 대해 이해하고 더 나아가 추후 Weaving에 관한 포스팅을 이해하는 데 도움이 되길 바란다.

### 학습목표

- 관심사의 분리에 대한 여러 방법
- Spring AOP과 Proxy의 관계에 대한 이해
- 이제 weaving에 대한 관심이 생겼다면 OK

### 관심사의 이슈

``` java

```
Proxy라는 단어를 이


```
proxy
```

- 등장배경에서 필요성이 약해보임
- 기술적인 대체인지 신규인지에 대한 관점으로 부연 설명

- 기존 기술과의 차이점이나 동적/정적인 구현 방법에 대한 설명

- 기능/적용범위에 대한 장점이나 단점

- 효용성이 조금 더 극적으로 설명할 필요가 있음

- 상속 관계에서는 쉽게 구현 가능하지만 불가능 한 경우에 대한 상황 설명
- 핵심 관심사에 집중했을때 이점
- Aspect, Advice, Pointcut, JoinPoint는 예제를 중심으로 설명이 필요


음... 왜 이 기술이 좋은가 내가 왜 써야하는가는 간단한 소스를 써서 advice를 소개하지 않으면 공감하기 어려울 수도 있을 듯

배상용   [3 hours ago]
이를 위한 개념도라도 있어야 할 것 같음

- 전체 구조가 클래스 다이어그램으로 나오면 좋겠음, 그러면서 포인트컷 조건과 실제 조인포인트에 대한 지점을 함께 표시
- 실제 구현 이후 쉽게 제거할 수 있음을 설명
- 기 구현된 코드를 다른 포인터컷으로 쉽게 적용하는 단계 기술

기존 코드: 로그인, 재고조회
부연 모듈: 시간재기
어드바이스1: 로그인에 적용
어드바이스2: 재고조회에 적용


#### Java의 단점


### 데코레이션 패턴

### 프록시 패턴

### Java Reflection

### JDK Dynamic Proxy

[Adrian Colyer](https://nofluffjuststuff.com/conference/speaker/adrian_colyeramming)


---


### 참고

- [Spring blog - Transactions, Caching and AOP: understanding proxy usage in Spring](https://spring.io/blog/2012/05/23/transactions-caching-and-aop-understanding-proxy-usage-in-spring)
- [Debunking myths: proxies impact performance](https://spring.io/blog/2007/07/19/debunking-myths-proxies-impact-performance)
- [Refactoring.guru - proxy](https://refactoring.guru/design-patterns/proxy)
