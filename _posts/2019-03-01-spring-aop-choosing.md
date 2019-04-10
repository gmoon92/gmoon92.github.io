---
layout: post
title: "Spring AOP 선언의 선택"
tags: [AOP, Spring, AspectJ, Srpring-AOP]
categories: [Spring, AOP]
subtitle: "Spring AOP로 시간 측정 Profiling 구현"
feature-img: "md/img/thumbnail/spring-aop-choosing.png"
thumbnail: "md/img/thumbnail/spring-aop-choosing.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# Spring AOP로 시간 측정 Profiling 구현

---

### 들어가기전

_**사용자 서비스에 실행 시간을 측정해서 알려주세요.**_

다음 요구사항이 처리하는 데 있어 Aspect가 가장 좋은 접근이라고 결정했다면, 다양한 구현방식 중에 어떻게 구현해야 할지 선택해야 한다. 본 포스팅에선 Spring AOP 선언 방식과 이 과정에서 간단하게 메소드 실행 시간을 측정하는 Profiling 목적을 띈 클래스를 구현하는 과정을 학습해보는 시간을 가지려 한다.

학습의 환경으론 Spring과 SpringBoot에서 진행했으며, XML 방법에선 Spring4 환경으로 @AspectJ 방법은 Spring Boot2을 사용했다. 본 포스팅에 사용했던 예제 코드는  [깃허브](https://github.com/gmun/spring-aop-simple-profiling-in-xml-config)를 참고하기 바란다.

### 학습목표

1. XML 방식의 이해
2. @AspectJ 어노테이션 방식의 이해
3. 두 Spring AOP 선언의 차이점에 대한 이해

### Spring AOP dependencies 설정

Spring AOP는 [aspectjweaver.jar](https://mvnrepository.com/artifact/org.aspectj/aspectjweaver)가 필요하다. 먼저 Maven의 빌드 설정 파일인 pom.xml 파일에 아래 코드를 추가하자.

``` xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.2</version>
</dependency>
```

### 비즈니스 로직 구현

필자는 JDK Dynamic Proxy를 사용하여 Aspect를 구현해보려 한다. JDK Dynamic Proxy와 CGLIB의 차이점에 대해선 다음에 자세히 다룰 예정이다.

#### 인터페이스 구현

본론으로 돌아와서, 아시다시피 JDK의 Proxy는 인터페이스만 지원하기 때문에, 먼저 비즈니스 인터페이스를 구현해보자.

``` java
public interface Business {
    public void doAction() throws Exception;
    public void doRuntimeException();
}
```

- doAction() → 비즈니스 로직 수행
- doRuntimeException() → 강제 예외 발생

첫 번째 메소드는 비즈니스 로직의 실행 시간을 측정하기 위함이고, 두 번째 메소드는 예외가 발생할 때 어떻게 처리가 될지 학습 테스트를 진행하기 위함이다.

#### Target Object 구현

BusinessImple 클래스는 advice(부가 기능) 적용될 Target Object이다.

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

다음 코드에서 doAction() 메소드는 Thread.sleep() 메소드를 사용하여 수행 시간을 지연시켰고 doRuntimeException() 메소드는 강제 예외를 발생시켰다.

여기서 AOP가 주는 가장 큰 이점을 발견할 수 있다. 이 점은 핵심 모듈(Aspect)과 부가 모듈을 분리하여 관리한다는 점이다. BusinessImple 클래스 내부에 메소드의 실행 시간을 측정과 관련된 어떠한 로직도 작성하지 않고 핵심 모듈로써 관리할 수 있다는 점을 살펴볼 수 있다.

#### 메소드 실행 측정 클래스 작성

이제 Aspect를 구현해보자.

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

- monitoring → advice 작성
- endStopWatch → 비즈니스 메소드 종료를 알리는 메소드

다음 `monitoring` 메소드의 매개 변수를 보면 ProceedingJoinPoint를 Spring AOP로부터 받는다는 걸 알 수 있다.

ProceedingJoinPoint를 사용함으로써 JoinPoint를 확장한 클래스로써 호출될 Target Object(BusinessImple.class)의 메소드 정보뿐만 아니라 `ProceedingJoinPoint.proceed()` 메소드를 통해 Target의 메소드의 호출 시점과 타깃의 매개 변수의 값을 제어할 수 있다.

따라서 Target Object 메소드의 실행 전후에 `org.springframework.util.StopWatch`의 메소드를 사용하여 실행 시간을 측정을 했다.

### Spring AOP 선택하기

Aspect까지 구현했다면 Spring AOP를 통해 `BusinessImple` 클래스에 Aspect를 접목시켜야한다.

일반적으로 JDK Dynamic Proxy를 구현하기 위해 `ProxyFactoryBean` 또는 `FactoryBean`을 통해 Aspect 구현할 수 있지만, Spring은 간편하게 AOP를 설정할 수 있도록 두 가지 방식을 제공하고 있다.

1. XML(스키마 기반)
2. @AspectJ(어노테이션 기반)

### XML(스키마) 방식

먼저 XML으로 Aspect를 정의하기 위해선 Spring이 제공하는 `aop`라는 Namespace 태그를 사용해야 한다.

 `aop` Namespace 태그 중에서 `<aop:config>`는 최상위 요소로써 XML 설정에 있어 가장 먼저 선언이 되어야 하고, AOP와 관련된 Aspect와 Advisor는 반드시 `<aop:config>` 요소 내부에 선언되어야 한다.

```xml
<aop:config>
    <aop:aspect>...</aop:aspect> <!-- aspect -->
    <aop:advisor>...</aop:advisor> <!-- advisor -->
</aop:config>
```

Spring AOP는 Spring IoC 컨테이너와 함께 사용된다. 이 말은 Spring에서 Aspect는 애플리케이션 컨텍스트에서 bean으로 정의된 일반 Java Object라고 해석할 수 있다. 따라서 `SimplePerformanceMonitor.class` Aspect 클래스를 bean으로 등록해줘야 한다.

```xml
<bean id="simplePerformanceMonitor" class="com.learning.aspect.SimplePerformanceMonitor" />
```

Aspect 클래스를 bean으로 등록했다면, Spring이 등록된 bean이 Aspect임을 인식할 수 있도록 Spring AOP 설정으로 Aspect임을 선언해줘야 한다.

Aspect 선언은 `<aop:aspect>` 태그를 사용하고, Spring에 등록된 `simplePerformanceMonitor` bean을 `<aop:aspect>`의 `ref` 속성을 사용하여 참조하면 된다. `ref`가 참조할 수 있는 bean은 부가 기능을 지원하는 bean으로써 기존에 사용된 Spring bean처럼 구성되고 DI 할 수 있다.

``` xml
<aop:config>
  <!-- Aspect 선언 및 부가 기능 bean 참조 -->
  <aop:aspect  id="simpleMonitoring"  ref="simplePerformanceMonitor" >
      ...
  </aop:aspect>
</aop:config>

<bean id="simplePerformanceMonitor" class="com.learning.aspect.SimplePerformanceMonitor" />
```

Aspect가 선언했다면 Target Object에 Aspect를 적용하기 위해 pointcut과 advice를 정의해줘야 한다. 먼저 pointcut에 대해 간단히 살펴보자.

``` xml
<aop:config>
  <aop:pointcut id="pointcutA" expression="..." /> <!-- <aop:config> 내부에 정의 -->

  <aop:aspect>
    <aop:pointcut id="pointcutB" expression="..." /> <!-- <aop:aspect> 내부에 정의 -->
  </aop:aspect>
</aop:config>
```

다음과 같이 pointcut은 `<aop:pointcut>` 태그를 사용하고 `<aop:config>` 태그 내부 또는 `<aop:aspect>` 태그 내부에 설정할 수 있다. Spring에서 제공하는 pointcut의 표현 식은 AspectJ 5의 pointcut 표현 식을 사용하고, `<aop:pointcut>` 태그의 `expression` 속성을 통해 구현할 수 있다.

``` xml
<aop:aspect id="simpleMonitoring" ref="simplePerformanceMonitor">
    <aop:pointcut id="businessService" expression="execution(* com.learning.business.*.*(..))" />
</aop:aspect>
```

마지막으로 adive는 `<aop:aspect>` 태그 내부에 `aop` 네임스페이스가 제공하는 advice 태그로 선언돼야 한다. 본 포스팅에선 서비스 프로세스의 시간 측정을 위해 `<aop:around>` 태그로 설정한다.

``` xml
<aop:around pointcut-ref="businessService" method="monitoring" />
```

> - `pointcut`
    - pointcut 표현 식을 직접 명시
> - `pointcut-ref`
    - `<aop:pointcut>` 태그 참조하여 pointcut 정의
> - `method`
    - `<aop:aspect>` 태그에서 지정한 aspect의 메소드 정의

#### 완성된 XML 설정

여기까지 XML을 활용하여 Spring AOP를 설정해보았다. 이제 테스트 코드를 만들어서 AOP가 잘 적용됐는지 확인해보자.

``` xml
<!-- Spring AOP config -->
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

#### XML 설정을 테스트 코드로 확인해보자

다음 학습 테스트 코드의 목적은 크게 세 가지로 볼 수 있다.

1. Business bean이 JDK Dynamic Proxy Object로 생성되는지
2. Aspect가 적용됐는지
3. Spring 통합 테스트도 통과되는지

``` java
public class BusinessTest {

    private ClassPathXmlApplicationContext ctx;
    private Business business;

    @Before
    public void init(){
        ctx = new ClassPathXmlApplicationContext("classpath:/spring-aop-test.xml");
        business = ctx.getBean(Business.class);
    }

    @Test
    public void isJDKDynamicProxy() throws Exception {
        //JDK Dynamic Proxy 생성 여부
        System.out.println(business.getClass());
        Assert.assertTrue(java.lang.reflect.Proxy.isProxyClass(business.getClass()));
    }
}
```

먼저 `ClassPathXmlApplicationContext.class`를 통해 설정한 애플리케이션 컨텍스트를 읽고 `getBean()` 메소드를 통해 생성된 Business bean이 Proxy로 생성되었는지 확인해보자.

``` html
class com.sun.proxy.$Proxy9
```

~~1. Business bean이 JDK Dynamic Proxy Object로 생성되는지~~

JUnit을 통해 테스트 코드를 실행하면 다음과 같이 Business bean이 Proxy Object로 생성되었다는 결과를 얻을 수 있다.

이제 Spring AOP 설정이 제대로 적용됐는지 테스트 해보자.

``` java
@Test
public void isJDKDynamicProxy() throws Exception {
    //JDK Dynamic Proxy 생성 여부
    System.out.println(business.getClass());
    Assert.assertTrue(java.lang.reflect.Proxy.isProxyClass(business.getClass()));

    for(int i=0; i<3; i++){
        business.doAction();
    }

    business.doRuntimeException();
}
```

``` html
execution(Business.doAction()) : 501 ms
execution(Business.doAction()) : 501 ms
execution(Business.doAction()) : 500 ms
execution(Business.doRuntimeException()) : 0 ms , ERROR > 에러가 발생하였습니다.
```

~~2. Aspect가 적용됐는지~~ <br/>
~~3. Spring 통합 테스트도 통과되는지~~

다음 결과를 통해 XML 방식으로 기존 Business 클래스를 별도의 수정 작업 없이 Aspect가 적용되었다는 걸 확인할 수 있었다. 하지만 XML 방식으로 Spring AOP를 설정할 때 다음과 같은 단점들을 생각해 볼 수 있다.

#### XML 스키마 방식의 단점

1. Encapsulate
2. 표현의 제한

첫 번째 단점은 `Encapsulate`이다.

``` xml
<!-- Aspect bean -->
<bean id="simplePerformanceMonitor" class="..." />

<!-- AOP 설정 -->
<aop:config>
  <aop:aspect id="simpleMonitoring" ref="simplePerformanceMonitor">
      ...
  </aop:aspect>
</aop:config>
```

다음과 같이 XML 방식은 Aspect 클래스를 bean으로 정의하고 `<aop:aspect>` 태그를 사용해서 빈을 참조하는 방식으로 구현한다.

- Aspect bean + Spring AOP 설정 XML

이는 DRY 원리에 위배 되는 사항이다. XML 방식을 사용할 때 aspect 기반의 bean 클래스의 선언과 AOP 설정 코드가 나누어져 하나의 설정으로 관리하지 못한다는 단점이 생긴다.

> DRY 원리 : 특정 정보와 기능이 하나의 원천으로 존재한다는 것을 강조하는 개발 원리로써, 단순히 중복 코드를 방지를 넘어 하나의 정보로 명확하고 신뢰할 수 있는 코드를 지향하여 최종적으로 `Clean Code`까지 달성할 수 있는 개발 원리이다.

 반면 @AspectJ 방식을 사용하는 경우엔 하나의 모듈로 관리할 수 있다.

``` java
@Aspect // aspect 클래스 명시 <aop:aspect ... >
@Component // bean 등록 <bean ... >
public class AspectA{
}
```

두 번째 단점으론 XML 방식은 AOP 표현에 있어 @AspectJ 방식보다 제약적이라는 점이다. XML 방식에서는 싱글톤 관점 인스턴스화 모델만 지원하고 XML에서 선언한 이름이 붙은 pointcut을 결합할 수 없다.

``` java
@pointcut ( ... ) public void pointCutA(){}
@pointcut ( ... ) public void pointCutB(){}
@pointcut ( pointCutA() || pointCutB() ) public void pointCutAorB(){}
```

반면 다음 코드처럼 @AspectJ에선 정의된 기존 정의한 pointcut들을 조합한 pointcut 사용을 지원하고, 모듈 단위로 Aspect를 관리할 수 있어진다.

또한, @Aspect 어노테이션은 Spring AOP에서 자체적으로 제공하는 어노테이션이 아닌  AspectJ 기반의 어노테이션을 사용하고 있다는 점은 Spring AOP와 AspectJ가 모두 @AspectJ 방식을 인식할 수 있고 Spring AOP에서 AspectJ로 쉽게 마이그레이션을 할 수 있다.

> On balance the Spring team prefer the @AspectJ style whenever you have aspects that do more than simple "configuration" of enterprise services. - Spring DOC

이러한 장점들 때문에 하나 이상의 Aspect를 설정하는 경우 Spring 팀에선 XML 방식보다 @AspectJ 방식을 선호한다고 한다.

### @AspectJ

@AspectJ 방식에 대해 자세히 알아보자.

@AspectJ 방식은 AspectJ 5 라이브러리의 어노테이션들을 사용하여 Aspect를 Java Object로 구현하는 방식을 의미한다. 이 Aspect는 Spring에 의해 자동으로 감지되며 Spring AOP에 의해 실행된다. @AspectJ 방식은 AspectJ의 Compiler/Weaver과는 무관하며 Spring에서 AspectJ의 방식을 사용하고 싶다면 다음 [링크](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-using-aspectj)를 참고하자.

먼저 Spring에서 @AspectJ 방식을 사용하기 위해선 Spring AOP의 자동 프록싱(`autoproxying`) 설정이 필요하다.

> `autoproxying`는 Spring은 bean이 하나 이상의 Aspect에 의해 advice을 받았다고 판단하면 자동으로 메소드 호출을 가로채고 advice가 실행이 보장되도록  해당 bean이 Proxy Object가 자동으로 생성되는 개념을 뜻한다. `autoproxying`에 대한 상세한 정보는 이 [링크](https://docs.spring.io/spring/docs/3.0.0.M3/reference/html/ch09s09.html)를 참고하자.

#### @AspectJ 어노테이션 활성화

Spring에 `autoproxying` 설정이 활성화되면 의해 Spring 애플리케이션 컨텍스트 또는 클래스를 통해 정의된 모든 bean 중에 @AspectJ 어노테이션 부착 여부를 자동으로 감지하고, Spring은 감지된 클래스를 사용하여 Spring AOP를 설정하는 데 사용된다.

Spring AOP에선 두 가지 방법을 통해 `autoproxying` 설정을 할 수 있다.

1. `<aop:aspectj-autoproxy />`
2. `@Configuration`, `@EnableAspectJAutoProxy`

첫 번째는 XML에서 `<aop:aspectj-autoproxy />` 태그를 작성하면 간편히 `autoproxying` 환경이 설정된다. 반면 순수 Java 코드로 `autoproxying` 환경을 설정할 수 있다.

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

다음 코드를 보면 `@Configuration`, `@EnableAspectJAutoProxy` 어노테이션을 사용하여 `autoproxying`을 설정할 수 있다. [@EnableAspectJAutoProxy](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/EnableAspectJAutoProxy.html)의 `proxyTargetClass` 속성은 CGLIB(하위 클래스 기반)으로 Proxy를 생성할지를 설정할 수 있지만, 기본값이 false임으로 별도의 설정이 아니라면 JDK Dynamic Proxy 기반으로 Proxy가 생성된다. 이제 @AspectJ 방식으로 Aspect 클래스를 구현해보자.

#### Spring Boot에서 @AspectJ로 Aspect 구현

@AspectJ 방식으로 Aspect를 구현하기 위해선 클래스에 @Aspect 어노테이션을 선언하여 해당 클래스가 Aspect 클래스라는 걸 선언해줘야 한다.

사실상 Spring Boot에선 자체적으로 `autoproxying` 설정이 내장되어 있으므로 `autoproxying`  설정 클래스(AspectJAutoProxyConfig.class)없이도  @Aspect와 함께 @Component를 설정하여 Aspect를 자체적으로 bean으로 등록하여 사용할 수도 있다.

``` java
@Aspect @Component public class SimplePerformanceMonitor { }
```

하지만 AspectJAutoProxyConfig.aspect()에서 Aspect bean을 등록해주었기 때문에 예제 코드에서는 명시적으로 @Component 어노테이션을 부착하진 않았다.

``` java
@Aspect public class SimplePerformanceMonitor { }
```

그다음 @Aspect 클래스 내부에 pointcut과 advice를 구현해줘야 한다. Pointcut은 @Pointcut 어노테이션을 사용하여 설정할 수 있다.

``` java
@Aspect
public class SimplePerformanceMonitor {

    @Pointcut ("execution(* com.learning.aop.business.*.*(..))")
    private void businessService () {}

}
```

@Pointcut은 메소드 수준에서 부착해야 하며, 사용되는 표현 식은 AspectJ5의 pointcut 표현 식과 같다. 표현 식의 상세한 정보는 [AspectJ5 DOC - PointCut](https://www.eclipse.org/aspectj/doc/released/progguide/semantics-pointcuts.html)를 참고하자.

그 다음 Around는 @Around 어노테이션을 사용하여 설정한다.

``` java
@Aspect
public class SimplePerformanceMonitor {

    @Pointcut ("execution(* com.learning.aop.business.*.*(..))")
    private void businessService () {}

    @Around("businessService()")
    public Object monitoring(ProceedingJoinPoint joinPoint) throws Throwable{
          ...
    }
    ...
}
```

XML 방식과 마찬가지로 advice에 관련된 AspectJ 어노테이션들은 정의된 pointcut 메소드를 참조할 수 있다.

#### @AspectJ 설정을 테스트 코드로 확인해보자

이제 @AspectJ 방식의 설정이 제대로 동작하는지  테스트 코드를 통해 확인해보자.

1. Business bean이 JDK Dynamic Proxy Object로 생성되는지
2. Aspect가 적용됐는지
3. Spring 통합 테스트도 통과되는지

``` java
public class BusinessTest {
    private AnnotationConfigApplicationContext ctx;
    private Business business;

    @Before
    public void init(){
        ctx = new AnnotationConfigApplicationContext();
    }

    @Test
    public void isJDKDynamicProxy(){
        ctx.register(AspectJAutoProxyConfig.class);
        ctx.refresh();
        business = ctx.getBean(Business.class);

        //JDK Dynamic Proxy 생성 여부
        System.out.println(business.getClass());
        Assert.assertTrue(java.lang.reflect.Proxy.isProxyClass(business.getClass()));
    }

    @After
    public void destory() throws Exception{
        for(int i=0; i<3; i++){
            business.doAction();
        }
        business.doRuntimeException();

        ctx.close();
    }
}
```

``` html
class com.sun.proxy.$Proxy22
execution(Business.doAction()) : 501 ms
execution(Business.doAction()) : 500 ms
execution(Business.doAction()) : 500 ms
execution(Business.doRuntimeException()) : 0 ms , ERROR > 에러가 발생하였습니다.
```

~~1. Business bean이 JDK Dynamic Proxy Object로 생성되는지~~<br/>
~~2. Aspect가 적용됐는지~~

테스트가 잘 진행되었다. 마지막으로 Spring Boot 환경과 함께 테스트 코드를 실행하여 통합 테스트 상황에서도 JDK Dynamic Proxy가 생성되는지 확인해보자.

``` java
@Test
public void isJDKDynamicProxyWithSpringRunner() throws Exception{
    ctx = (AnnotationConfigApplicationContext) SpringApplication.run(MyApplication.class, new String[]{""});
    business = ctx.getBean(Business.class);

    //JDK Dynamic Proxy 생성 여부
    System.out.println(business.getClass());
    Assert.assertTrue(java.lang.reflect.Proxy.isProxyClass(business.getClass()));
}
```

``` html
class com.learning.aop.business.BusinessImple$$EnhancerBySpringCGLIB$$e5825056
```

분명 AspectJAutoProxyConfig 클래스에서  `@EnableAspectJAutoProxy`
JDK Dynamic Proxy로  설정했는데 Spring에 의해 Proxy가 자동적으로 CGLIB Proxy로 생성되었다.

무슨일일까... 이와 같은 현상은  [springboot-issues#8434](https://github.com/spring-projects/spring-boot/issues/8434)에서 답을 찾을 수 있다.

>We've generally found cglib proxies less likely to cause unexpected cast exceptions. - Phil Webb(Spring Framework committer and current lead of Spring Boot.)

현재 Spring Boot의 프로젝트 리더인 Phil Web은 CGLIB Proxy 방식이 예기치 않은 캐스팅 예외를 일으킬 가능성이 적다고 한다.

이러한 이슈에 대해 Spring 4.3, Spring Boot 1.4 이상의 버전부터 proxyTargetClass 옵션이 `proxyTargetClass=true`로 기본적으로 셋팅되도록 Release 되었다. 즉 bean을 구성하기 위해 인터페이스로 구현하지 않아도 되고 인터페이스 유무와 상관없이 CGLIB Proxy가 생성됨을 의미한다.

Spring Boot 환경에서 CGLIB를 강제하지 않고 JDK Dynamic Proxy로 설정하는 해결 방법으론 `application.properties` 파일에 `spring.aop.proxyTargetClass=false`를 추가하는 방법을 제시하고 있다. 테스트 코드로 확인해보자.

``` html
class com.sun.proxy.$Proxy45
```

~~3. Spring 통합 테스트도 통과되는지~~

### 추가 공부거리...

- @Pointcut의 다양한 표현 식
- JDK Dynamic Proxy와 CGLIB Proxy 비교
- Proxy 방식 AOP의 self-invocation 이슈

### 마무리

프로젝트의 시간이 많이 지난 시점에서 추가 요구사항이 처리하는 데 있어 난감한 경우가 많다.

문제없이 잘 돌아가는 시스템에 기존 코드를 수정해야 하는 상황이 발생한다면 개발자로선 부담된다. 특히 추가 기능이 다수의 기존 기능에 추가되어야 하는 상황이라면 개발자에겐 최악의 시나리오다. 물론 이러한 상황을 배제하더라도 기존 코드를 분석하는 시간과 테스트 시간 및 요구사항을 구현하는 데 있어 예기치 못한 오류가 발생하여 상당한 시간이 소요될 수 있다.

이러한 이슈들을 최소화하고 부가 기능을 추가하는 방법으로 관점(Aspect)으로 본 포스팅처럼 AOP로 구현하는 방법이 가장 나은 접근일 수 있다.

하지만 Aspect와 관계없이 비즈니스 로직을 수정할 때도 무리하게 AOP로 접근하는 방식은 오히려 관리 포인트가 높아지기 때문에 상황을 고려하여 신중히 적용해야 한다고 생각한다.

### 참고

- [Spring 4.3.15 RELEASE DOC - Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
- [Spring 4.3.15 RELEASE DOC - AOP choosing](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-choosing)
- [Spring 4.3.15 RELEASE DOC - AOP Schema](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-schema)
- [Spring 4.3.15 RELEASE DOC - AOP AspectJ](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)
- [Spring 4.3.15 RELEASE DOC - AOP using AspectJ](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html#aop-using-aspectj)
- [Spring-Boot-1.4-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-1.4-Release-Notes)
