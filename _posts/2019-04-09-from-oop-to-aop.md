---
layout: post
title: "OOP에서 AOP"
tags: [AOP]
categories: [Spring, AOP]
subtitle: "Separation of Concerns"
feature-img: "md/img/thumbnail/from-oop-to-aop.png"
thumbnail: "md/img/thumbnail/from-oop-to-aop.png"
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

### OOP의 모듈의 재사용과 다양한 적용

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

먼저 비즈니스 클래스들의 doAction() 메소드안을 살펴보면 실행시간을 측정하는 로직과 비즈니스 로직이 공존하고 있다. AOP의 용어에선 비즈니스 로직을 `핵심 관심사`(Core Concerns)라 하고 그 외 메소드 측정 기능과 같은 부가기능을 `횡단 관심사`(Cross-cutting Concern)라 한다. 이처럼 하나의 객체에 두 관심사가 공존하고 있는 경우 **`비즈니스 로직을 파악하기 어렵고`** 또한 **`부가기능의 관리가 쉽지 않다.`**

#### 1.2. Target Object와 Aspect

따라서 관심사의 기준으로 `핵심 관심사 모듈`(Target Object)과 `횡단 관심사 모듈`(Aspect)로 분리하여 각 관심사에 맞게 관리해야 한다.

![img](/md/img/aop/from-oop-to-aop/class-diagram2.png)

- `****Business.class` → 비즈니스 로직의 모듈화
- `SimplePerformanceMonitor.class` → 실행 시간을 측정하는 로직의 모듈화

이처럼 관심사들을 독립적인 모듈로써 관리만 할 수 있다면야 기존에 제기되었던 문제들을 해결할 수 있을 것이다.

### OOP와 디자인 패턴

하지만 여기서 **＂**`어떻게 Target Object에 Aspect를 적용할 수 있을까?`**”** 라는 해결해야 할 문제가 생긴다. 이 문제에 대해선 일반적으로 사용되고 있는 여러 디자인 패턴을 활용하여 구조적으로 해결하고 부가기능(Advice)이 독립적인 모듈로써 제 기능을 할 수 있는지 같이 살펴보자.

1. 분리된 환경에서 Target Object에 Aspect 적용되는지
2. Advice가 다양한 시점에 적용되는지
3. Aspect를 재사용할 수 있는지

<<<<<<< HEAD
### 2.1. 템플릿 메소드 패턴
=======
#### 2.1. 상속의 템플릿 메소드 패턴
>>>>>>> 69e010b548aa58fccdb4b5fbfe2437df833dbf7a

첫 번째 생각할 수 있는 해결 방안은 상속이다.

상속은 OOP에서 기능을 확장하기 위한 가장 보편적인 접근 방법이 아닐까 생각한다. 이러한 상속의 특징을 활용하여 고안된 디자인 패턴이 바로 템플릿 메소드 패턴이다. GoF Design Pattern 책의 내용을 인용하자면 템플릿 메소드 패턴은 뼈대를 구축하기 위한 패턴이라 정의되어 있다.

>Defines the skeleton of an algorithm in a method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithms structure. - GoF Design Pattern

따라서 템플릿 메소드 패턴은 상위 클래스에서 전체적인 흐름을 잡아주고 상속받은 하위 클래스에선 상세한 로직을 오버라이딩하여 구현하는 방식으로 개발된다.

<img src="/md/img/aop/from-oop-to-aop/template-method1.png" style="max-height:none;">

1. `****Business` 클래스들의 공통된 `doAction()` 메소드는 추상화 메소드로 정의하여 일반화한다.
2. 부가기능의 적용 여부를 `****Business` 클래스에서 정할 수 있도록 상위 클래스에 Hooking 목적을 띈 `isMonitoring()` 메소드를 정의한다.
3. `doActionWithMonitoring()` 메소드에서 메소드 실행시간 측정 부가기능을 적용한다.
4. `****Business` 클래스는 상위 클래스에서 `doAction()` 메소드를 재정의하여 비즈니스 로직을 구현한다.

``` java
abstract public class SimplePerformanceMonitor {
    private final StopWatch stopWatch = new StopWatch();

    // 하위 클래스에게 기능 구현을 맡김
    abstract protected void doAction();

    // Hooking Method 실행시간 측정 여부 목적
    protected boolean isMonitoring() {
      return false;
    }

    // 부가기능을 접목시킨 메소드
    final protected void doActionWithMonitoring() {
        if(isMonitoring()) {
            stopWatch.start();
        }

        this.doAction(); // 비즈니스 로직

        if(isMonitoring()) {
            stopWatch.stop();
            System.out.format("time : %s ms\n", stopWatch.getLastTaskTimeMillis());
        }
    }
}
```

#### 구현 이슈

``` html
담당자 비즈니스 로직 수행중... time : 1008 ms    ← Hooking 메소드 활성화
사원 비즈니스 로직 수행중...                     ← Hooking 메소드 비활성화
```

~~1. 분리된 환경에서 Target Object에 Aspect 적용되는지~~~<br/>

테스트 결과를 보면 첫 번째 이슈는 해결되었다. 하지만 하나의 상위 클래스에서 여러 Advice를 해결하려 했기 때문에 Aspect를 추가하면 할 수록 코드가 복잡해질 수 밖에 없고 Aspect의 탈 부착이 쉽지 않다. 또한, 이처럼 정립된 클래스들을 역으로 일반화해야 하므로 오히려 기존 구조가 복잡해질 수 있다.

**`2. Advice가 다양한 시점에 적용되는지`** 에 대한 부분은 `isMonitoring()` Hooking 메소드를 통해 부가기능의 활성화 여부를 하위 클래스에서 정할 순 있다지만 적용해야 할 Target Object들에 대한 정확한 구조와 공통점들을 파악하기 어렵다면 구현하기 힘들다. 마지막으로 상위 클래스의 `doActionWithMonitoring()` 메소드만 봐도 Adivce와 비즈니스 로직이 공존함으로 **`3.  Aspect를 재사용할 수 있는지`**  에 대한 부분도 부족하다.

일반적으로 상속을 사용한 디자인 패턴은 유연하지 않고 정적인 구조로 되어 있기에 이러한 문제들이 제기될 수밖에 없다. 특히 Aspect는 수정하거나 추가로 또 다른 Aspect를 부착시키는 작업들이 빈번하게 발생할 수밖에 없다. 따라서 본 패턴을 통해 해결할 수 없고 Aspect를 구조적으로 유연하게 관리할 수 있는 디자인 패턴으로 해결해야 한다.

<<<<<<< HEAD
### 2.2. 데코레이터 패턴
=======
### 2.2. 프록시를 기반으로한 패턴들
>>>>>>> 69e010b548aa58fccdb4b5fbfe2437df833dbf7a

Aspect와 같은 부가기능을 동적으로 유연하게 관리하기 위해선 Proxy 기반의 디자인 패턴을 사용하면 구조적으로 해결할 수 있다.

<<<<<<< HEAD
다시 한번 정리를하자면 데코레이터 패턴은 인터페이스를 이용한 디자인 패턴으로써 객체의 책임을 전가를 하는 방식으로 런타임 시 다이나믹게 부가기능들을 덭붙여 기능을 완성시키는 패턴이다. 대표적으로 `java.io` 패키지의 InputStream.class, OutputStream.class가 데코레이터 패턴을 적용한 클래스들이다.
=======
- 데코레이터 패턴 : 부가기능 목적
- 프록시 패턴 : 접근제어 목적
>>>>>>> 69e010b548aa58fccdb4b5fbfe2437df833dbf7a

#### 2.2.1. 데코레이터 패턴

먼저 데코레이터 패턴은 Aspect와 같은 부가기능을 동적으로 유연하게 관리할 수 있도록 고안된 Proxy 기반의 디자인 패턴이다.

<<<<<<< HEAD
실제 Target Object 앞에 부가기능을 정의하는 Aspect 클래스들을 Proxy라 한다.
=======
이 패턴은 인터페이스를 이용한 디자인 패턴으로써 객체의 책임을 전가를 하는 방식으로 기능들을 덭붙여 객체를 완성시키는 패턴이다. 대표적으로 `java.io` 패키지의 InputStream.class, OutputStream.class가 데코레이터 패턴을 적용한 클래스들이다. 이해를 돕기 위해 생활속에서 예를 들자면 하나의 자동차를 생성한다고 가정해보자.
>>>>>>> 69e010b548aa58fccdb4b5fbfe2437df833dbf7a

![img](/md/img/aop/from-oop-to-aop/car-factory-process.png)

독립적으로 각 공정은 맡은 부분만 작업하고 해당 공정의 관심사에 벗어난 작업들은 다음 공정으로 책임을 전가하여 자동차를 완성하는 방식을 따른다. 이와 마찬가지로 데코레이터 패턴도 Aspect를 유연하게 관리하기 위해 각 모듈에서 독립적으로 기능을 덧붙이고 모듈을 점차 발전시키는 구조로 설계된다.

<img src="/md/img/aop/from-oop-to-aop/decorator1.png" style="max-height:none;">

1. 완성시킬 모듈을 인터페이스로 정의한다.
2. 각 데코레이터의 기능의 위임은 생성자나 수정자 메소드를 통해 전달한다.

다음 구조처럼 각 모듈은 자신의 기능을 완성하고 다음 데코레이터 객체로 전달하기 위해 정의한 Business 인터페이스를 생성자나 수정자 메소드를 통해 전달된다. 따라서 데코레이터 패턴은 최종적인 객체를 얻기 위해 객체의 순서를 정해줘야 한다.

<<<<<<< HEAD
  @Override
  public void doAction(){
      // ... 부가기능 구현
      business.doAction(); // AdminBusiness 클래스에게 기능 위임
      //...
  }
=======
데코레이터의 다음 위임 대상은 인터페이스로 선언하고 생성자나 수정자 메소드를 통해 위임 대상을 외부에서 런타임 시에 주입받을 수 있도록 만들어야 한다.
>>>>>>> 69e010b548aa58fccdb4b5fbfe2437df833dbf7a

``` java
public Business createAdminBusiness() {
    Business business = new AdminBusiness(); // 비즈니스 Target Object
    business = new SimplePerformanceMonitor(business); // 실행시간 측정 Aspect
    return business;
}
```

`createAdminBusiness()` 메소드를 통해 데코레이터의 순서를 정해주고 생성된 `Business` 객체의 기능 테스트를 진행해보면 다음과 같은 결과를 얻을 수 있었다.

``` java
@Test
public void isWeaving() {
    Business amdin = createAdminBusiness();
    admin.doAction();
}
```
``` html
SimplePerformanceMonitor.class ← 부가기능 로직 수행
AdminBusiness.class            ← 비즈니스 로직 수행
담당자 비즈니스 로직 수행중... time : 1001 ms
```

_Call → 1) SimplePerformanceMonitor → 2) AdminBusiness → End_

다음 결과를 보면 클라이언트의 관점에선 단순히 비즈니스 클래스의 기능이 호출되는 것처럼 보이지만 사실상 SimplePerformanceMonitor에 의해 AdminBusiness 클래스를 요청된다는 걸 알 수 있다. 이 과정에서
AdminBusiness 클래스를 대신하여 SimplePerformanceMonitor 클래스가 수행한다 해서 SimplePerformanceMonitor 클래스를 **`Proxy`** 라 한다. 즉 Proxy는 클라이언트가 타깃에 접근하는 방법을 제어하거나 타깃에 부가적인 기능을 부여하는 목적을 가지고 있다.

~~1. 분리된 환경에서 Target Object에 Aspect 적용되는지~~<br/>
~~2. Advice가 다양한 시점에 적용되는지~~<br/>

결과적으로 부가기능은 Proxy에서 독립적으로 관리할 수 있고, 추후 Aspect가 추가되더라도 Proxy를 추가하고 Proxy의 순서만 정해주면 최종적인 부가기능이 부착된 비즈니스 객체를 얻을 수 있다.

- 다른 객체에 영향을 주지 않고 동적으로 독립적인 Aspect를 추가할 수 있다.
- 데코레이터 클래스들은 각 기능에만 집중하면 된다.
- 각각의 데코레이터 클래스는 목적이 뚜렷하기 때문에 테스트가 수월하다.
- 적용할 Aspect가 많아져도 데코레이터의 위임의 순서만 정의해주면 유연하게 관리할 수 있다.
- 언제든 Aspect의 철회가 가능하다.
- 각 비즈니스 클래스에 필요에 따라 Aspect를 붙이거나 각각 다르게 적용할 수 있다.

#### 구현 이슈

**3. Aspect를 재사용할 수 있는지**

- 구현해야할 과정들이 번거롭다.
- 대상 위임의 은닉성
- 부가기능의 중복

기존 비즈니스 클래스를 인터페이스로 구성하고 각 데코레이터 클래스들의 대상 위임을 설정하는 과정은 번거로울 수밖에 없다. 또한, 데코레이터 패턴은 인터페이스를 통해 위임하는 방식이기 때문에 코드 레벨에선 위임의 순서를 미리 알 수 없다. 이러한 익명성으로 중복된 부가기능을 생성할 경우가 발생할 수 있다.

### AspectJ를 활용한 AOP 적용
