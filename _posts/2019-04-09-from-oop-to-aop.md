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

### 모듈의 재사용과 다양한 적용

하나의 프로세스는 하나의 기능이라는 개발 관점이 지배적이던 시절에 OOP의 등장은 혁신과 같았다.

절차적 프로그래밍은 동작의 관점에서 프로그래밍하므로 데이터의 흐름이 굉장히 중요했다. 반면 OOP는 프로그램의 프로세스와는 별개로 데이터 추상화 작업을 통해 동작과 데이터를 객체라는 하나의 모듈로 정의하고 이를 재사용함으로써 프로그래밍을 한다. 자연스레 객체 간의 구조가 중요하고, 흔히 OOP에선 낮은 결합 도와 높은 응집도를 가진 구조가 이상적인 구조라 말한다.

#### 1.1. 관심사의 분리

이러한 이상적인 구조를 설계하기 위해 고안된 여러 디자인 원칙과 패턴이 존재한다. 그중에 [관심사의 분리(Separation of Concerns)](https://en.wikipedia.org/wiki/Separation_of_concerns)라는 디자인 원칙은 프로그래밍에 있어 OOP의 근간을 잘 지키는 원칙이라할 수 있다. 그렇다면 `관심사의 분리`란 무엇일까. 다음 클래스의 구조를 살펴보자.

![img](/md/img/aop/from-oop-to-aop/class-diagram1.png)

``` java
public class ****Business {
    private StopWatch stopWatch = new StopWatch();

    public void doAction() {
        stopWatch.start();

        // ... 비즈니스 로직 생략

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint()); // 시간 측정
    }
}
```

- doAction() → 비즈니스 로직(핵심 관심사) + 실행시간 측정 로직(횡단 관심사)

먼저 비즈니스 클래스들의 doAction() 메소드안을 살펴보면 실행시간을 측정하는 로직과 비즈니스 로직이 공존하고 있다. AOP의 용어에선 비즈니스 로직을 `핵심 관심사`(Core Concerns)라 하고 그 외 메소드 측정 기능과 같은 부가 기능을 `횡단 관심사`(Cross-cutting Concern)라 한다. 이처럼 하나의 객체에 두 관심사가 공존하고 있는 경우 `1) 비즈니스 로직을 파악하기 어렵고` 또한 `2) 부가 기능의 관리가 쉽지 않다.`

#### 1.2. Target Object와 Aspect

따라서 관심사의 기준으로 `핵심 관심사 모듈`(Target Object)과 `횡단 관심사 모듈`(Aspect)로 분리하여 각 관심사에 맞게 관리해야 한다.

![img](/md/img/aop/from-oop-to-aop/class-diagram2.png)

- ****Business.class → 비즈니스 로직의 모듈화
- SimplePerformanceMonitor.class → 실행 시간을 측정하는 로직의 모듈화

이처럼 관심사들을 독립적인 모듈로써 관리만 할 수 있다면야 기존에 제기되었던 문제들을 해결할 수 있을 것이다.

~~1. 비즈니스 로직을 파악하기 어렵다.~~<br/>
~~2. 부가 기능의 관리가 어렵다.~~

### OOP와 디자인 패턴

하지만 여기서 **＂**`어떻게 Target Object에 Aspect를 적용할 수 있을까?`**”** 라는 해결해야 할 문제가 생긴다. 이 문제에 대해선 일반적으로 사용되고 있는 여러 디자인 패턴을 활용하여 해결해보려 한다.

#### 2.1. 템플릿 메소드 패턴

첫 번째 생각할 수 있는 해결 방안은 상속이다. 상속은 OOP에서 기능을 확장하기 위한 가장 보편적인 접근 방법이 아닐까 생각한다. 이러한 상속의 특징을 활용하여 고안된 디자인 패턴이 바로 템플릿 메소드 패턴이다.

템플릿 메소드 패턴을 적용하면 다음과 같은 구조가 형성된다.

![img](/md/img/aop/from-oop-to-aop/class-diagram3.png)

- isMonitoring() : Hooking 목적
- doActionWithMonitoring() : 부가 기능 정의

``` java
abstract protected class SimplePerformanceMonitor {
    private final StopWatch stopWatch = new StopWatch();

    final protected void doActionWithMonitoring() {
        if(isMonitoring()) {
            stopWatch.start();
        }

        doAction(); // 비즈니스 로직

        if(isMonitoring()) {
            stopWatch.stop();
            System.out.format("time : %s ms\n", stopWatch.getLastTaskTimeMillis());
        }
    }

    protected boolean isMonitoring() {
        return false;
    }

    abstract protected void doAction();
}
```
``` java
@Component("MemberBusinessDesign1")
public class MemberBusiness extends SimplePerformanceMonitor{

    @Override
    public void doAction() {
        try {
            System.out.format("사원 비즈니스 로직 수행중... ");
            Thread.sleep(500);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

@Component("AdminBusinessDesign1")
public class AdminBusiness extends SimplePerformanceMonitor{

    @Override
    public void doAction() {
        try {
            System.out.format("담당자 비즈니스 로직 수행중... ");
            Thread.sleep(1000);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public boolean isMonitoring() {
        return true; // 모니터링 활성화
    }
}

```

### 테스트 검증

``` java
public class TemplateMethodTest {

    private ApplicationContext context;

    private AdminBusiness  adminBusiness;
    private MemberBusiness memberBusiness;

    @Before
    public void init() {
        context = new AnnotationConfigApplicationContext(MyApplication.class);
        adminBusiness  = context.getBean(AdminBusiness.class, "AdminBusinessDesign1");
        memberBusiness = context.getBean(MemberBusiness.class, "MemberBusinessDesign1");
    }

    @Test
    public void isApplyOfAspect() {
        adminBusiness.doActionWithMonitoring();
        memberBusiness.doActionWithMonitoring();
    }
}
```
``` html
담당자 비즈니스 로직 수행중... time : 1008 ms
사원 비즈니스 로직 수행중...
```


#### 2.2. 메소드 팩토리 패턴

#### 2.3. 데코레이션 패턴

코드가 작동해야하는 객체의 정확한 유형과 종속성을 미리 알지 못하는 경우 팩토리 메서드를 사용하십시오.

 AOP의 메커니즘은 프로그램을 관심사(Concerns) 기준으로 크게 핵심 관심사와 횡단 관심사로 분류하고 있다.

핵심 관심사는 프로그램의 핵심 가치와 목적이 그대로 드러난 관심 영역을 뜻한다. 해당 프로그램의 비즈니스 기능이 그러하다. 반면 횡단 관심사는 비즈니스 기능과는 다른 관심 영역을 뜻한다.

#### 2.4. 프록시 패턴

### AOP
