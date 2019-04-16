---
layout: post
title: "OOP에서 AOP"
tags: [AOP]
categories: [Spring, AOP]
subtitle: "Separation of Concerns"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# AOP의 마법

---

### 들어가기전

컴퓨터 프로그래밍의 패러다임은 늘 변화한다.

_절차적 프로그래밍 → 객체 지향 프로그래밍(OOP) → 관점 지향 프로그래밍(AOP)_

새로운 패러다임은 이전 패러다임의 한계를 느끼고 이를 개선한 방식을 제시한다. 그렇다면 완벽하기만 했던 OOP가 어떠한 기술적인 한계가 있길래 AOP(Aspect Oriented Programming)가 등장했는지 궁금증이 생겼다.

AOP는 흔히 마법과 같다고 말한다. 본 포스팅이 그 이유에 대한 해답이 되길 바란다.

### 학습목표

1. OOP의 기술적인 한계
2. AOP의 사용성의 이해

### 1. 모듈의 재사용과 다양한 적용

하나의 프로세스는 하나의 기능이라는 개발 관점이 지배적이던 시절에 OOP의 등장은 혁신과 같았다.

절차적 프로그래밍은 동작의 관점에서 프로그래밍하므로 데이터의 흐름이 굉장히 중요했다. 반면 OOP는 프로그램의 프로세스와는 별개로 데이터 추상화 작업을 통해 동작과 데이터를 객체라는 하나의 모듈로 정의하고 이를 재사용함으로써 프로그래밍을 한다. 자연스레 객체 간의 구조가 OOP에선 중요하고, 흔히 낮은 결합 도와 높은 응집도를 가진 구조가 이상적인 구조라 말한다.

이러한 이상적인 구조를 설계하기 위해 고안된 여러 디자인 원칙과 패턴이 존재한다. 그중에 [관심사의 분리(Separation of Concerns)](https://en.wikipedia.org/wiki/Separation_of_concerns)라는 디자인 원칙은 OOP의 근간을 잘 지키는 원칙이라할 수 있다.

``` java
public class Business{
    public void doAction(){
        System.out.println("");
        ... logic
        System.out.println("");
    }
}
```

1. 기능의 재사용
2. 기능의 독립성

이 관심사의 분리를 잘 사용하면 OOP의 장점들이다.

따라서 분리된 관심사를 어떻게 효율적으로 기존 로직에 추가하는 방법을 알아야한다.


이 개념을 통해 OOP의 한계와 더불어 AOP를 사용해야 하는 이유를 살펴볼 수 있다.

#### 클래스 구성

![img](/md/img/aop/oop-with-aop/class-diagram1.png)

1. AdminBusiness → 업무 담당자 클래스
2. MemberBusiness → 일반 사원 클래스

다음과 클래스의 구조를 형성하고 있다.

이때 doAction() 메소드는 기존

1. 이체 금액은 1000원 이상 부터 가능합니다.

다음 이체 로직은 문제가 없다고 가정하고 이때 다음과 같은 요구사항을 추가해보자.

### 1. 템플릿 메소드 패턴

### 2. 메소드 팩토리 패턴

### 3. 데코레이션 패턴

코드가 작동해야하는 객체의 정확한 유형과 종속성을 미리 알지 못하는 경우 팩토리 메서드를 사용하십시오.

 AOP의 메커니즘은 프로그램을 관심사(Concerns) 기준으로 크게 핵심 관심사와 횡단 관심사로 분류하고 있다.

핵심 관심사는 프로그램의 핵심 가치와 목적이 그대로 드러난 관심 영역을 뜻한다. 해당 프로그램의 비즈니스 기능이 그러하다. 반면 횡단 관심사는 비즈니스 기능과는 다른 관심 영역을 뜻한다.

### 4. 프록시 패턴

### AOP
