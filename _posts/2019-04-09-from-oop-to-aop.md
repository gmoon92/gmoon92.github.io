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
#display: "false"
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

### 1. OOP의 모듈의 재사용과 다양한 적용

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
3.  Aspect를 재사용할 수 있는지

### 2.1. 템플릿 메소드 패턴

첫 번째 생각할 수 있는 해결 방안은 상속이다.

상속은 OOP에서 기능을 확장하기 위한 가장 보편적인 접근 방법이 아닐까 생각한다. 이러한 상속의 특징을 활용하여 고안된 디자인 패턴이 바로 템플릿 메소드 패턴이다. GoF Design Pattern 책의 내용을 인용하자면 템플릿 메소드 패턴은 뼈대를 구축하기 위한 패턴이라 정의되어 있다.

>Defines the skeleton of an algorithm in a method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithms structure. - GoF Design Pattern

따라서 템플릿 메소드 패턴은 상위 클래스에서 전체적인 흐름을 잡아주고 상속받은 하위 클래스에선 상세한 로직을 오버라이딩하여 구현하는 방식으로 개발된다.

<img src="/md/img/aop/from-oop-to-aop/class-diagram3.png" style="max-height:none;">

1. `****Business` 클래스들의 공통된 `doAction()` 메소드는 추상화 메소드로 정의하여 일반화한다.
2. 부가기능의 적용 여부를 `****Business` 클래스에서 정할 수 있도록 상위 클래스에 Hooking 목적을 띈 `isMonitoring()` 메소드를 정의한다.
3. `doActionWithMonitoring()` 메소드에서 메소드 실행시간 측정 부가기능을 적용한다.
4. `****Business` 클래스는 상위 클래스에서 `doAction()` 메소드를 재정의하여 비즈니스 로직을 구현한다.

``` java
abstract public class SimplePerformanceMonitor {
    private final StopWatch stopWatch = new StopWatch();

    abstract protected void doAction();

    // Hooking Method 실행시간 측정 여부 목적
    protected boolean isMonitoring() {
      return false;
    }

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

이처럼 일반적으로 상속을 사용한 디자인 패턴은 유연하지 않고 정적인 구조로 되어 있다. 특히 Aspect의 특성상 추후 수정하거나 추가로 다른 Aspect를 부착시키는 동적인 작업들이 빈번하게 발생할 수밖에 없다. 따라서 본 패턴을 통해 해결할 수 없고 Aspect를 구조적으로 유연하게 관리할 수 있는 디자인 패턴을 찾아야 한다.

### 2.2. 데코레이터 패턴

데코레이터 패턴은  Aspect를 동적으로 유연하게 관리할 수 있는 디자인 패턴이다. `토비의 스프링`에서도 데코레이터 패턴을 자세히 소개할 만큼 이 패턴에 대해선 다들 잘 아실꺼라 생각한다.

다시 한번 정리를하자면 데코레이터 패턴은 인터페이스를 이용한 디자인 패턴으로써 객체의 책임을 전가를 하는 방식으로 런타임 시 다이나믹게 부가기능들을 덭붙여 기능을 완성시키는 패턴이다. 대표적으로 `java.io` 패키지의 InputStream.class, OutputStream.class가 데코레이터 패턴을 적용한 클래스들이다.

런타입 시 다이내믹하게 부여해주기 위해 프록시를 사용하는 패턴을 말한다. 프록시로서 동작하는 각 데코레이터는 위임하는 대상에도 인터페이스로 접근하기 때문에 자신이 최종 타깃으로 위임하는지 아니면 다음 단계의 데코레이터 프록시로 위임하는지 알지 못한다.

데코레이터의 다음 위임 대상은 인터페이스로 선언하고 생성자나 수정자 메소드를 통해 위임 대상을 외부에서 런타임 시에 주입받을 수 있도록 만들어야 한다.

데코레이터 패턴은 인터페이스를 통해 위임하는 방식이기 때문에 어느 데코레이터에서 타깃으로 연결될지 코드 레벨에선 미리 알 수 없다.

실제 Target Object 앞에 부가기능을 정의하는 Aspect 클래스들을 Proxy라 한다.

_Call → 1) Business → 2) SimplePerformanceMonitor → 3) AdminBusiness → End_

1. Business 데코레이터 인터페이스 정의
2. SimplePerformanceMonitor 에서 메소드 실행시간 측정 로직 구현, 다음 단계에서 호출될 객체에 기능을 위임
3. AdminBusiness에서 비즈니스 로직 구현

``` java
public interface Business{
  public void doAction();
}

// Proxy
public class SimplePerformanceMonitor implements Business{
  private final Business business;

  public SimplePerformanceMonitor(Business business){
      this.business = business;
  }

  @Override
  public void doAction(){
      // ... 부가기능 구현
      business.doAction(); // AdminBusiness 클래스에게 기능 위임
      //...
  }

}
public class AdminBusiness implements Business {
    @Override
    public void doAction(){
        //...비즈니스 로직 구현
    }
}
```

- 데코레이터 패턴의 구조에서 각 클래스들은 각 기능에만 집중하면 된다.
- 기능이 독립된 클래스 구조 덕분에 테스트가 수월해진다.
- 다른 객체에 영향을 주지 않고 동적으로 독립적인 Aspect를 추가할 수 있다.
- 적용할 Aspect가 많아져도 DI 순서만 정의해주면 유연하게 관리할 수 있다.
- 언제든 Aspect의 철회가 가능하다.
- 각 비즈니스 클래스에 필요에 따라 Aspect를 붙이거나 각각 다르게 적용할 수 있다.

#### 구현 이슈

#### 2.3. 프록시 패턴

데코레이터 패턴도 이러한 Proxy를 기반으로 설계된 디자인 패턴이지만 다르다.

프록시 패턴은 클라이언트가 직접적인 타깃에 접근하지 못하도록 제어의 목적을 가진 디자인 패턴이다.

프록시 패턴의 프록시는 타깃의 기능을 확장하거나 추가하지 않는다. 대신 클라이언트가 타깃에 접근하는 방식을 변경해준다.

  타깃 오브젝트를 생성하기가 복잡하거나 당장 필요하지 않은 경우에는 꼭 필요한 시점까지 오브젝트를 생성하지 않는 편이 좋다. 그런데 타깃 오브젝트에 대한 레퍼런스가 미리 필요할 수 있을때 이 패턴을 적용하면 된다.

클라이언트에게 타깃에 대한 레퍼런스를 넘겨야 하는데, 실제 타깃 오브젝트는 만드는 대신 프록시를 넘겨주는 것이다. 그리고 프록시의 메소드를 통해 타깃을 사용하려고 시도하면, 그때 프록시가 타깃 오브젝트를 생성하고 요청을 위임해주는 방식이다.






Proxy의 목적

1. 클라이언트가 타깃에 접근하는 방법을 제어
2. 타깃에 부가적인 기능을 부여하기 위함

####  Pattern Issues

#### 2.3. 메소드 팩토리 패턴

#### 구현 이슈

코드가 작동해야하는 객체의 정확한 유형과 종속성을 미리 알지 못하는 경우 팩토리 메서드를 사용하십시오.

![img](/md/img/aop/from-oop-to-aop/class-diagram4.png)

``` java
abstract public class BusinessMonitorFactory{
	private final StopWatch stopWatch = new StopWatch();

	abstract protected Business newInstance();
	final protected void doActionWithMonitoring() {
		stopWatch.start();

		try {
			newInstance().doAction();
		}catch(NullPointerException e) {
			e.printStackTrace();
			throw new RuntimeException("Business is null...");
		}catch(Exception e) {
			e.printStackTrace();
			throw new RuntimeException("throws ...");
		}

		stopWatch.stop();
		System.out.format("time : %s ms\n", stopWatch.getLastTaskTimeMillis());
	}
}

public class SimplePerformanceMonitor extends BusinessMonitorFactory{
	private Object targetObj;

	@Override
	protected Business newInstance() {
		try {
			return (Business) targetObj;
		}catch (Exception e) {
			e.printStackTrace();
			throw new RuntimeException("is Business.newInstance Error...");
		}
	}

	public SimplePerformanceMonitor setTargetObj(Object targetObj) {
		this.targetObj = targetObj;
		return this;
	}
}

public class FactoryMethodTest {

	private ApplicationContext context;
	private Business adminBusiness;
	private Business memberBusiness;

	@Before
	public void init() {
		context = new AnnotationConfigApplicationContext(MyApplication.class);
		adminBusiness  = context.getBean(AdminBusiness.class, "AdminBusinessDesignOfFM");
		memberBusiness = context.getBean(MemberBusiness.class, "MemberBusinessDesignOfFM");
	}

	@Test
	public void isApplyOfAspect() {
		BusinessMonitorFactory factory = new SimplePerformanceMonitor().setTargetObj(adminBusiness);
		factory.doActionWithMonitoring();
	}
}
```

``` html
담당자 비즈니스 로직 수행중... time : 1001 ms
```


### AOP
