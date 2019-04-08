---
layout: post
title: "Spring에서 AOP 선언의 선택"
tags: [AOP, Spring, AspectJ, Srpring-AOP]
categories: [Spring, AOP]
subtitle: "XML & @AspectJ"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# XML & @AspectJ

---

### 들어가기전

_**사용자 서비스에 실행 시간을 측정해서 알려주세요.**_

다음 요구사항이 처리하는 데 있어 Aspect가 가장 좋은 접근이라고 결정했다면, 다양한 구현방식 중에 어떻게 구현해야 할지 선택해야 한다.

본 포스팅에선 Spring AOP에서 지원하는 Aspect 구현 방식과 이 과정에서 간단하게 메소드 실행 시간을 측정하는 Profiling 목적을 띈 클래스를 구현하는 과정을 학습해보는 시간을 가지려 한다.

> [예제 코드](https://github.com/gmun/spring-aop-simple-profiling-in-xml-config)

### 학습목표

1. XML스키마 기반 방식의 이해
2. @AspectJ 어노테이션 방식의 이해

### Spring AOP dependencies 설정

Spring AOP는 [aspectjweaver.jar](https://mvnrepository.com/artifact/org.aspectj/aspectjweaver)가 필요하기 때문에 Maven의 빌드 설정 파일인 pom.xml 파일에 아래 코드를 추가하자.

``` xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.2</version>
</dependency>
```

### 비즈니스 로직 구현

필자는 JDK Dynamic Proxy를 사용하여 Aspect를 구현해보려 한다. JDK Dynamic Proxy와 CGLIB의 차이점에 대해선 다음에 자세히 다룰 예정이다.

본론으로 돌아와서, 아시다시피 JDK의 Proxy는 인터페이스만 지원하기 때문에, 먼저 작성 비즈니스 인터페이스를 구현해보자.

#### 인터페이스 구현

``` java
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
public class SimplePerformanceMonitor {

    public Object monitoring(ProceedingJoinPoint joinPoint) throws Throwable{
        StopWatch stopWatch = new StopWatch("Simple Monitoring");
        stopWatch.start(joinPoint.toShortString()); // Timer 시작

        try {
            return joinPoint.proceed(); // Target Object 메소드 실행
        } catch (RuntimeException e) {
            endStopWatch(stopWatch, e);
            throw e;
        } finally {
            endStopWatch(stopWatch, null);
        }
    }

    // Timer 종료 메소드
    private void endStopWatch(StopWatch stopWatch, Throwable throwable){
        stopWatch.stop(); // Timer 종료

        TaskInfo taskInfo = stopWatch.getLastTaskInfo(); // 작업정보
        String taskName = taskInfo.getTaskName(); // 작업 명
        long time = taskInfo.getTimeMillis(); // 작업 시간

        String errorMessage = (throwable == null ? "" : ", ERROR > " + throwable.getMessage());

        System.out.format("%s : %d ms %s \n", taskName, time, errorMessage);
    }
}
```

- monitoring → Advice 작성
- endStopWatch → 비즈니스 메소드 종료를 알리는 메소드

다음 `monitoring` 메소드의 매개 변수를 보면 ProceedingJoinPoint를 Spring AOP로부터 받는다는 걸 알 수 있다.

ProceedingJoinPoint를 사용한 이유는 JoinPoint를 확장한 클래스로써 호출될 Target Object(BusinessImple.class)의 메소드 정보뿐만 아니라 `ProceedingJoinPoint.proceed()` 메소드를 통해 Target의 메소드의 호출 시점과 타깃의 매개 변수의 값을 제어할 수 있기 때문이다.

실행 시간을 측정은 `org.springframework.util.StopWatch` 클래스를 통해 기능을 구현하였다.

### Spring AOP 방식 선택

부가 기능까지 구현했다면 Spring AOP를 통해 서비스 메소드의 프로세스 시간을 측정하는 데 사용하려는 `SimplePerformanceMonitor` Aspect 클래스와 `Business` 클래스에 접목시켜야한다.

일반적으로 JDK Dynamic Proxy를 구현하기 위해 `ProxyFactoryBean` 또는 `FactoryBean`을 사용하여 직접적으로 Aspect 구현할 수 있지만, Spring은 간편하게 AOP를 설정할 수 있도록 두 가지 방식을 제공하고 있다.

1. [XML(스키마 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-schema)
2. [@AspectJ(어노테이션 기반 접근)](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)

### XML 스키마 기반

먼저 XML 기반의 형식으로 Aspect를 정의하기 위해선 Spring이 제공하는 `aop`라는 Namespace 태그를 사용해야 한다.

 `aop` Namespace 태그 중에서 `<aop:config>`는 최상위 요소로써 XML 설정에 있어 가장 먼저 선언이 되어야 하고, AOP와 관련된 Aspect와 Advisor는 반드시 `<aop:config>` 요소 내부에 선언되어야 한다.

```xml
<aop:config>
    <aop:aspect>...</aop:aspect> <!-- aspect -->
    <aop:advisor>...</aop:advisor> <!-- advisor -->
</aop:config>
```

일반적으로 Spring AOP는 Spring IoC 컨테이너와 함께 사용된다. 따라서 Spring에서 Aspect는 애플리케이션 컨텍스트에서 bean으로 정의된 일반 Java Object일 뿐이다. 따라서 XML 설정에서 apect를 선언하기 앞서 `SimplePerformanceMonitor` Aspect 클래스를 bean으로 등록해줘야 한다.

```xml
<bean id="simplePerformanceMonitor" class="com.learning.aspect.SimplePerformanceMonitor" />
```

Aspect 클래스를 bean으로 등록했다면 해당 bean이 Aspect임을 선언해야 한다.

Aspect는 `<aop:aspect>` 태그를 사용하여 선언하고, Spring에 등록된 Aspect bean을 `ref` 속성을 사용하여 참조하면 된다. `ref`가 참조할 수 있는 bean은 부가 기능을 지원하는 bean으로써 기존에 사용된 Spring bean처럼 구성되고 DI 할 수 있다.

``` xml
<aop:config>
  <!-- Aspect 선언 및 부가 기능 bean 참조 -->
  <aop:aspect  id="simpleMonitoring"  ref="simplePerformanceMonitor" >
      ...
  </aop:aspect>
</aop:config>

<bean id="simplePerformanceMonitor" class="com.learning.aspect.SimplePerformanceMonitor" />
```

다음과 같이 정의된 `simplePerformanceMonitor` bean은  `<aop:aspect>` 태그의 `ref` 속성을 사용하여 참조하면 된다.

XML 설정을 통해 Aspect가 선언되었다 해서 끝이 아니다. 가장 중요한 pointcut과 adivce를 정의해줘야 한다. pointcut와 advice에 대한 설명은 추후 해당 주제로 포스팅할 예정이라 별도의 자세한 설명은 생략하겠다.

간단하게 설명하자면 Spring에서 제공하는 pointcut은 AspectJ pointcut 표현식 언어를 사용하고있다. XML 설정은 `<aop:config>` 태그 내부에 설정할 수 있고 또한 `<aop:aspect>` 태그 내부에도 설정이 가능하다.

``` xml
<aop:config>
  <aop:pointcut id="businessServiceA" expression="..." /> <!-- pointcut 정의 -->

  <aop:aspect>
    <aop:pointcut id="businessServiceB" expression="..." /> <!-- pointcut 정의 -->
  </aop:aspect>
</aop:config>
```

adive는 `<aop:aspect>` 태그 내부에 선언해야 하고, `aop` 네임스페이스가 제공하는 advice를 선택하고 설정해준다. 일반적으로 다음과 같은 속성을 사용하여 adivce를 설정해야 한다.

- `pointcut`
- `pointcut-ref`
- `method`

 속성 중에서 가장 독특한 점은 `method`라는 속성을 통해 Aspect의 메소드를 지정하여 해당 adivce를 활성화할 수 있다는 점이다.

``` xml
<!-- method 속성은 <aop:aspect> 태그에서 지정한 aspect의 메소드 -->
<aop:around pointcut-ref="businessService" method="monitoring" />
```

adivce는 서비스 프로세스의 시간 측정을 위해 Around를 선언하였고, method 속성을 통해 `SimplePerformanceMonitor.monitoring()` 메소드를 정의하여 advice에서 사용하겠다고 선언해주면 된다.

``` xml
<!-- Spring AOP -->
<aop:config>
  <aop:aspect id="simpleMonitoring" ref="simplePerformanceMonitor">
    <!-- pointcut 정의 : Advice가 적용될 JoinPoint를 정한다.  -->
    <aop:pointcut id="businessService" expression="execution(* com.learning.business.*.*(..))" />

    <!--
    	Around의 pointcut-ref 속성을 사용하여 미리 정의한 "businessService"을 참조한다.
    	aspect가 참조하는 bean의 부가기능 메소드를 method 속성을 통해 정의한다.
    -->
    <aop:around pointcut-ref="businessService" method="monitoring" />
  </aop:aspect>
</aop:config>

<!-- 서비스 메서드의 프로세스 시간을 측정하는 데 사용하려는 monitoring 클래스 -->
<bean id="simplePerformanceMonitor" class="com.learning.aspect.SimplePerformanceMonitor" />
```

여기까지 XML 스키마를 활용하여 Spring AOP를 설정해보았다. 이제 테스트 코드를 만들어서 AOP가 잘 적용됐는지 확인해보자.


#### 테스트 코드를 통해 XML 설정 확인해보자

다음 학습 테스트 코드의 목적은 크게 두 가지로 볼 수 있다.

1. Business bean이 JDK Dynamic Proxy Object로 생성되는지
2. Aspect가 적용됐는지

``` java
public class BusinessTest {

    private ClassPathXmlApplicationContext ctx;
    private Business business;

    @Before
    public void init(){
        ctx = new ClassPathXmlApplicationContext("classpath:/spring-aop-test.xml");
        business = ctx.getBean(Business.class);

        //JDK Dynamic Proxy 생성 여부
        System.out.println(business.getClass());
    }
}
```

먼저 `ClassPathXmlApplicationContext.class`를 통해 설정한 애플리케이션 컨텍스트를 읽고 `getBean()` 메소드를 통해 생성된 Business bean이 Proxy로 생성되었는지 확인해보자.

``` console
class com.sun.proxy.$Proxy9
```

JUnit을 통해 테스트 코드를 실행하면 다음과 같이 Business bean이 Proxy Object로 생성되었다는 결과를 얻을 수 있다.

이제 Spring AOP 설정이 제대로 적용됐는지 테스트 해보자.

``` java
@Test
public void monitoringTest() throws Exception {
    for(int i=0; i<3; i++){
        business.doAction();
    }

    business.doRuntimeException();
}
```

``` console
execution(Business.doAction()) : 501 ms
execution(Business.doAction()) : 501 ms
execution(Business.doAction()) : 500 ms
execution(Business.doRuntimeException()) : 0 ms , ERROR > 에러가 발생하였습니다.
```

다음 결과를 통해 XML 설정으로 기존 Business 클래스를 별도의 수정 작업 없이 Aspect가 적용되었다는 걸 확인할 수 있었다.

마지막으로 @AspectJ 방식에 대해 알아보자.

### @AspectJ

@AspectJ 방식은 AspectJ 5 라이브러리의 어노테이션들을 사용하여 Aspect를 Java Object로 구현하는 방식을 의미한다. 이 Aspect는 Spring에 의해 자동으로 감지되며 Spring AOP에 의해 실행된다. @AspectJ 방식은 AspectJ의 Compiler/Weaver과는 무관하며 Spring에서 AspectJ의 방식을 사용하고 싶다면 다음 [링크](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-using-aspectj)를 참고하자.

Spring에서 @AspectJ 방식을 사용하기 위해선 Spring AOP의 자동 프록싱(`autoproxying`) 설정이 필요하다.

> `autoproxying`는 Spring은 bean이 하나 이상의 Aspect에 의해 advice을 받았다고 판단하면 자동으로 메소드 호출을 가로채고 advice가 실행이 보장되도록  해당 bean이 Proxy Object가 자동으로 생성되는 개념을 뜻한다.

#### @AspectJ 어노테이션 활성화

Spring AOP에선 두 가지 방법을 통해 자동 프록싱 설정을 할 수 있다.

1. `@Configuration`, `@EnableAspectJAutoProxy`
2. `<aop:aspectj-autoproxy />`

>@EnableAspectJAutoProxy 어노테이션에 대한 자세한 정보는 이 [링크](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/EnableAspectJAutoProxy.html)를 참고하자.

두 번째 방식은 XML스키마 기반의 설정임으로 생략하자.

``` java
@Configuration
@EnableAspectJAutoProxy
@Import(BusinessImple.class)  // business bean import
public class AspectJAutoProxyConfig {

    @Bean
    public SimplePerformanceMonitor aspect() {
        return new SimplePerformanceMonitor();
    }
}
```

@AspectJ 지원이 활성화되면 Spring에 의해 애플리케이션 컨텍스트 또는 클래스를 통해 정의된 모든 bean 중에 @AspectJ 어노테이션 부착 여부를 자동으로 감지하고, 이를 Spring AOP를 설정하는 데 사용된다.

#### @AspectJ로 Aspect 구현

이제 @Aspect를 어노테이션을 사용하여 Aspect를 구현해보자.

``` java
@Aspect
@Component
public class SimplePerformanceMonitor {

    // 포인트 컷
    @Pointcut ("execution(* com.learning.aop.business.*.*(..))")
    private void businessService () {}

    @Around(value="businessService()")
    public Object monitoring(ProceedingJoinPoint joinPoint) throws Throwable{
        ...
    }
}
```

제일 먼저 @AspectJ를 활용한다면 해당 @Pointcut, @Around 등 어노테이션을 사용해야한다.


하지만 XML스키마 방식에 대한 몇 가지 단점들을 살펴볼 수 있다.

#### XML스키마 방식의 단점

1. Encapsulate
2. 표현의 제한

첫 번째 단점은 `Encapsulate`이다.

``` xml
<!-- Aspect bean XML -->
<bean id="aspectBean" class="..." />

<!-- AOP 설정 XML-->
<aop:config>
  <aop:aspect id="aspectA" ref="aspectBean">
      ...
  </aop:aspect>
</aop:config>
```

다음과 같이 XML스키마 방식은 Aspect 클래스를 bean으로 정의하고 `<aop:aspect>` 태그를 사용해서 빈을 참조하는 방식으로 구현한다.

- Aspect bean + AOP 설정

이는 DRY 원리에 위배 되는 사항이다. XML 방식을 사용할 때 aspect 기반의 bean 클래스의 선언과 설정파일의 XML에 나누어져 하나의 설정으로 관리하지 못한다는 단점이 생긴다.

> DRY 원리 : 특정 정보와 기능이 하나의 원천으로 존재한다는 것을 강조하는 개발 원리로써, 단순히 중복 코드를 방지를 넘어 하나의 정보로 명확하고 신뢰할 수 있는 코드를 지향하여 최종적으로 Clean Code까지 달성할 수 있는 개발 원리이다.

 반면 @AspectJ 방식을 사용하는 경우엔 하나의 모듈로 관리할 수 있다.

``` java
@Aspect // aspect 클래스 명시 <aop:aspect ... >
@Component // bean 등록 <bean ... >
public class AspectA{
}
```

두 번째 단점으론 XML스키마 방식은 AOP 표현에 있어 @AspectJ 방식보다 제약적이라는 점이다. XML스키마 방식에서는 싱글톤 관점 인스턴스화 모델만 지원하고 XML에서 선언한 이름이 붙은 pointcut을 결합할 수 없다.

``` java
@pointcut ( ... ) public void pointCutA(){}
@pointcut ( ... ) public void pointCutB(){}
@pointcut ( pointCutA() || pointCutB() ) public void pointCutAorB(){}
```

반면 @AspectJ에선 다음 코드처럼 정의된 기존 정의한 pointcut들을 조합한 pointcut 사용을 지원하고, 모듈 단위로 관점을 유지할 수 있다는 장점을 지니고 있다.

- `org.aspectj.lang.annotation.Aspect`

또한, @Aspect 어노테이션은 Spring AOP에서 자체적으로 제공하는 어노테이션이 아닌  AspectJ 기반의 어노테이션을 사용하고 있다는 점은 Spring AOP와 AspectJ가 모두 @AspectJ 방식을 인식할 수 있고 Spring AOP에서 AspectJ로 쉽게 마이그레이션을 할 수 있다는 장점이 있다.

이러한 @AspectJ 방식의 장점들 때문에 Spring 팀에선 XML스키마 방식보다 @AspectJ 방식을 선호한다.




#### 마무리

프로젝트의 시간이 많이 지난 시점에서 추가 요구사항이 처리하는 데 있어 난감한 경우가 많다.

문제없이 잘 돌아가는 시스템에 기존 코드를 수정해야 하는 상황이 발생한다면 개발자로선 부담된다. 특히 추가 기능이 다수의 기존 기능에 추가되어야 하는 상황이라면 개발자에겐 최악의 시나리오다.

물론 이러한 상황을 배제하더라도 기존 코드를 분석하는 시간과 테스트 시간 및 요구사항을 구현하는 데 있어 예기치 못한 오류가 발생하여 상당한 시간이 소요될 수 있다.

이러한 문제점들을 최소화하고 부가 기능을 추가하는 방법으로 관점(Aspect)으로 접근하는 방법이 가장 나은 접근일 수 있다.

하지만 Aspect와 관계없이 비즈니스 로직을 수정할 때도 무리하게 aspect를 접근하는 방식은 오히려 관리 포인트가 높아지기 때문에 상황을 고려하여 신중히 적용해야 한다고 생각한다.

### 참고

- [AOP 구현 세가지 방법 비교](https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ)

  - [Spring-DOC : 11. Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
