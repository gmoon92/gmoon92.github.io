---
layout: post
title: "JDK Dynamic Proxy와 CGLIB의 차이점은 무엇일까?"
tags: [AOP, Spring AOP, Proxy, JDK Dynamic Proxy, CGLIB]
categories: [Spring, AOP]
subtitle: "Spring AOP가 제공하는 두 가지 AOP Proxy"
feature-img: "md/img/thumbnail/spring-aop.png"
thumbnail: "md/img/thumbnail/spring-aop.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# Spring AOP가 제공하는 두 가지 AOP Proxy

---

### 들어가기전

Spring AOP는 프록시 기반으로 JDK Dynamic Proxy와 CGLIB을 활용하여 AOP 제공하고 있습니다.

- 어떠한 차이가 있을까?
- 왜 두 가지 방식을 제공하는데?

본 포스팅은 다음 두 가지의 의문점을 가지고 작성했습니다. 또한, 이 둘의 Spring AOP의 방식에 대해 비교/분석을 해보며, 무엇보다 이들의 차이에 대해 명확히 이해하고 사용하기 위해 작성하게 되었습니다.

### 학습목표

1. JDK Dynamic Proxy와 CGLIB의 차이를 알아보자.
2. Spring AOP가 제공하는 두 가지 AOP Proxy를 구분해서 사용하자.

### 1. IoC 컨테이너와 AOP Proxy의 관계

먼저 Spring AOP는 Proxy의 메커니즘을 기반으로 AOP Proxy를 제공하고 있습니다.

![img](/md/img/aop/jdk-dynamic-proxy-and-cglib/aop-proxy-mechanism.png)

다음 그림처럼 Spring AOP는 사용자의 특정 호출 시점에 IoC 컨테이너에 의해 AOP를 할 수 있는 Proxy Bean을 생성해줍니다. 동적으로 생성된 Proxy Bean은 타깃의 메소드가 호출되는 시점에 부가기능을 추가할 메소드를 자체적으로 판단하고 가로채어 부가기능을 주입해주는데요. 이처럼 호출 시점에 동적으로 위빙을 한다 하여 런타임 위빙(Runtime Weaving)이라 합니다.

따라서 Spring AOP는 런타임 위빙의 방식을 기반으로 하고 있으며, Spring에선 런타임 위빙을 할 수 있도록 상황에 따라 JDK Dynamic Proxy와 CGLIB 방식을 통해 Proxy Bean을 생성을 해주는데요. 그렇다면 이 두 가지 AOP Proxy는 어떠한 상황에 생성하게 되는걸까요?

### 2. 두 가지 AOP Proxy는 어떠한 상황에 생성하게 될까?

가장 쉽게 설명을 드리자면 Spring은 AOP Proxy를 생성하는 과정에서 자체 검증 로직을 통해 타깃의 인터페이스 유무를 판단합니다.

>Target이란 횡단기능(Advice)이 적용될 객체(Object)를 뜻합니다. AOP의 개념적인 용어들은 이전에 작성된 [AOP 개념편](https://gmun.github.io/spring/aop/2019/01/15/aspect-oriented-programming-concept.html) 포스트를 참고해주세요.

![img](/md/img/aop/jdk-dynamic-proxy-and-cglib/aop-proxy-mechanism2.png)

이때 만약 타깃이 하나 이상의 인터페이스를 구현하고 있는 클래스라면 JDK Dynamic Proxy의 방식으로 생성되고 인터페이스를 구현하지 않은 클래스라면 CGLIB의 방식으로 AOP 프록시를 생성해줍니다.

### 3. Spring AOP의 근간이 되는 JDK Dynamic Proxy 방식

우선 JDK Dynamic Proxy란 Java의 리플렉션 패키지에 존재하는 Proxy라는 클래스를 통해 생성된 Proxy 객체를 의미합니다. 그렇다면 왜 이름이 JDK Dynamic Proxy일까요?

이유는 단순합니다. 리플랙션의 Proxy 클래스가 동적으로 Proxy를 생성해준다하여 우리가 아는 JDK Dynamic Proxy라 불리는거죠. 이 클래스를 사용하여 Proxy의 생성하기 위해선 몇 가지 조건이 있지만, 아무래도 그 중 핵심은 타깃의 인터페이스를 기준으로 Proxy를 생성해준다는 점입니다.

>Spring AOP defaults to using standard JDK dynamic proxies for AOP proxies. - [Spring Reference](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-introduction
)

무엇보다 Spring AOP는 JDK Dynamic Proxy를 기반으로 AOP 기술을 구현했을 만큼, JDK Dynamic Proxy가 어떻게 Proxy를 생성하는지에 대한 부분은 Spring AOP를 통해 Aspect를 구현하신다면 꼭 짚고 넘어가야할 부분입니다.

#### 3.1. JDK Dynamic Proxy의 Proxy

먼저 JDK Dynamic Proxy를 사용하여 Proxy 객체를 생성하는 방법은 간단합니다.

``` java
Object proxy = Proxy.newProxyInstance(ClassLoader       // 클래스로더
                                    , Class<?>[]        // 타깃의 인터페이스
                                    , InvocationHandler // 타깃의 정보가 포함된 Handler
              										  );
```

단순히 리플랙션의 Proxy 클래스의 newProxyInstance 메소드를 사용하면 되는데요. JDK Dynamic Proxy가 이 파라미터를 가지고 Proxy 객체를 생성해주는 과정은 다음과 같습니다.

![img](/md/img/aop/jdk-dynamic-proxy-and-cglib/jdk-dynamic-proxy1.png)
_타깃의 인터페이스를 구현하여 Proxy 생성_
1. 타깃의 인터페이스를 자체적인 검증 로직을 통해 ProxyFactory에 의해 타깃의 인터페이스를 상속한 Proxy 객체 생성
2. Proxy 객체에 InvocationHandler를 포함시켜 하나의 객체로 반환

다음과 같이 Proxy를 생성하는 과정에서 핵심적인 부분은, 무엇보다 인터페이스를 기준으로 Proxy 객체를 생성해준다는 점입니다. 따라서 구현체는 인터페이스를 상속받아야하고, @Autowired를 통해 생성된 Proxy Bean을 사용하기 위해선 반드시 인터페이스의 타입으로 지정해줘야 됩니다.

이러한 Proxy의 구조를 이해하지 못한다면 [다음과 같은 상황](https://stackoverflow.com/questions/37726351/can-not-set-field-to-com-sun-proxy-proxy/37726709
)이 벌어질 수 있습니다.

``` java
@Controller
public class UserController{
  @Autowired
  private MemberService memberService; // <- Runtime Error 발생...
  ...
}

@Service
public class MemberService implements UserService{
  @Override
  public Map<String, Object> findUserId(Map<String, Object> params){
    ...isLogic
    return params;
  }
}
```

_인터페이스 타입이 아닌 클래스 타입으로 DI_

바로 다음과 같은 상황인데요.

MemberService 클래스는 인터페이스를 상속받고 있기 때문에 Spring은 JDK Dynamic Proxy 방식으로 Proxy Bean을 생성해줍니다. 다음과 같은 코드를 실행을 하면 Runtime Exception이 발생합니다. 여기서 Runtime 에러가 발생되는 부분은 바로 `@Autowired MemberService memberService` 코드인데요.

_MemberService memberService -> UserService memberService_

JDK Dynamic Proxy는 인터페이스 타입으로 DI를 받아줘야 하기 때문에, ~~@Autowired MemberService service~~ 가 아닌 `private UserService service`로 형식을 구성해줘야 합니다.

#### 3.2. 인터페이스 기준 그리고 내부적인 검증 코드

다른 관점에서 보자면 JDK Dynamic Proxy는 Proxy 패턴의 관점을 구현한 구현체라 할 수 있습니다.

이 Proxy 패턴은 접근제어의 목적으로 Proxy를 구성한다는 점도 중요하지만, 무엇보다 사용자의 요청이 기존의 타깃을 그대로 바라볼 수 있도록 타깃에 대한 위임코드를 Proxy 객체에 작성해줘야 합니다. 생성된 Proxy 객체의 타깃에 대한 위임코드는 바로 InvocationHandler에 작성해줘합니다.

![img](/md/img/aop/jdk-dynamic-proxy-and-cglib/jdk-dynamic-proxy2.png)
_타깃의 객체가 아닌 인터페이스를 기준으로 생성_

따라서 사용자의 요청이 최종적으로 생성된 Proxy의 메소드를 통해 호출할 때 내부적으로 invoke에 대한 검증과정이 이뤄집니다. 결과적으로 코드는 다음과 같습니다.

``` java
public Object invoke(Object proxy, Method proxyMethod, Object[] args) throws Throwable {
  Method targetMethod = null;
  // 주입된 타깃 객체에 대한 검증 코드
  if (!cachedMethodMap.containsKey(proxyMethod)) {
    targetMethod = target.getClass().getMethod(proxyMethod.getName(), proxyMethod.getParameterTypes());
    cachedMethodMap.put(proxyMethod, targetMethod);
  } else {
    targetMethod = cachedMethodMap.get(proxyMethod);
  }

  // 타깃의 메소드 실행
  Ojbect retVal = targetMethod.invoke(target, args);
  return retVal;
}
```

이 과정에서 검증과정이 이뤄지는 까닭은 다름아닌 Proxy가 기본적으로 인터페이스에 대한 Proxy만을 생성해주기 때문입니다. 따라서 개발자가 타깃에 대한 정보를 잘 못 주입할 경우를 대비하여 JDK Dynamic Proxy는 내부적으로 주입된 타깃에 대한 검증코드를 형성하고 있습니다.

그렇다면 CGLib은 무엇일까요?

### 4. CGLib(Code Generator Library)

CGLib은 Code Generator Library의 약자로, 클래스의 바이트코드를 조작하여 Proxy 객체를 생성해주는 라이브러리입니다.

Spring은 CGLib을 사용하여 인터페이스가 아닌 타깃의 클래스에 대해서도 Proxy를 생성해주고 있는데요. CGLib은 Enhancer라는 클래스를 통해 Proxy를 생성할 수 있습니다.

``` java
Enhancer enhancer = new Enhancer();
         enhancer.setSuperclass(MemberService.class); // 타깃 클래스
         enhancer.setCallback(MethodInterceptor);     // Handler
Object proxy = enhancer.create(); // Proxy 생성
```

>CGLib을 통해 Proxy를 구성하는 자세한 방법은 별도로 작성하지 않겠습니다. Proxy를 생성하는 방법은 [다음 링크(Baeldung-CGLib)](https://www.baeldung.com/cglib)를 참고해주세요.

다음과 같이 CGlib은 타깃의 클래스를 상속받아 다음 그림과 같이 Proxy를 생성해줍니다.

![img](/md/img/aop/jdk-dynamic-proxy-and-cglib/cglib1.png)
_타깃 클래스를 상속받아 Proxy 생성_

- Final methods can't be advised, as they can't be overridden.

이 과정에서 CGLib은 타깃 클래스에 포함된 모든 메소드를 재정의하여 Proxy를 생성해줍니다.

이 때문에 CGLib은 Final 메소드 또는 클래스에 대해 재정의를 할 수 없으므로 Proxy를 생성할 수 없다는 단점이 있지만, CGlib은 바이트 코드로 조작하여 Proxy를 생성해주기 때문에 성능에 대한 부분이 JDK Dynamic Proxy보다 좋습니다.

### 5.1. invoke의 차이, 성능의 차이

성능의 차이의 근본적인 이유는 CGLib은 타깃에 대한 정보를 제공 받기 때문입니다.

따라서 CGLib은 제공받은 타깃 클래스에 대한 바이트 코드를 조작하여 Proxy를 생성하기 때문에, Handler안에서 타깃의 메소드를 호출할 때 다음과 같은 코드가 형성됩니다.

``` java
public Object invoke(Object proxy, Method proxyMethod, Object[] args) throws Throwable {
  Method targetMethod = target.getClass().getMethod(proxyMethod.getName(), proxyMethod.getParameterTypes());
  Ojbect retVal = targetMethod.invoke(target, args);
  return retVal;
}
```

1. 메소드가 처음 호출되었을 때 동적으로 타깃의 클래스의 바이트 코드를 조작
2. 이후 호출시엔 조작된 바이트 코드를 재사용

CGLib은 성능이 좋긴하지만, Spring은 JDK Dynamic Proxy를 기반으로 Proxy를 생성해주고 있습니다.

>성능의 차이에 대한 부분은 Spring Boot의 블로그인 Baeldung에서도 사용된 공식적인 [AOP 성능 분석표](https://web.archive.org/web/20150520175004/https://docs.codehaus.org/display/AW/AOP+Benchmark)를 참고하세요.


### 4.2. 권장하지 않았던 CGLib

이러한 이유엔, 기존의 CGLiB은 3 가지의 한계가 존재했기 때문입니다.

![img](/md/img/aop/jdk-dynamic-proxy-and-cglib/cglib2.png)
_[Spring AOP defaults to using standard JDK dynamic proxies for AOP proxies. – Spring Reference](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-introduction-proxies)_

- net.sf.cglib.proxy.Enhancer 의존성 추가
- default 생성자
- 타깃의 생성자 두 번 호출

우선, 첫 번째는 Spring에서 기본적으로 지원하지 않는 방식이었기 때문에 별도로 **의존성을** **추가하여** 개발했습니다. 그다음으론 CGLib을 구현하기 위해선 반드시 파라미터가 없는 **default** **생성자가** 필요했고, 생성된 CGLib Proxy의 메소드를 호출하게 되면 **타깃의** **생성자가** **2번** 호출된다는 단점이 존재했습니다.

### 4.3. Spring Boot가 선택한 CGLib

하지만 어느 시점부터 Spring Boot에선 CGLib을 방식으로 Proxy를 생성해주고 있었는데요. 이러한 상황 때문에 [SpringBoot github-issues](https://github.com/spring-projects/spring-boot/issues/8434)에서 어느 Spring 개발자가 문제를 제기하게 되었습니다. 상황을 요약하자면 다음과 같습니다.

_인터페이스를 구현한 클래스가 왜 CGLib방식으로 Proxy를 생성하고 있냐_

이에 Spring Boot의 리더이자 총책임자인 **Phil**은 다음과 같이 답했습니다.

<img src="/md/img/aop/jdk-dynamic-proxy-and-cglib/cglib3.png" style="max-height:none;">

Phil의 답변을 토대로 보자면 CGLib의 문제가 되었던 부분들이 개선되어 안정화가 되었다고 밖에 생각할 수 없었습니다. 따라서 Spring 레퍼런스를 토대로 다음과 같은 상황들을 추척하여 결론을 내릴 수 있게 되었습니다.

<img src="/md/img/aop/jdk-dynamic-proxy-and-cglib/cglib4.png" style="max-height:none;">

우선 CGlib의 의존성을 추가하여 개발해야 된다는 점은 [Spring 3.2](https://docs-stage.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-api) 버전부터 CGLib을 Spring Core 패키지에 포함시켜 더이상 의존성을 추가하지 않아도 개발할 수 있게 되엇습니다.

그 다음 4 버전에선 [Objensis 라이브러리](http://objenesis.org/)의 도움을 받아 **[2]** default 생성자 없이도 Proxy를 생성할 수 있게 되었고, **[3]** 생성자가 2 번 호출되던 상황도 같이 개선이 되었습니다.

결과적으로 기존의 CGLib이 가지고 있던 대부분의 한계들이 개선이 되어, Spring에선 성능이 좋은 CGLib으로 Proxy를 생성하게 되었습니다.

### 마무리

여기까지 JDK Dynamic Proxy와 CGLib에 대해 알아보았습니다.

정리를 해보자면, JDK Dynamic Proxy는 인터페이스를 구현하여 Proxy를 생성해주고, Spring은 인터페이스가 아닌 클래스를 가지고 Proxy를 생성해주기 위해 CGLib 방식을 지원하고 있습니다.

CGLib은 클래스를 상속받아 Proxy를 생성해준다는 점, 마지막으로 CGLib이 가지고 있었던 3 가지 한계점이 모두 개선되어 Spring Boot에선 기본 Proxy 생성 방법으로 사용하고 있다는 부분을 인지하셨으면 좋겠습니다.

또한, JDK Dynamic Proxy는 Spring AOP의 AOP 기술의 근간이 되는 방식이기 때문에 Spring에서 사용되는 AOP의 기술들은 Proxy 메커니즘을 따르고 있습니다. 즉 CGLib이든 JDK Dynamic Proxy든 Proxy 메커니즘을 따른다는 점을 인지하셔야 됩니다.

---

### 참고

- [Spring 4.0.1.RELEASE - AOP Proxies](https://docs.spring.io/spring/docs/4.0.1.RELEASE/spring-framework-reference/htmlsingle/#aop-understanding-aop-proxies)
- [Spring Blog - CGLIB](https://www.baeldung.com/cglib)
- [Spring Blog - AspectJ](https://www.baeldung.com/aspectj)
- [Spring Blog - Spring AOP vs AspectJ](https://www.baeldung.com/spring-aop-vs-aspectj)
- [Objenesis](http://objenesis.org/)
- [ASM](https://asm.ow2.io/)
