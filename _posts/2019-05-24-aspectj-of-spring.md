---
layout: post
title: "Spring Boot2에서 AspectJ 위빙으로 바꿔볼까?"
tags: [AOP, Spring, Spring Boot, AspectJ, Mojo, CTW, PCW, LTW, compile time weaving, post compile weaving, load time weaving]
categories: [Spring, AOP]
subtitle: "Mojo를 통해 위빙 방식으로 바꿔보자"
feature-img: "md/img/thumbnail/aspectj.png"
thumbnail: "md/img/thumbnail/aspectj.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# Mojo를 통해 위빙 방식으로 바꿔보자

---

### 들어가기전

위빙은 AOP의 꽃이라 할 수 있습니다.

우선 위빙(Weaving)이란 AOP에서 사용되는 용어로 횡단코드를 핵심코드에 적용되는 일련의 과정을 의미합니다. Spring에선 Spring AOP라하여 프록시 기반으로 런타임 위빙 방식을 따르고 있습니다.

_그렇다면 왜 위빙을 바꿔야할까?_

본 포스팅에선 Spring AOP에서 위빙을 전환해야 되는 이유와 AspectJ의 다양한 위빙 방식에 대해 알아보려합니다.

>전체 소스코드는 [GitHub](https://github.com/gmun/spring-boot-2-aspectj-demo-master)를 참조해주시기 바랍니다.

### 학습목표

1. Spring AOP의 위빙에 대한 이해
2. 다양한 AspectJ의 위빙 방식에 대한 이해와 사용법

- Spring Boot2
- Mojo(AspectJ Maven Plugin)
- JPA
- H2
- Web

### 1. Spring AOP에서, 왜 위빙을 바꿔야할까?

위빙을 바꿔야하는 이유는, Spring AOP가 프록시 메커니즘을 기반으로 동작하기 때문입니다.

Spring은 객체지향적으로 AOP 기술을 구현하기 위해 프록시 패턴의 관점을 선택했습니다. 이러한 패턴의 추상적인 관점을 구체화하기 위해, Java에서 기본적으로 제공해주고 있는 JDK Dynamic Proxy(프록시 패턴의 관점의 구현체)를 기반으로 추상적인 AOP 기술을 객체 모듈화하여 설계되어있습니다.

또한 Spring은 성숙한 AOP 기술을 제공해주기 위해 Spring 2.0 부터 @AspectJ 애노테이션 방식을 지원하였고, Aspect를 구현하는데 있어 AspectJ5 라이브러리에 포함된 일부 애노테이션을 사용할 수 있습니다. AspectJ의 강력한 AOP 기술을 Spring에서 쉽게 구현이 가능해졌기 때문에 개발자는 보다 비즈니스 개발 집중할 수 있습니다. 이는 Spring이 기존에 지향하고 있는 방향성이기 때문입니다.

물론, @AspectJ 애노테이션 방식을 통해 구현된 Aspect는 IoC 컨테이너에서 Bean으로 자동으로 관리해주고 있고 Bean에 대한 위빙이 가능합니다.

그렇다 하여 Spring AOP는 AspectJ를 대체할 수 있는 완벽한 AOP 솔루션이라 할 수 없습니다. 이러한 이유엔, 아무래도 프록시 메커니즘엔 크게 두 가지 고려해야 할 부분이 존재하기 때문입니다.

- Self Invocation
- 성능

#### 1.1. 자기 호출 이슈

프록시 메커니즘을 기반으로 한 Spring AOP에서 가장 많이 거론되는 문제는 자기 호출(Self-Invocation)에 관한 문제입니다.

![spring-aop](/md/img/aop/aspect/spring-aop.png)

우선 Spring AOP의 동작 방식을 보자면, 클라이언트가 특정 메소드를 호출할 시, 호출된 메소드만 포인트 컷에 의해 검증되고, 검증된 어드바이스 코드가 작동됩니다.

여기서 중요한 점은 타깃(Proxy Bean)에 대한 메소드가 수행할 시점엔 어떠한 Aspect가 동작하지 않습니다. 이는 프록시 메커니즘을 기반으로 한 AOP 기술에서 발생할 수 있는 공통적인 문제로써, 이를 자기 호출 문제라 합니다. Spring AOP에서 제공하는 CGLib(바이트 조작 기반)도 마찬가지로 JDK Dynamic Proxy를 기반으로 설계된 구조를 통해 동작하기 때문에 자기 호출 문제가 발생합니다.

Spring에서 자기 호출의 문제에 대한 여러 해결 방안이 존재합니다. 이 부분에 대해선, 이전에 작성한 ["Self Invocation은 왜 발생할까?"](https://gmoon92.github.io/spring/aop/2019/04/01/spring-aop-mechanism-with-self-invocation.html) 포스팅을 참고해주시면 감사하겠습니다.

#### 1.2. 성능 이슈

두 번째 문제는 성능에 관련된 문제입니다.

Spring AOP는 런타임 시점(메소드 호출 시점)에 타깃에 대한 메소드 호출을 가로채고 내부적인 AOP 프로세스에 의해 어드바이스를 타깃의 메소드와 하나의 프로세스로 연결합니다.

![spring-aop](/md/img/aop/aspect/spring-aop2.png)

이러한 형태를 사슬(Chain) 모양을 띄고 있다하여 어드바이스 체이닝이라 합니다. 이 체이닝은 런타임시 순차적으로 어드바이스 코드를 실행을 하고 다음 클라이언트가 원하는 로직(타깃의 메소드)이 수행됩니다. 이 점은 프록시 패턴의 특징인 타깃에 대한 안정성을 보장 받을 수 있다는 측면이라 볼 수 있습니다.

하지만 Aspect가 늘어날수록 자연스레 실질적으로 수행될 타깃의 메소드는 늦어질 수밖에 없고, 이는 결과적으로 성능에 관한 문제로 직결됩니다.

### 2. AspectJ의 바이트 코드 조작

프록시 메커니즘을 가지고 있던 문제는 AspectJ 위빙 방식으로 전환하면 모든 문제를 해결할 수 있습니다. 이러한 이유엔 기본적으로 AspectJ는 바이트 코드을 기반으로 위빙하기 때문에 성능 문제와 더불어 자기 호출에 대한 문제를 해결할 수 있습니다.

- 완벽한 AOP 솔루션을 제공
- 프록시 객체를 동적으로 구성하는 방식이 아닌, 타깃의 바이트 코드를 조작

### 3. AspectJ 위빙의 조건

Spring에서 AspectJ로 전환하기 위해선 4 가지 조건이 필요합니다.

1. AspectJ 형식의 Aspect
2. AspectJ Runtime
3. AspectJ Weaver
4. AspectJ Compiler(AJC)

#### 3.1. AspectJ 형식의 Aspect

우선 첫 번째는 AspectJ 형식으로 구현된 Aspect가 필요합니다.

- `.aj` 파일로 구현된 Aspect
- Java 파일로 구현된 Aspect(@AspectJ 애노테이션 기반)

`*.aj` 확장자를 띈 파일로 구현된 Aspect는 순수한 AspectJ의 Aspect입니다.

``` java
public aspect OriginalAspect{
    // 포인트컷
    pointcut pcd()
      : call(* ..*Service.method(..));

    // 어드바이스
    void around()
      : pcd() // 포인트컷 정의
    {
          proceed();
    }
}
```

이 `*.aj` 파일의 코드 스타일은 Java와 비슷하면서도 다르고, AspectJ의 모든 기능을 사용할 수 있습니다. 커스텀마이징이 가능한 만큼 고려할 사항도 많고, 초기 학습에 대한 진입 장벽이 높고 어렵습니다.

하지만 Java 형식으로도 AspectJ 형식의 Aspect를 구현할 수 있습니다.

흔히 Spring AOP에서 흔히 사용하고 있는 @AspectJ 애노테이션 스타일의 Aspect는 AspectJ5 라이브러리의 일부 애노테이션을 사용하는 방식으로 전형적인 AspectJ의 형식입니다.

``` java
@Aspect
@Component
public class SpringAOPAspect{
    ...
}
```

일반적으로 개발자분들이 @AspectJ 애노테이션 스타일로 Aspect를 구현했던 이유는 쉬운 접근성도 있지만 AspectJ 위빙으로 전환할 때 AspectJ와 완벽히 호환되기 때문입니다.

#### 3.2. AspectJ Runtime

그 다음 AspectJ Runtime이 필요합니다.

AspectJ Runtime를 구성하기 위해선 AspectJ Runtime 라이브러리인 `aspectjrt.jar` 의존성만 추가시켜주면 됩니다.

- [Maven AspectJ Runtime](https://mvnrepository.com/artifact/org.aspectj/aspectjrt)

``` xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.9.4</version>
</dependency>
```

AspectJ로 구성된 애플리케이션을 실행하게 되면, AspectJ Compiler(AJC)엔 위빙할 객체의 정보가 포함되어 있고, AspectJ Runtime은 AJC에 포함된 객체의 정보를 토대로 위빙된 코드를 타깃에게 적용합니다.

> AspectJ Runtime에 대한 자세한 내용은 아드리안 콜리어가 답변해준 다음 [링크](https://www.eclipse.org/lists/aspectj-users/msg02750.html)를 참조해주시기 바랍니다.

#### 3.3. AspectJ Weaver

Aspect Runtime과 더불어 Aspect Weaver 라이브러리인 `aspectjweaver.jar`를 추가해줘야 합니다.

- [Maven AspectJ Weaver](https://mvnrepository.com/artifact/org.aspectj/aspectjweaver)

``` xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

Aspect Weaver 라이브러리에는 @Aspect, @Pointcut, @Before, @Around... 등 AspectJ 5 라이브러리에 속한 애노테이션들을 포함하고 있기 때문에, Spring AOP에서 @AspectJ 애노테이션 스타일 방식을 사용하기 위해선 반드시 Aspect Weaver 의존성을 추가해줘야 했습니다.

![spring-aop](/md/img/aop/aspect/weaver.png)

AspectJ Weaver는 Aspect와 타깃의 바이트 코드를 위빙하고, 위빙된 바이트 코드를 컴파일러에게 제공하는 역할을 하고 있습니다.

#### 3.4. AspectJ Compiler

마지막으로 AspectJ Compiler(AJC)라는 컴파일러가 필요합니다.

AJC는 Java Compiler를 확장한 형태의 컴파일러로써, AspectJ는 AJC를 통해 Java 파일을 컴파일하며, 컴파일 과정에서 타깃의 바이트 코드 조작(어드바이스 삽입)을 통해 위빙을 수행합니다.

- [AspectJ Development Tool(AJDT)](https://www.eclipse.org/aspectj/downloads.php#ides)
   - Eclipse에서 지원하는 툴(AJC가 내장되어있음)
- [Mojo](http://www.mojohaus.org/aspectj-maven-plugin/ajc_reference/standard_opts.html)
   - AspectJ Maven Plugin

AJC는 이클립스에서 지원하는 [AspectJ Development Tool(AJDT)](https://www.eclipse.org/aspectj/downloads.php#ides)을 활용하여 Aspect를 개발하면 됩니다. 하지만 미리 개발이 되었거나 상황이 안된다면, Mojo의 AspectJ Maven Plugin을 사용하면 됩니다. 본 포스팅에선 [Mojo 플러그인](http://www.mojohaus.org/aspectj-maven-plugin/ajc_reference/standard_opts.html)을 사용하여 Spring Boot에서 AspectJ 위빙 방식으로 전환해보려 합니다.

### 4. AspectJ의 Weaving

AspectJ는 3 가지 위빙 방식을 지원합니다.

- CTW(Compile Time Weaving) : AJC(AspectJ Compiler)를 이용해서, 소스 코드가 컴파일할 때 위빙
- PCW(Post Compile Weaving) : 이미 컴파일된 바이너리 클래스에 위빙
- LTW(Load Time Weaving) : Class Loader가 클래스를 로딩할 때 위빙 (Weaving Agent 필요)

> 참고 - [eclipse.aspectj](https://www.eclipse.org/aspectj/doc/next/devguide/ltw.html)

#### 4.1. AJC를 이용해서 컴파일 시점에 위빙 (Compile Time Weaving, CTW)

우선 CTW는 3 가지 위빙 중에서는 가장 빠른 퍼포먼스를 보여줍니다.

CTW는 타깃의 코드가 JVM 상에 올라갈 때(컴파일 시점) 바이트 코드를 직접 조작하여, 타깃의 메소드 내에 어드바이스 코드를 삽입시켜 주기 때문입니다. 또한, 컴파일 시점에만 바이트 코드를 조작하여 호출된 타깃의 메소드는 조작된 코드가 수행되기 때문에 정적인 위빙 방식이라 합니다.

Mojo의 AspectJ Maven Plugin의 플러그인 설정 방식은 다음과 같습니다.

``` xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>${mojo.version}</version>
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
> 자세한 모조 플러그인의 설정 옵션들은 다음 [링크 : Mojo - AspectJ 설정 옵션](http://www.mojohaus.org/aspectj-maven-plugin/ajc_reference/standard_opts.html)를 참조해주시기 바랍니다.

``` java
@Test
public void testSimpleProfilingAspect() {
    System.err.println(orderService.getClass());
    orderService.orderGoods(goodsList);
    // ^--- @Transactional orderGoods(..)

    System.err.println(testService.getClass());
    testService.testMethod(goodsList);
    // ^--- testMethod(..)
}
```
``` html
class com.moong.postcompileweaving.service.OrderService$$EnhancerBySpringCGLIB$$ade9b7af
...
class com.moong.postcompileweaving.service.TestService
...
```

주의할 점은 @Transactional이 부착된 메소드에 대해선 에러가 발생하지 않지만, CTW 위빙 방식이 아닌 프록시 기반으로 동작한다는 점입니다. 물론 해결 방안은 존재합니다. 몇 가지 설정을 더 해줘야 하지만 본문에선 다루지 않겠습니다.

>참고 - [LTW, CTW를 이용한 Transactional의 사용](https://netframework.tistory.com/entry/LTW-CTW%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-Transactional%EC%9D%98-%EC%82%AC%EC%9A%A9)

#### 4.2. JAR에 위빙 (Post Compile Weaving, PCW)

CTW가 컴파일 시점에 위빙을 한다면, PCW는 컴파일 직후에 위빙을 합니다.

주로 PCW는 컴파일 된 바이너리 코드 또는 JAR에 포함된 소스에 위빙하는 목적으로 사용됩니다. PCW를 하기 위해선 Mojo의 AspectJ Maven Plugin의 플러그인 설정에서 위빙할 JAR 파일을 설정해야합니다.

``` xml
<configuration>
    <weaveDependencies>
        <!-- [1] JAR 설정 -->
        <weaveDependency>
            <groupId>...</groupId>
            <artifactId>...</artifactId>
        </weaveDependency>

        <!-- [2] JAR 설정 -->
        <weaveDependency>
            <groupId>...</groupId>
            <artifactId>...</artifactId>
        </weaveDependency>
    </weaveDependencies>
</configuration>
```
> 참고 - [Mojo Plugin AspectJ Weaver JAR ](http://www.mojohaus.org/aspectj-maven-plugin/examples/weaveJars.html)

#### 4.3. 클래스 로더를 이용한 위빙 (Load Time Weaving, LTW)

마지막으로 LTW는 JVM에 클래스가 로드되는 시점에 위빙을 합니다.

LTW는 RTW(Runtime Weaving)처럼 바이트 코드에 직접적으로 조작을 가하지 않기 때문에 컴파일 시간은 상대적으로 CTW와 PCW보다 짧지만, 오브젝트가 메모리에 올라가는 과정에서 위빙이 일어나기 때문에 런타임 시 위빙 시간은 상대적으로 느립니다.

- Compile Time
    - CTW < PCW < LTW
- Runtime
    - LTW < PCW < CTW

#### 4.3.1 LTW 설정

LTW는 CTW, PCW와 달리 상대적으로 설정들이 다소 복잡합니다.

> 참고 - [https://www.eclipse.org/lists/aspectj-users/msg09286.html](https://www.eclipse.org/lists/aspectj-users/msg09286.html)

#### 4.3.1. AspectJ Weaver 설정

우선 LTW는 Java Agent의 도움을 받아 위빙될 클래스를 구성해야 합니다.

<img src="/md/img/aop/aspect/jvm-agent.png" style="max-height:none;">

Java Agent는 JVM에 의해 로드되는 동안 클래스를 인터셉트합니다. 인터셉트 된 클래스는 `aop.xml`이라는 메타 파일에 포함된 AspectJ 설정을 기반으로 Agent에 의해 바이트 코드가 수정됩니다.

`aop.xml` 파일은 `META-INF` 폴더를 생성하여 추가해줘야 합니다.

- `src/main/resources/META-INF/aop.xml`
- `src/test/resources/META-INF/aop.xml`

``` xml
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "http://www.eclipse.org/aspectj/dtd/aspectj.dtd">
<aspectj>
  <weaver options="-Xset:weaveJavaxPackages=true -verbose -showWeaveInfo -debug">
      <include within="com.moong.loadtimeweaving.service.*"/>
  </weaver>
  <aspects>
      <aspect name="com.moong.loadtimeweaving.aspect.ProfilingAspect"/>
  </aspects>
</aspectj>
```

- `<weaver>` : 위빙될 모든 클래스를 정의
- `<aspects>` : LTW의 위빙 과정에서 사용되는 모든 Aspect 요소를 정의

#### 4.3.2. Agent 옵션 활성화, AspectJ Maven Plugin

그 다음 LTW를 하기 위해 AspectJ 설정을 활성화해줘야 합니다.

JVM에서 `-javaagent:[경로]/aspectjweaver-${aspectj.version}.jar` 지정하여 Java Agent 옵션을 직접적으로 활성화할 수 있지만, `maven-surefire-plugin` 플러그인을 사용하면 보다 쉽게 설정할 수 있습니다.

`maven-surefire-plugin` 플러그인은 일반적으로 JVM에게 인수를 전달해주는 설정으로써 사용되며, 여기선 IDE(ex Eclipse, IntelliJ)에서 Spring Boot 애플리케이션을 실행하는 경우에만 사용할 수 있습니다.

``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.10</version>
    <configuration>
        <argLine>
            -javaagent:"${settings.localRepository}"/org/aspectj/
            aspectjweaver/${aspectj.version}/
            aspectjweaver-${aspectj.version}.jar
        </argLine>
        <useSystemClassLoader>true</useSystemClassLoader>
        <forkMode>always</forkMode>
    </configuration>
</plugin>
```

다음과 같이 `<argLine>` 요소에 JVM에 전달할 -javaagent 설정을 추가해주시면 됩니다.

- [Mojo - AspectJ 설정 옵션](http://www.mojohaus.org/aspectj-maven-plugin/ajc_reference/standard_opts.html)
- [Mojo - include, exclude 옵션](https://www.mojohaus.org/aspectj-maven-plugin/examples/includeExclude.html)

#### 4.3.3. 실행

마지막으로 실행에 앞서 Maven을 Clean하고, 다시 Update Maven Project를 진행해줍니다.

실행할땐 JVM의 VM에 argument에 `-javaagent` 옵션을 추가해줘야 합니다.

``` xml
-javaagent:${settings.localRepository}/aspectjweaver-${aspectj.version}.jar
-javaagent:${settings.localRepository}/spring-instrument-${spring.version}.jar
```

- ${settings.localRepository} : 라이브러리 경로
- ${spring.version} : Spring 버전
- ${aspectj.version} : AspectJ 버전

### 마무리

Spring Boot2에서 AspectJ 형식으로 위빙을 전환했습니다.

>전체 소스코드는 [GitHub](https://github.com/gmun/spring-boot-2-aspectj-demo-master)를 참조해주시기 바랍니다.

가장 큰 특징으론 AspectJ는 ACJ의 기반으로 동작이 됨으로 Lombok과 같이 컴파일 과정에서 코드를 조작하는 플러그인을 같이 사용할 경우, 컴파일 과정에서 서로 충돌할 가능성이 큽니다.

물론 해결 방안으로 Mojo 플러그인의 설정을 통해 해결할 수 있지만, 무엇보다 AspectJ에선 DI의 부분을 해결하기 위해 설정이 복잡하고 어렵습니다.

``` java
@Aspect
public class Aspect(){
  @Autowired
  private MemberService memberService;
            ^----- DI가 적용되지 않는다.
}
```

Spring AOP은 프록시 기반이기 때문에 오버 헤드가 발생한다곤 하지만, IoC Container와 완전 호환됨으로 DI를 활용할 수 없습니다. AspectJ와 Spring AOP의 장단점을 인지하고 사용하는것이 무엇보다 중요한것 같습니다.

---

### 참고

- [Spring Baeldung - AspectJ](https://www.baeldung.com/aspectj)
- [Spring Baeldung - Spring AOP vs AspectJ](https://www.baeldung.com/spring-aop-vs-aspectj)
- [Spring Baeldung - java asm](https://www.baeldung.com/java-asm)
- [AOP Architecture](http://www.fsl.cs.sunysb.edu/ssw/research.html)
- [klevas - aopnotes](https://klevas.mif.vu.lt/~plukas/resources/Metaprogramming/aopnotes.pdf)
- [Toby - Spring 3.0 (11) Aspects 모듈의 선택 라이브러리 분석](http://toby.epril.com/?p=620)
- [WhiteShip - 스프링 AOP, 선택, 활용, 이슈](https://slidesplayer.org/slide/11065231/)
- [doanduyhai - Anti AspectJ](https://doanduyhai.wordpress.com/2011/12/12/advanced-aspectj-part-ii-inter-type-declaration/)
- [kezhuw - Spring AspectJ LTW](http://blog.kezhuw.name/2017/08/31/spring-aspectj-load-time-weaving/)
- [Credera - Spring JDK Proxies vs CGLIB vs AspectJ](https://www.credera.com/blog/technology-insights/open-source-technology-insights/aspect-oriented-programming-in-spring-boot-part-2-spring-jdk-proxies-vs-cglib-vs-aspectj/)
- [howtodoinjava - Top Spring AOP Interview Questions with Answers](https://howtodoinjava.com/interview-questions/top-spring-aop-interview-questions-with-answers/)
- [nurkiewicz - CTW vs LTW Performance in Spring](https://www.nurkiewicz.com/2009/10/yesterday-i-had-pleasure-to-participate.html)
https://github.com/tokuhirom/java-samples/tree/master/aspectj-post-compile-weaving
- [Slide Share - AOP LTW](https://www.slideshare.net/AnselmKim/3-aop-ltw)
