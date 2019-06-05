---
layout: post
title: "Spring AOP의 메커니즘과 Proxy Bean 생성"
tags: [AOP, Spring, SpringAOP, XML, AspectJ]
categories: [Spring, AOP]
subtitle: "JDK Dynamic Proxy와 FactoryBean 그리고 ProxyFactoryBean"
feature-img: "md/img/thumbnail/aop-proxy.png"
thumbnail: "md/img/thumbnail/aop-proxy.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# JDK Dynamic Proxy와 FactoryBean 그리고 ProxyFactoryBean

---

### 들어가기전

Spring AOP는 프록시를 기반으로 하고 있다. 그렇다면 프록시를 어떻게 구축할까?

본 포스팅에선 JDK 다이내믹 프록시와 ProxyFactoryBean를 통해 AOP 프록시를 구현해보며, Spring AOP의 기본적인 동작 원리와 구현방법에 대해 학습해보려 한다. 학습 환경으론 SpringBoot에서 진행했고 학습 과정에서 사용했던 코드는 [GitHub](https://github.com/gmun/spring-aop-proxy)를 참고하기 바란다.

### 학습목표

- JDK Dynamic Proxy의 이해
- ProxyFactoryBean의 이해

### 프록시 기반 디자인 패턴의 한계

이전 포스팅 "[OOP에서 AOP](https://gmun.github.io/spring/aop/oop/2019/02/09/from-oop-to-aop.html)"에서 디자인 패턴들로 관심사의 분리를 해보았지만 크게 두 가지의 문제로 관심사를 분리하기에 어려움이 있었다.

- 객체의 관계에서 구체적인 구조를 파악되야 한다.
- 어찌됐든 기존 구조를 변형시켜 해결한다.

우선 Weaving을 하기 위해선 적용할 타깃의 전체적인 구조가 파악해야 된다는 전제가 깔려 있다. 또한, 전체적인 구조가 파악됐더라도 기존의 구조를 변형시킨다는 점은 추후 개발에 있어 고려해야될 사항들이 많았다.

반면 AspectJ은 1) 포인트컷으로 타깃의 호출 시점을 정의하고, 2) 어드바이스를 통해 부가기능을 구현했다. 어드바이스가 주입될 시점을 포인트컷으로 명시되어있기 때문에 타깃의 구체적인 구조를 파악할 필요 없이 타깃의 바이트 코드를 조작하여 타깃에 어드바이스를 적용시킬 수 있다. 이 점은 프록시 기반의 디자인 패턴의 매커니즘과 매우 유사하다.

- 데코레이터 클래스에 다음 대상 위임 코드와 어드바이스를 구현한다.
- 지정된 요청 시에 부가기능을 수행한다.

하지만 프록시 기반의 디자인 패턴은 AspectJ와 달리 부가기능을 구현하기 위해, 프록시를 구성하는 과정들이 번거롭다는 문제가 발생했다.

### 1. JDK Dynamic Proxy

이 문제는 JDK 다이내믹 프록시를 사용하면 간단히 해결할 수 있다.

JDK 다이내믹 프록시란 `java.lang.reflect.Proxy` 클래스를 사용함을 의미한다. 이 클래스는 리플렉션 기능을 적극적으로 활용하여 동적으로 프록시를 구성해준다. 이점은 부가기능을 주입하기 위해 바이트 코드를 조작하는 AspectJ의 접근 방법과 유사하다 할 수 있다.

>리플렉션이란 클래스 자체의 원시적인 코드에 접근할 수 있도록 지원하는 Java API이다. 코드 자체를 추상화해서 접근하기 때문에 구체적인 클래스의 종속 관계나 타입을 알지 못해도 메소드를 호출하거나 심지어 메타정보를 활용하여 객체를 조작할 수 있다.

프록시 객체는 다음과 같이 `Proxy.newProxyInstance(...)` 메소드를 통해 생성할 수 있다.

``` java
Object proxy = Proxy.newProxyInstance(loader      // ClassLoader
                                     ,interfaces  // Class<?>[]
                                     ,handler     // InvocationHandler
                                  );
```

- ClassLoader loader
- Class<?>[] interface
- InvocationHandler handler

newProxyInstance() 메소드에 각각의 매개변수는 프록시 생성에 있어 반드시 제공해줘야 할 매개변수들이다. 무엇보다 이 매개변수들의 사용 목적을 알기 위해선 프록시가 생성되는 과정을 요약한 순서도를 보면 도움이 된다.

#### 1.1. Proxy의 생성 과정과 매개변수들

다음 그림은 `Proxy.newProxyInstance()` 메소드의 내부적인 프록시가 생성되어지는 프로세스를 요약한 순서도이다.

<img src="/md/img/aop/spring-aop/jdk-dynamic-proxy1.png" style="max-height:none; padding:0px;">

- ClassLoader : 동적으로 프록시 생성을 하기 위해 사용될 클래스 로더
- Interface : 프록시 객체로 생성할 타깃의 인터페이스
- InvocationHandler : 부가기능과 위임 대상 코드가 포함된 InvocationHandler 인터페이스의 구현체

먼저 `java.lang.ClassLoader`는 프록시를 생성할 수 있는지에 대한 검증을 위한 목적을 띄고 있다. JDK 다이내믹 프록시는 이 클래스 로더를 활용하여 부가기능이 부착할 인터페이스의 구조의 검증과 궁극적으로 런타임 시 프록시 객체를 생성하기 위함이다. 이 점은 AspectJ와 가장 큰 차이를 알 수 있다.

두 번째 매개변수인 interface는 클래스 로더를 통해 런타임 시 동적으로 새로운 클래스를 확장하기 위함이다. 클래스 로더를 통해 JVM에 새로운 클래스 자체를 동적으로 적재하기 위해선 기본적으로 추상화 클래스나 인터페이스가 필요하기 때문이다.

``` java
public static Object newProxyInstance(...){
    ...
    // 클래스 로더를 사용하여 복사된
    // 타깃의 인터페이스를 통해 프록시 객체로 생성할 하위 클래스를 찾고
    // 프록시 객체를 생성한다.
    Class<?> cl = getProxyClass0(loader ← java.lang.ClassLoader
                                ,intfs  ← 복사된 타깃의 인터페이스
                                );
    ...
}
```

이 과정들은 결과적으로 `getProxyClass0()` 메소드를 통해 이뤄진다. 이 메소드가 호출이 되면 프록시 팩토리에게 클래스 로더와 인터페이스 정보를 제공하여 해당 프록시 객체를 자동으로 생성해준다.

> Constructors in java.lang.ClassLoader (and its subclasses) allow you to specify a parent when you instantiate a new class loader. - [Oracle ClassLoader](https://www.oracle.com/technetwork/articles/javase/classloaders-140370.html)

마지막으로 InvocationHandler h 매개변수는 프록시를 생성하는 과정에 있어, 맨 처음부터 null 체크를 하는 만큼 중요한 매개변수라 할 수 있다. InvocationHandler는 부가 기능을 구현하는 인터페이스이다.

``` java
public static Object newProxyInstance(...){
    //InvocationHandler h null 체크, null인 경우 예외
    Objects.requireNonNull(h);
    ...
    final Constructor<?> cons = cl.getConstructor(constructorParams);
    ...
    // 검증된 프록시 객체의 생성자에
    // InvocationHandler 기능을 확장한 새로운 객체를 반환한다.
    return cons.newInstance(new Object[]{h});
}
```

다음 코드처럼 JDK 다이내믹 프록시는 제공된 인터페이스의 정보를 통해 인터페이스의 구현체를 프록시 객체로 생성해주고, 최종적으로 이 프록시 객체에 InvocationHandler 기능을 확장한 프록시 객체가 반환된다. 따라서 필요한 부가기능은 InvocationHandler를 직접 구현하여 제공해줘야 한다.

``` java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

다음과 같이 InvocationHandler는 invoke() 메소드만 구현하면 되는 간단한 인터페이스다. invoke() 메소드는 리플렉션의 `java.lang.reflect.Method` 인터페이스를 매개 변수로 받고 있는데 이 매개 변수에 대한 이해는 다음 JDK 다이내믹 프록시의 동작 방식을 보면 이해가 쉽다.

#### 1.2. JDK Dynamic Proxy의 동작 원리

<img src="/md/img/aop/spring-aop/jdk-dynamic-proxy2.png" style="max-height:none; padding:0px;">

우선 클라이언트의 모든 요청에 대해 프록시 객체는 1) 호출된 Method 정보를 리플렉션 정보로 변환하여 2) InvocationHandler.invoke() 메소드에 제공해주고 InvocationHandler는 Method 정보를 통해 3) 최종적으로 타깃의 메소드를 호출한다. 이러한 동작 방식에 대해 숙지하고 간단히 구현해보면 다음과 같은 구조가 형성된다.

<img src="/md/img/aop/spring-aop/jdk-dynamic-proxy3.png" style="max-height:none; padding:0px;">

``` html
MemberBusiness.doAction() ← 비즈니스 로직
doAction result : 1000 ← 부가기능
```

다음 그림과 테스트 결과의 핵심은 부가기능인 MonitorHandler와 Business 클래스들의 독립적인 관계에서도 부가기능을 확장할 수 있다는 점이다.

이러한 이유엔 JDK 다이내믹 프록시는 Proxy.newInstance()라는 스태틱 팩토리 메소드로만 프록시 객체를 생성할 수 있기 때문이다. 팩토리 메소드 디자인 패턴의 특성상 타깃의 구체적인 유형 및 종속성과 고려하지 않아도 되고, 결과적으로 InvocationHandler의 기능이 확장된 새로운 객체가 생성되기 때문에 부가기능을 독립적으로 구현하고 타깃에 쉽게 적용할 수 있었다.

#### 1.3. Proxy의 동적인 클래스 정보

하지만 JDK 다이내믹 프록시는 일반적인 방법으로 Spring의 Bean으로 등록할 수 없다.

Spring Bean은 기본적으로 클래스 이름과 프로퍼티로 정의되어 있고, Bean 정의에서 지정된 클래스 이름과 리플렉션 API의 `Class.forName("...").newInstance();`를 활용하여 내부적인 로직을 통해 Bean을 생성해준다. 하지만 JDK 다이내믹 프록시는 지정된 호출 시점에 동적으로 프록시 클래스를 객체해주기 때문에 사전에 프록시 객체의 정보를 미리 알 수 없다.

### 2. JDK Dynamic Proxy & FactoryBean

Spring은 정적인 클래스 정보를 가지고 Bean을 생성해주는 방식 이외에도 여러 Bean을 생성할 수 있는 방법을 제공하고 있다. 대표적으로 FactoryBean을 이용하여 Bean을 생성할 수 있다.

FactoryBean은 Bean 객체의 생성로직을 구현할 수 있는 인터페이스이다. 인터페이스의 구성은 다음과 같다.

``` java
public interface FactoryBean<T> {
    T getObject() throws Exception;  → Bean 객체를 생성하고 반환
    Class<?> getObjectType();  → FactoryBean에 의해 생성된 객체의 Type
    default boolean isSingleton() {return true;}   → getObject()의 반환된 객체의 싱글톤 여부
}
```

FactoryBean 인터페이스의 getObject() 메소드는 Bean 객체를 생성하는 목적을 가지고 있다. 따라서 FactoryBean의 구현 클래스를 구성하고 기존에 작성했던 Proxy.newInstance()의 로직을 getObject() 메소드에 작성해주면 FactoryBean에 의해 프록시 객체가 Bean으로 생성되어진다.

#### 2.1. FactoryBean 구현

``` java
@Setter
public class MonitorFactoryBean implements FactoryBean<Object>{
    private Class<?> interfaces;
    private Object   target;

    @Override
    public Object getObject() throws Exception {
        return Proxy.newProxyInstance(getClass().getClassLoader()
                                    , new Class[] {interfaces}
                                    , new MonitorHandler(target));
    }
    ...
}
```

다음 코드처럼 getObject() 메소드를 통해 프록시 객체를 반환해주면 되는데, 마지막으로 Spring이 생성된 동적인 프록시 Bean을 인식시켜주면 된다.

#### 2.2. Proxy Bean 설정

``` java
@Configurable
public class MyBeanConfig {
    @Bean
    public MonitorFactoryBean monitorFactoryBean () {
        MonitorFactoryBean factory = new MonitorFactoryBean();
        factory.setInterfaces(Business.class);
        factory.setTarget(new MemberBusiness());
        return factory;
    }
}
```

@Configurable 어노테이션은 클래스를 통해 Bean을 구현할 수 있도록 도와주는 어노테이션이다. 이 어노테이션을 활용하여 다음과 같이 FactoryBean을 Bean으로 등록을 시켜주면 getObject() 메소드에서 반환되는 동적인 프록시 Bean이 자동으로 Spring에서 Bean으로 인식된다.

<img src="/md/img/aop/spring-aop/jdk-dynamic-proxy4.png" style="max-height:none; padding:0px;">

FactoryBean을 통해 JDK 다이내믹 프록시를 적용에 대한 전체적인 플로우는 다음과 같다. 이제 테스트 코드를 통해 검증을 해보자.

#### 2.3. FactoryBean 테스트

테스트의 목적은 두 가지이다.

1. FactoryBean에 의해 프록시 Bean이 자동으로 생성되는지
2. FactoryBean의 설정만으로도 기존 Business 클래스에 Bean이 DI 되는지

첫 번째는 FactoryBean에 의해 동적인 프록시 Bean을 생성할 수 있는지 그리고 MyBeanConfig 클래스에서 FactoryBean를 Bean으로 등록시켜 주었는데, FactoryBean.getObject()에 생성 되어지는 프록시 Bean이 최종적으로 기존 Business 클래스의 수정없이 타깃에 DI 해주는지에 대한 테스트이다.

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes= {MyBeanConfig.class}) // 애플리케이션 컨텍스트의 설정 파일 지정
public class ProxyWithFactoryBeanTest {

    @Autowired
    private ApplicationContext context;

    @Test // FactoryBean을 통해 프록시 Bean의 생성여부
    public void isContextBeanConfigTest() throws Exception {
      /**
      * FactoryBean이 만들어주는 Bean 객체가 아닌 FactoryBean 자체를 가져오고 싶을 경우엔
      * getBean 메소드에 '&'를 Bean 이름 앞에 붙여주면 FactoryBean 자체를 반환한다.
      * */
      MonitorFactoryBean factory = (MonitorFactoryBean) context.getBean("&monitorFactoryBean");
      Business business = (Business) factory.getObject();
      business.doAction();
    }

    @Autowired
    private Business memberBusiness;

    @Test // FactoryBean에 의해 생성된 프록시 Business Bean이 자동으로 주입되는지
    public void isWeaving() throws Exception {
        memberBusiness.doAction();
    }
}
```
``` html
MemberBusiness.doAction() ← FactoryBean에 의해 프록시 Bean이 자동으로 생성되는지
doAction result : 1004

MemberBusiness.doAction() ← FactoryBean의 설정만으로도 기존 Business 클래스에 Bean이 DI 되는지
doAction result : 1001
```

~~1. FactoryBean에 의해 프록시 Bean이 자동으로 생성되는지~~<br/>
~~2. FactoryBean의 설정만으로도 기존 Business 클래스에 Bean이 DI 되는지~~

다음 테스트 결과를 통해 FactoryBean로 생성된 프록시 Bean을 생성하고 사용할 수 있을까에 대한 근본적인 의문을 해결할 수 있었다. 또한, 부가기능을 확장하기 위해 기존 비즈니스 클래스의 구조를 수정하는 작업도 없고 부가기능을 독립적으로 관리할 수 있고 재사용할 수 있다.

하지만 FactoryBean는 일반적으로 순수 프록시를 목적을 띈 인터페이스가 아니다 보니 기능 자체에도 한계가 존재한다.

#### 2.4. FactoryBean의 세 가지의 한계

우선 JDK 다이내믹 프록시에 대해 다시 생각해보자. 부가기능은 InvocationHandler를 통해 구현하였고, 메소드 단위에 대한 부가기능을 확장했다. 이러한 기본 동작 원리에 따라 다음 두 가지 시점에서 FactoryBean에 대한 고민을 해볼 수 있다.

1. 부가기능 기준으로 여러 타깃에 적용
2. 타깃의 기준으로 여러 부가기능 확장

<img src="/md/img/aop/spring-aop/jdk-dynamic-proxy5.png" style="max-height:none; padding:0px;">

다음 그림과 함께 두 가지 상황을 고려해보자.

먼저 같은 부가기능을 여러 타깃에 적용한다고 가정해보면 InvocationHandler에 위임할 타깃에 대한 정보만 바뀌고 나머지 부분은 중복된 코드가 발생한다. 마지막으로 타깃의 기준으로 여러 부가기능을 확장한다고 하면 프록시 Bean 설정 부분에 수정 작업이 동반된다. 결과적으로 부가기능을 확장하는데 OCP 원칙을 잘 준수되느냐에 대한 의문이 생긴다.

### 3. ProxyFactoryBean

Spring에선 순수 프록시 Bean만을 생성해주는 목적을 띈 ProxyFactoryBean을 제공하고 있다.

``` java
public class ProxyFactoryBean extends ProxyCreatorSupport
implements FactoryBean<Object>, BeanClassLoaderAware, BeanFactoryAware {
    ...
    public Object getObject() throws BeansException { ← 프록시 Bean 객체 반환
      ...
    }
    ...
}
```

ProxyFactoryBean은 FactoryBean 인터페이스를 구현한 클래스로 FactoryBean과 마찬가지로 getObject() 메소드를 통해 프록시 Bean 객체를 반환 받을 수 있다. 우선 ProxyFactoryBean을 통해 프록시를 구현해보자.

``` java
@Test
public void createProxyBean(){
    ProxyFactoryBean factory = new ProxyFactoryBean();
    factory.setTarget(new MemberBusiness()); ← 타깃
    factory.addAdvice(new MonitorHandler()); ← 부가기능

    Business member = (Business)factory.getObject(); ← 프록시 Bean을 반환받는다.
    System.out.println(factory.getObjectType());
    member.doAction();

    assertThat(Proxy.isProxyClass(factory.getObjectType()), is(true));
}
```
``` html
class com.sun.proxy.$Proxy5
MemberBusiness.doAction()
result : 1000
```

다음 코드에서 살펴볼 수 있듯이 타깃과 부가기능을 ProxyFactoryBean에 제공하면 알아서 프록시 Bean을 반환해주는 전형적인 템플릿/콜백 구조로 되어있다는 걸 알 수 있다. 또한, 자세히 보면 JDK 다이내믹 프록시와는 다른 구현 방식을 찾을 수 있다.

1. 타깃의 인터페이스 정보가 필요없다.
2. 부가기능은 addAdvice()로 구현한다.

첫 번째는 타깃의 인터페이스 정보를 제공하지 않았다는 점이다. 이러한 이유엔 ProxyFactoryBean이 자체적으로 타깃의 클래스를 통해 역으로 타깃의 인터페이스를 검출하여 JDK 다이내믹 프록시를 생성해주기 때문이다. 만약 인터페이스가 없는 타깃이라면 CGLIB이라고하는 오픈소스 바이트코드 생성 프레임워크를 이용해 프록시를 생성해준다.

> ProxyFactoryBean.newPrototypeInstance()의 setInterfaces()가 없으면 CGLIB 있으면 JDK Dynamic Proxy

다음은 부가기능의 구현 방식이다. 부가기능은 addAdvice() 메소드로 구현되는데, 메소드 명을 보면 타깃과 달리 add라는 점을 알 수 있다. 자세한 내용은 ProxyCreatorSupport 클래스의 부모 클래스인 AdvisedSupport 클래스를 보면 알 수 있다.

``` java
public class AdvisedSupport extends ProxyConfig implements Advised {
    ...

    private List<Advisor> advisors = new ArrayList<>(); ← 리스트로 구현된 부가기능

    public void addAdvice(Advice advice) throws AopConfigException {
        int pos = this.advisors.size();
        addAdvice(pos, advice);
    }
    ...

    private void addAdvisorInternal(int pos, Advisor advisor) throws AopConfigException {
        ...
        this.advisors.add(pos, advisor);
        ...
    }

}
```

AdvisedSupport 클래스의 내부 로직을 보면 부가기능은 리스트로 구현되어 있다는걸 알 수 있다. 즉 타깃에 대한 여러 부가기능을 확장하고 관리할 수 있으므로 FactoryBean의 한계인 `2. 타깃의 기준으로 여러 부가기능 확장` 에 대한 부분을 해소할 수 있다.

특히한 점은 addAdvice(Advice advice) 메소드는 Advice 인터페이스를 매개변수로 받고 있다는 점이다. JDK 다이내믹 프록시는 부가기능을 InvocationHandler를 통해 구현했지만 ProxyFactoryBean에선 부가기능을 MethodInterceptor 인터페이스로 구현해서 만든다.

<img src="/md/img/aop/spring-aop/proxy-factory-bean1.png" style="max-height:none; padding:0px;">

#### 3.1. MethodInterceptor의 순수 부가기능 구현

우선 MethodInterceptor 인터페이스 구성을 살펴보면, InvocationHandler 인터페이스와 마찬가지로 invoke() 메소드만 구성되어 있다.

``` java
@FunctionalInterface
public interface MethodInterceptor extends Interceptor {
  Object invoke(MethodInvocation invocation) throws Throwable;
}
```

> Intercepts calls on an interfac on its way to the target ... These are nested "on top" of the target. ... A method invocation is a joinpoint and can be intercepted by a method interceptor - Rod Johnson

MethodInterceptor 인터페이스는 타깃보다 먼저 수행하기 때문에 타깃의 메소드 호출을 가로챌 수 있다. 이 타깃의 메소드 정보는 ProxyFactoryBean에 의해 MethodInvocation 매개변수에 제공된다. 이처럼 타깃이 없는 순수한 부가기능을 Advice한다.

그 외에도 ProxyFactoryBean은 포인트컷을 객체로 관리할 수 있다. 구현은 Spring에서 Pointcut 인터페이스를 구현한 다양한 클래스들을 제공하고 있으므로 이를 사용하면 된다.

_어드바이져 = 포인트컷 + 어드바이스_

ProxyFactoryBean에 포인트컷을 등록하기 위해선 addAdvisor()를 사용하면 된다. 어드바이저란 Spring에서 사용되는 AOP 용어로 포인트컷과 어드바이스의 기능을 묶은 의미를 뜻한다.

#### 3.2. 동작 원리 비교

결과적으로 ProxyFactoryBean은 템플릿/콜백 구조이므로 각각의 목적을 띈 기능들을 독립적인 객체로 관리할 수 있다. 다음 동작 원리를 비교해보며 정리해보자.

<img src="/md/img/aop/spring-aop/proxy-factory-bean2.png" style="padding:0px;">

우선 FactoryBean은 InvocationHandler에서 어드바이스와 포인트컷을 관리했었다. 따라서 타깃의 기능이 수정하는 작업이 동반되면 InvocationHandler의 작업도 동반됐다. 하지만 ProxyFactoryBean은 목적을 띈 각 객체들은 ProxyFactoryBean에 의해 다음과 같은 순서로 타깃에게 부가기능을 전달해준다.

#### 3.3. IoC 컨테이너에 Proxy Bean 등록

이제 ProxyFactoryBean을 통해 생성된 프록시 Bean을 IoC 컨테이너에 등록해보자.

``` java
@Configurable
public class MyAdvisorConfig {
    @Bean
    public Pointcut businessPointcut() {
        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.addMethodName("do*");
        return pointcut;
    }

    @Bean
    public Advice monitorAdvice() {
        return new MonitorHandler();
    }

    @Bean
    public Advisor myAdvisor() {
        return new DefaultPointcutAdvisor(businessPointcut(), monitorAdvice());
    }
}
```

- businessPointcut ← 비즈니스 클래스의 메소드가 do로 시작되는 포인트컷
- monitorAdvice ← 메소드 실행시간 측정 부가기능
- myAdvisor ← 비즈니스 어드바이저 (businessPointcut + monitorAdvice)

다음으론 ProxyFactoryBean에 타깃 Bean과 어드바이저 Bean을 등록해주자.

``` java
@Configurable
@Import(MyAdvisorConfig.class)
public class MyBeanConfig {

    @Bean
    public ProxyFactoryBean factoryBean() {
        ProxyFactoryBean factory = new ProxyFactoryBean();
        factory.setTarget(new MemberBusiness());
        factory.setInterceptorNames( new String[]{"myAdvisor"} );
        return factory;
    }
}
```

- setTarget ← 타깃 클래스
- setInterceptorNames ← Bean으로 등록된 어드바이스, 어드바이저를 등록시킨다.

setInterceptorNames는 Bean으로 등록된 어드바이스와 어드바이저를 동시에 설정할 수 있는 프로퍼티이다. String 타입으로 구성되어 있기 때문에, 여러 어드바이스나 어드바이저의 Bean 아이디 값으로 넣어주면 된다.

#### 3.4. 테스트 검증

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes= {MyBeanConfig.class})
public class MyBeanConfigTest {

    @Autowired
    private Business memberBusiness;

    @Test
    public void isWeaving() {
        memberBusiness.doAction();
    }
}
```
``` html
MemberBusiness.doAction()
result : 1035
```

다음 테스트 결과를 통해 ProxyFactoryBean에 의해 생성된 프록시 Bean 객체가 비즈니스 클래스에 주입된걸 알 수 있다.

<img src="/md/img/aop/spring-aop/proxy-factory-bean3.png" style="padding:0px;">

- Pointcut Bean으로 생성 가능
- Advice Bean으로 생성 가능
- Pointcut, Advice 재사용 가능

다시 한 번 정리해보자면 ProxyFactoryBean은 템플릿/콜백 구조로 되어있고 OCP 원칙을 준수하며 어드바이스와 포인트컷을 독립적인 객체로 관리할 수 있었다. 이 말인즉슨 각 기능을 Bean으로 관리할 수 있다는 의미고 최종적으로 어드바이스와 포인트컷의 Bean을 재사용하기 때문에, 이들의 조합을 통해 새로운 어드바이져를 만들거나 기존의 어드바이저를 수정하기 수월해졌다.

### 추가 공부거리...

- 자동 프록싱

하지만 ProxyFactoryBean은 하나의 타깃에만 적용이 되는 설정의 한계를 여전히 가지고 있기 때문에 적용시킬 타깃이 늘어날 수록 ProxyFactoryBean이 늘어나고 자연스레 중복된 설정 코드가 생길 수밖에 없다. 이에 대해 자동 프록싱을 어떻게 구현할지에 대해 고민해봐야겠다.

### 마무리

이전 포스팅 "[OOP에서 AOP](https://gmun.github.io/spring/aop/oop/2019/02/09/from-oop-to-aop.html)"에서 OOP의 디자인 패턴들을 통해 관심사의 분리가 어렵다는 걸 알 수 있었고 AspectJ를 통해 관심사의 분리에 대한 문제를 근본적으로 해결할 수 있었다.

이 AspectJ는 AOP 프레임워크의 대명사라 할 수 있는데, 아드리안 콜리어(Adrian Colyer)는 AspectJ 프로젝트팀의 리더이면서 한 때는 Spring CTO까지 지냈던 사람이다. Spring AOP가 굳이 많은 AOP 프레임워크 중에 AspectJ5 라이브러리의 기반으로 구현되어있다는 부분 역시 그의 영향이 미쳤다는 걸 알 수 있는 대목이다. 하지만 `*.aj`라는 모듈로 AOP를 구현하는 AspectJ와 달리 Spring AOP는 순수 Java를 활용하여 AOP를 구현할 수 있는데 이 점에서 아드리안 콜리어는 어떻게 객체의 관계를 해결했는지 궁금해졌다.

물론 동적 Weaving을 도입한 이유에 대해 먼저 고민해봐야겠지만, 본 포스팅에선 JDK Dynamic Proxy와 FactoryBean 그리고 ProxyFactoryBean을 학습을 해보며, 아드리안 콜리어는 Spring에서 객체의 관계에 대해 어떻게 해결했는지 생각해보는 시간을 가져 보았다.

---

### 참고

- [The Aspect Blog](http://www.aspectprogrammer.org/blogs/adrian/2006/01/typed_advice_in.html)
- [The Aspect Blog - Spring 2.0 (Typed Advice in Spring 2.0)](http://www.aspectprogrammer.org/blogs/adrian/2006/01/typed_advice_in.html)
- [Spring 3.0.0 RELEASE DOC - ProxyFactoryBean](https://docs.spring.io/spring/docs/3.0.0.M4/reference/html/apbs05.html)
- [Spring Blog - Proxies impact performance](https://spring.io/blog/2007/07/19/debunking-myths-proxies-impact-performance)
- [Spring Blog - Transactions, Caching and AOP: understanding proxy usage in Spring](https://spring.io/blog/2012/05/23/transactions-caching-and-aop-understanding-proxy-usage-in-spring)
- [Stackoverflow - What is a Java ClassLoader](https://stackoverflow.com/questions/2424604/what-is-a-java-classloader)
