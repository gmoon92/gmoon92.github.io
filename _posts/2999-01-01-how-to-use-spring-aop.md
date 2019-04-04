---
layout: post
title: "Spring에서 AOP 선언의 선택"
tags: [AOP, Spring, AspectJ, Srpring-AOP]
categories: [Spring, AOP]
subtitle: "XML & @AspectJ & AspectJ"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# XML & @AspectJ & AspectJ

---

### 들어가기전

_**사용자 서비스에 실행 시간을 측정해서 알려주세요.**_

다음 요구사항이 처리하는 데 있어 Aspect가 가장 좋은 접근이라고 결정했다면, 다양한 구현방식 중에 어떻게 구현해야 할지 선택해야 한다.

본 포스팅에선 Spring AOP에서 지원하는 Aspect 구현 방식과 이 과정에서 간단하게 메소드 실행 시간을 측정하는 Profiling 목적을 띈 클래스를 구현하는 과정을 학습해보는 시간을 가지려 한다.

### 학습목표

1. XML스키마 기반 방식의 이해
2. @AspectJ 어노테이션 방식의 이해
3. AspectJ 방식의 이해

### 비즈니스 로직 구현

필자는 JDK Dynamic Proxy를 사용하여 aspect를 구현해보려 한다. JDK Dynamic Proxy와 CGLIB의 차이점에 대해선 다음에 자세히 다룰 예정이다.

본론으로 돌아와서, 아시다시피 JDK의 Proxy는 인터페이스만 지원하기 때문에, 먼저 작성 비즈니스 인터페이스를 구현해보자.

#### 인터페이스 구현

``` java
package com.learning.busniess;

public interface Business {
	public void doAction() throws Exception;
	public void doRuntimeException();
}
```

다음 두 메소드는 각 학습 목적을 띄고 있다.

- doAction → 비즈니스 로직 수행
- doRuntimeException  →  강제 예외 발생

첫 번째 메소드는 비즈니스 로직의 실행 시간을 측정하기 위함이고, 두 번째 메소드는 예외가 발생할 때 어떻게 처리가 될지 학습 테스트를 진행하기 위해 고려했다.

#### Target Object 구현

BusinessImple 클래스는 Advice(부가 기능)가 적용될 Target Object이다.

``` java
package com.learning.business;

import org.springframework.stereotype.Component;

@Component
public class BusinessImple implements Business{

	@Override
	public void doAction() throws Exception {
		Thread.sleep(500); // 실행 0.5초 지연
	}

	@Override
	public void doRuntimeException() {
		throw new RuntimeException("에러가 발생하였습니다.");
	}
}
```

다음 코드를 보면 메소드 실행 시간 측정을 테스트하기 위해 sleep 메소드를 사용하여 doAction 메소드 수행 시간을 지연시켰고 doRuntimeException 메소드는 강제 예외를 발생시켰다.

여기서 중요한 점은 메소드의 실행 시간을 측정과 관련된 어떠한 로직도 비즈니스 클래스 내에서 작성하지 않았다는 점이다. 이 점은 AOP가 주는 가장 큰 이점이라 생각한다.

#### 메소드 실행 측정 클래스 작성

이제 부가 기능 클래스를 작성해보자.

``` java
package com.learning.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;
import org.springframework.util.StopWatch.TaskInfo;

public class SimpleMethodStopWatch {
	public Object profiling(ProceedingJoinPoint pjp) throws Throwable{
		StopWatch stopWatch = new StopWatch();
		stopWatch.start(pjp.toShortString()); // Timer 시작

		try {
			return pjp.proceed(); // Target Object 메소드 실행
		} catch (RuntimeException e) {
			endStopWatch(stopWatch, e);
			throw e;
		} finally {
			endStopWatch(stopWatch, null);
		}
	}

	// Timer 종료 메소드
	private void endStopWatch(StopWatch stopWatch, Throwable throwable){
		stopWatch.stop();

		TaskInfo taskInfo = stopWatch.getLastTaskInfo();
		String taskName = taskInfo.getTaskName();
		long time = taskInfo.getTimeMillis();

		String errorMessage = (throwable == null ? "" : ", ERROR > " + throwable.getMessage());

		System.out.format("%s : %d ms %s \n", taskName, time, errorMessage);
	}
}
```

- profiling → Advice 작성
- endStopWatch → 비즈니스 메소드 종료를 알리는 메소드

다음 `profiling` 메소드의 매개 변수를 보면 ProceedingJoinPoint 객체를 Spring AOP로부터 받는다는 걸 알 수 있다.

ProceedingJoinPoint 객체는 JoinPoint를 확장한 클래스로써 호출될 Target Object(BusinessImple.class)의 메소드 정보뿐만 아니라 ProceedingJoinPoint.proceed() 메소드를 통해 Target의 메소드의 호출 시점과 타깃의 매개 변수의 값을 제어할 수 있기 때문이다.

비즈니스 메소드의 실행 시간을 측정하기 위해 `org.springframework.util.StopWatch` 기능을 사용하였다.

### AOP 방식 선택

Spring AOP에선 기본적으로 두 가지 방식을 통해 Aspect를 구현할 수 있다.

1. [XML(스키마 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-schema)
2. [@AspectJ(어노테이션 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)

#### XML스키마 기반

먼저 XML 기반의 형식으로 aspect를 정의하기 위해선 Spring이 제공하는 `aop`라는 Namespace 태그를  사용해야 한다.

 `aop` 태그 중에서 `<aop:config>`는 최상위 요소로써, aop와 관련된 aspect와 advisor는 반드시 `<aop:config>` 요소 내부에 선언되어야 한다.

``` xml
<aop:config>
  <aop:aspect  id="customLogging"  ref="performanceMonitor" > <!-- 부가 기능 빈 참조 -->
      ...
  </aop:aspect>
</aop:config>

<bean  id="performanceMonitor"  class="..." > <!-- 부가 기능 빈 정의 -->
  ...
</bean>
```

aspect는 `<aop:aspect>` 태그를 사용하여 선언하고, Spring에 등록된 부가 기능 빈을 `ref` 속성을 사용하여 참조하면 된다. `ref`가 참조하는 bean은 부가 기능을 지원하는 bean은 물론 기존에 사용된 Spring bean처럼 구성되고 DI 할 수 있다.


#### @AspectJ

#### 마무리

프로젝트의 시간이 많이 지난 시점에서 추가 요구사항이 처리하는 데 있어 난감한 경우가 많다.

문제없이 잘 돌아가는 시스템에 기존 코드를 수정해야 하는 상황이 발생한다면 개발자로선 부담된다. 특히 추가 기능이 다수의 기존 기능에 추가되어야 하는 상황이라면 개발자에겐 최악의 시나리오다.

물론 이러한 상황을 배제하더라도 기존 코드를 분석하는 시간과 테스트 시간 및 요구사항을 구현하는 데 있어 예기치 못한 오류가 발생하여 상당한 시간이 소요될 수 있다.

이러한 문제점들을 최소화하고 부가 기능을 추가하는 방법으로 관점(Aspect)으로 접근하는 방법이 가장 나은 접근일 수 있다.

하지만 Aspect와 관계없이 비즈니스 로직을 수정할 때도 무리하게 Aspect를 접근하는 방식은 오히려 관리 포인트가 높아지기 때문에 상황을 고려하여 신중히 적용해야 한다고 생각한다.

### 참고

- [AOP 구현 세가지 방법 비교](https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ)

  - [Spring-DOC : 11. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
