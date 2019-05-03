---
layout: post
title: "Self Invocation은 왜 발생할까?"
tags: [AOP, SpringAOP, Self-invocation]
categories: [Spring, AOP]
subtitle: "Spring 프록시의 메커니즘"
feature-img: "md/img/thumbnail/aop-proxy-self-invocation.png"
thumbnail: "md/img/thumbnail/aop-proxy-self-invocation.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# Spring 프록시의 메커니즘과 자기 호출의 해결책

---

### 들어가기전

Spring AOP의 JDK 다이내믹 프록시와 CGLIB는 프록시를 기반으로 AOP를 적용한다. 이외에도 Spring에선 다양한 방면에서 프록시의 메커니즘을 기반으로 기술을 제공하고 있는데, 대표적으로 Cache, Transaction이 이에 해당한다. 무엇보다 이 프록시 메커니즘의 가장 큰 이슈는 Self-Invocation이다.

마찬가지로 Spring AOP도 같은 이슈가 발생하는데 본 포스팅에선 Self-Invocation 발생하는 근본적인 원인과 이를 해결하고자 한다.

학습 환경으론 Spring Boot 2.1.4에서 진행하였고 학습 과정에서 사용했던 코드는 [GitHub](https://github.com/gmun/aop-proxy-self-invocation)를 참고하기 바란다.

### 학습목표

1. 프록시의 메커니즘에 대한 이해
2. Self-Invocation에 대한 해결책

### 1. Spring AOP의 동작 과정

Spring AOP는 프록시 기반으로 IoC 컨테이너에서 빈을 생성하는 시점에 AOP를 적용할지 여부를 판단하여 프록시 빈을 생성해준다.

이 과정에서 JDK Proxy와 CGLIB 둘 중에 선택 해서 프록시 빈을 생성하는 데, 일반적으로 타깃이 인터페이스를 구현하고 있는 경우 JDK Proxy의 방식으로 프록시를 생성해주고 타깃의 객체가 인터페이스를 구현하지 않으면 CGLIB 방식으로 프록시가 생성된다.

이 프록시의 핵심적인 기능은 지정된 메소드가 호출(Invocation)될 때 이 메소드를 가로채어 부가기능들을 추가할 수 있도록 지원하는 것이다.

### 2. Self-Invocation이 뭔데

CGLIB 방식이든 JDK Proxy 방식이든 Spring AOP는 프록시를 기반으로 AOP를 적용한다는 점을 생각해봐야 한다. 그렇다면 Self-Invocation가 뭘까? 다음 코드를 살펴보자.

``` java
@Component
public class BusinessI implements Business{
    @Override
    public void ready() {
        System.out.println("ready...");
        this.go(); ← go() 메소드 호출
    }

    @Override
    public void go() {
        System.out.println("go...");
    }
}
```

다음 코드에서 BusinessI.ready()에 보면 this.go() 메소드를 호출하는걸 알 수 있다. 즉 같은 객체의 자신의 메소드 외의 다른 메소드를 호출을 의미한다. 이러한 호출을 Self-Invocation이라 하고 프록시에서 이 Self-Invocation 문제가 발생한다.

### 3. Self-Invocation를 강제로 발생시키자

예를들어 Self-Invocation를 발생하기 위해 go() 메소드에 Advice를 제공한다고 가정해보자.

``` java
@Aspect
@Component
public class MyAspect {

    @Pointcut("execution(void com.moong.aopproxies.business.Business*.go(..))")
    public void myPointcut() { }

    @Before("myPointcut()")
    public void before() {
        System.err.println("Advice logic...");
    }
}
```

- BusinessI.go() 메소드에 Beafor Advice 설정

간단하게 @AspectJ를 사용하여 Spring AOP를 구성해보았다. 이제 테스트 코드를 작성해보자.

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
@EnableAspectJAutoProxy
public class SelfInvocationTest {
    @Autowired private Business businessI;

    @Test
    public void isSelefInvocation() {
        assertTrue(Proxy.isProxyClass(businessI.getClass()));  ← 프록시 객체 생성여부
        System.out.println(businessI.getClass());
        businessI.ready();
    }
}
```

테스트 코드를 실행하기에 앞서, 흔히 Self-Invocation의 오류를 범하는 사람들의 예상 시나리오 다음과 같다.

``` html
class com.sun.proxy.$Proxy48
ready...        ← [1] businessI.ready() 메소드 호출
Advice logic... ← [2] AOP에 의해 @Before 어드바이스 호출
go...           ← [3] go() 메소드 호출
```

하지만 테스트 코드를 실행해보면 결과는 @Before 어드바이스를 호출하지 않는다.

``` html
class com.sun.proxy.$Proxy48
ready...
go...
```

이게 바로 프록시에서 흔히 발생하는 Self-Invocation 이슈이다. "왜 Self-Invocation의 문제가 발생할까"에 대한 의문점을 해결하기 위해선 프록시의 메커니즘을 이해하면 도움이 된다.

### 4. 프록시의 매커니즘

<img src="/md/img/aop/selef-invocation/proxy1.png" style="max-height:none; padding:0px;">

``` java
@Test
public void isSelefInvocation() {
  assertTrue(Proxy.isProxyClass(businessI.getClass())); ← 프록시 객체 생성여부
  System.out.println(businessI.getClass());
  businessI.ready(); ← 프록시 객체의 ready() 호출
}
```
``` html
class com.sun.proxy.$Proxy48 ← 프록시 객체의 참조
ready...  ← [1][2] 프록시 객체의 ready() 메소드 호출
go...     ← [3] 타깃의 go() 메소드 호출
```

다음 그림에서 중요한 점은 businessI의 메소드 호출이 Spring AOP에 의해 생성된 프록시를 참조한다는 점이다. 따라서 클라이언트는 $Proxy48.ready()를 호출하게 되고, 프록시에 의해 호출된 메소드와 Pointcut을 비교하여 Advice를 수행하게 된다.

하지만 ready() 메소드 호출이 끝이나고 go()가 호출되는 시점이 되면, 더이상 프록시를 참조하지 않고 타깃을 참조하게 된다. 즉 **`$Proxy48.go`** 가 아닌 BusinessI를 참조하는 **`this.go()`** 가 호출 되어지기 때문에, 결과적으론 Self-Invocation으로 호출된 메소드는 AOP가 적용할 수 없다.

### 5. Self-Invocation 해결 방안들

그렇다면 Self-Invocation의 해결 방법은 없을까?

- AopContext
- IoC 컨테이너 Bean 활용
- AspectJ Weaving

#### 5.1. AopContext

첫 번째 방법으론 AopContext 추상화 클래스를 사용하는 것이다.

AopContext는 현재 AOP 호출에 대한 정보를 얻기 위해 사용되는 추상 클래스이다. 이 클래스의 currentProxy() 메소드는 현재 AOP 프록시 반환해주는데 이를 이용하면 Self-Invocation 문제를 해결할 수 있다.

``` java
@Component
public class BusinessI implements Business{

    @Override
    public void ready() {
        System.out.println("ready...");
        //go();
        ((Business) AopContext.currentProxy()).go(); ← 호출된 프록시 객체를 활용
    }

    @Override
    public void go() {
        System.out.println("go...");
    }
}
```

_`$Proxy`.ready() → `$Proxy`.go()_

다음 코드처럼 호출된 AOP 프록시를 활용하여 go() 메소드를 호출해주면 된다. 즉 기존의 타깃을 참조되던 this.go() 메소드를 프록시로 참조할 수 있도록 AopContext.currentProxy()를 활용한 것이다. 이 AopContext.currentProxy() 메소드를 사용하기 위해선 expose-proxy 옵션을 활성화 해줘야 한다.

- <aop:aspectj-autoproxy expose-proxy="true"/>
- ProxyFactory.setExposeProxy(true)

일반적으로 자동 프록싱 XML 태그인 `&lt;aop:aspectj-autoproxy&gt;`의 expose-proxy 옵션을 통해 활성화해주는 방법과 ProxyFactory의 setExposeProxy() 메소드를 활용하는 방법이 있다. 하지만 이 방법들론 expose-proxy을 활성화할 수 없다. 첫 번째 이윤 본 포스팅의 학습 환경이 Spring Boot이기 때문에 XML 설정을 할 수 없다. 마지막으로 @AspectJ 어노테이션으로 Spring AOP를 구축했기 때문에 ProxyFactory 방식을 사용할 수 없다.

Spring Boot에선 @EnableAspectJAutoProxy 어노테이션을 활용하여 자동 프록싱을 지원하는데, 이를 활용해보자.

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
@EnableAspectJAutoProxy(exposeProxy=true) ← expose-proxy 옵션 활성화
public class SelfInvocationTest {

    @Autowired private Business businessI;

    @Test
    public void isSelefInvocation() {
        assertTrue(Proxy.isProxyClass(businessI.getClass()));
        System.out.println(businessI.getClass());
        businessI.ready();
    }
}
```
``` html
class com.sun.proxy.$Proxy48
ready...
Advice logic...
go...
```

- AopContext.currentProxy()
- expose-proxy 활성화

#### IoC 컨테이너 Bean 활용

두 번째 방법으론 IoC 컨테이너에 등록된 자기 자신의 빈을 활용하는 방법이다.

- @PostConstruct 와 @Autowired ApplicationContext context
- @Autowired
- @Resource
- @Inject

이 접근할 수 있는 방식은 다양하지만 @Resource를 사용하여 Self-Invocation을 해결해보자.

``` java
@Component
public class BusinessWithResource implements Business{
    @Resource(name="businessWithResource") ← 자기 자신의 빈을 주입 받는다.
    Business self;

    @Override
    public void ready() {
        System.out.println("ready...");
        self.go(); ← DI된 자기 자신의 Bean을 이용해 다시 호출
    }

    @Override
    public void go() {
        System.out.println("go...");
    }
}
```

다음 코드를 보면 @Resource 어노테이션을 사용하여 빈을 주입 받는다. 이는 자기 자신의 빈 객체를 복사본을 활용한다고 생각하면 된다.

메소드 호출은 this.go() 방식이 아닌 self.go() 방식으로 호출하기 되기 때문에 Proxy를 통해 호출하는 효과를 만드는 방법이다. 테스트를 통해 검증해보자.

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
@EnableAspectJAutoProxy
public class ResourceTest {

    @Autowired private Business businessWithResource;

    @Test
    public void isSelefInvocation() {
        System.out.println(businessWithResource.getClass());
        businessWithResource.ready();
    }
}
```
``` html
class com.sun.proxy.$Proxy48
ready...
Advice logic...
go...
```

#### 5.3. AspectJ Weaving

마지막으론 Spring AOP의 Weaving 방식을 AspectJ Weaving 방식으로 바꾸는 것이다.

AspectJ Weaving은 Spring AOP와 달리 바이트 코드를 조작하는 방식이기 때문에 Proxy의 Self-Invocation 이슈가 발생하지 않는다. 물론 Self-Invocation를 해결하기 위해 기존 코드를 수정하는 작업이 필요하지 않다.

무엇보다 본 포스팅에서 @AspectJ 어노테이션으로 Spring AOP를 구현한 이유이기도 하다. AspectJ의 Weaving 방식에는 CTW, PTW, LTW 세 가지 방식이 있는데 그중에 CTW 방식으로 변환해보겠다.

>@AspectJ 어노테이션은 AspectJ 5 라이브러리에 포함된 어노테이션으로 `aspectjweaver.jar` 라이브러리를 추가하면 된다.

#### 5.3.1. Maven 추가

먼저 AspectJ의 Weaving 방식으로 변환하기 위해선 `aspectjweaver.jar`와 더불어 AspectJ 런타임 라이브러리인 `aspectjrt.jar`가 필요하다.

``` xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>${org.aspectj.version}</version>
</dependency>
```

#### 5.3.2. Plugin 설정

AspectJ는 기본적으로 AJW(AspectJ Compiler)에 의해 Weaving을 처리하기 때문에, AJW가 내장되어 있는 [AspectJ Development Tool](https://www.eclipse.org/aspectj/downloads.php#ides)을 사용하여 개발을 해야한다. 하지만 본 포스팅처럼 미리 개발이 되어있는 상태된 상태라면 [Mojo의 AspectJ Maven Plugin](http://www.mojohaus.org/aspectj-maven-plugin/ajc_reference/standard_opts.html) 플러그인을 사용하면 된다.

``` xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.10</version>
    <configuration>
        <encoding>UTF-8 </encoding>	<!-- 인코딩 -->
        <source>${java.version}</source> <!-- source level : 1.3 to 1.8 -->
        <target>${java.version}</target> <!-- classfile : 1.1 to 1.8 -->
        <complianceLevel>${java.version}</complianceLevel>
        <showWeaveInfo>true</showWeaveInfo> <!-- 위빙에 대한 정보를 알기 위함 -->
        <verbose>true</verbose>
        <Xlint>ignore</Xlint>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>test-compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

#### 5.3.3. CTW 검증

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
@EnableAspectJAutoProxy
public class CTWConfigTest {

    @Autowired
    private Business businessI;

    @Test
    public void selfInvocation() {
        System.out.println(businessI.getClass());
        businessI.ready();
    }
}
```
``` html
class com.moong.selfinvocationctw.business.BusinessI ← 프록시가 아님
ready...
Adivce logic...
go...
```

결과적으로 별도의 수정없이 Weaving 방식만 변경하여 Self-Invocation 문제를 해결하였다. 마지막으로 class 파일을 확인해보자.

``` java
@Component
public class BusinessI
  implements Business
{
    public void ready()
    {
        System.out.println("ready...");
        go();
    }

    public void go()
    {
        MyAspect.aspectOf().myBefore();System.out.println("go..."); ← AspectJ에 의해 class 코드가 변경
    }
}
```

### 마무리

프록시 기반의 기술에서 흔히 발생하는 Self-Invocation에 대한 원인과 해결 방안을 살펴보았다.

AspectJ의 Weaving을 활용하여 코드의 변경 없이 AOP를 적용할 수 있었다. 하지만 "특정 @AspectJ만 설정 파일만 AspectJ Weaving을 적용할 순 없을까?"라는 의문이 생겼다. 이에 대해 [Mojo - IncludeExclude](https://www.mojohaus.org/aspectj-maven-plugin/examples/includeExclude.html)을 참고하여 학습하여 추후 AspectJ Weaving에 대해 포스팅하면 좋을 것 같다.

---

## 참고

- [Spring DOC 4.0.1 - AOP Proxies](https://docs.spring.io/spring/docs/4.0.1.RELEASE/spring-framework-reference/htmlsingle/#aop-understanding-aop-proxies)
- [Spring DOC - @EnableAspectJAutoProxy](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/EnableAspectJAutoProxy.html)
- [Spring Blog - AspectJ](https://www.baeldung.com/aspectj)
- [Mojo - AspectJ Compiler reference: standard options](http://www.mojohaus.org/aspectj-maven-plugin/ajc_reference/standard_opts.html)
- [Github Issues - @EnableAspectJAutoProxy can't set 'expose-proxy' property.](https://github.com/spring-projects/spring-boot/issues/6113)
- [Github Issues - Support @Autowired-like self injection](https://github.com/spring-projects/spring-framework/issues/13096)
