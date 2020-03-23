---
layout: post
title: "Java 날짜 API"
tags: [Java, Java8, JDK8, Date, JodaTime, LocalDate]
categories: [Java, Date, LocalDate]
subtitle: "기본 날짜 API의 문제점과 Java8 날짜 API 정리"
feature-img: "md/img/thumbnail/java-logo.png"
thumbnail: "md/img/thumbnail/java-logo.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily

priority: 1.0
---

<!--more-->

# 기본 날짜 API의 문제점과 Java8 날짜 API 정리 

---

### 들어가기전

프로젝트를 진행하던 중에 여러 나라의 시간대를 맞추기 위해 기본 날짜 API를 사용한 유틸성 클래스가 존재했습니다. 최종적으로 기본 날짜 API를 Java8의 Time API로 개선할 예정이기 때문에 기본 날짜 API에 대한 학습의 시간이 필요했습니다. 본 포스팅에선 파악된 기본 날짜 API의 문제점을 기반으로 Java8 Time API 코드로 개선하려는 이유에 대해 공유하려 합니다.

### 기본 날짜 API (JDK 1.1 ~ 7)

Java8 이전엔 날짜와 관련된 다양한 클래스들을 제공하고 있습니다.

``` text
java
  ㄴ util
      ㄴ Date.class
      ㄴ Calendar.class
      ㄴ TimeZone.class
```

다음 클래스를 통해 특정 날짜를 생성하여 표현하지만, 일반적으로 특정 일자를 더하거나 빼거나 하는 날짜 계산이 필요한 경우가 많습니다. 기본 날짜 API에선 날짜 계산을 위해 java.util.Date, java.util.Calendar 클래스를 사용하게 됩니다. 하지만 기본 날짜 API의 문제들을 많은 개발 커뮤니티를 통해 쉽게 발견할 수 있습니다.

### 1. 기본 날짜 API의 문제점

Java에서 기본적으로 제공하고 있는 Date, Calendar 클래스의 대표적인 문제점은 다음과 같습니다.

- 햇갈리는 날짜 계산
    - int 상수 납용
- Date와 Calendar의 애매한 역할
- Not immutable
- 하위 클래스의 문제점
- 찾기 어려운 버그들

### 1.1. 햇갈리는 날짜 계산

다음 문제점들은 2020-01-01 날짜를 문자열로 반환하는 다음 예제 코드를 통해 설명하겠습니다.

``` java
@Test
@DisplayName("2020-01-01 날짜 생성하기")
void createNewYearByBasicJDK() {
    // [1] Calendar 객체 사용
    Calendar calendar = Calendar.getInstance();
    calendar.set(2020, 1, 1);
    
    // [2] getDate가 아닌 getTime으로 Date 타입 반환
    Date newYear = calendar.getTime();
    
    // [3] 테스트 결과는?
    assertAll("반환된 날짜는 2020-01-01 인가", 
            () -> assertEquals(2020, calendar.get(Calendar.YEAR)), 
            () -> assertEquals(1, calendar.get(Calendar.MONTH)), 
            () -> assertEquals(1, calendar.get(Calendar.DAY_OF_MONTH)), 
            () -> assertEquals("2020-01-01", formatting(newYear)));
}
```

1. Calendar 객체를 사용하여 날짜 지정
    - `calendar.set(2020, 1, 1)`
2. 지정된 날짜를 Date 타입으로 반환하기 위해 getTime 메서드를 사용
    - `calendar.getTime()`

다음 코드는 Calendar 객체를 사용하여 지정된 날짜를 Date 타입으로 반환하는 간단한 코드입니다. 하지만 다음 코드에서 Calendar 클래스의 문제점을 찾을 수 있습니다.   

바로 **getTime** 메서드입니다.

좋은 메서드는 이름만 보더라도 반환될 타입을 추론할 수 있습니다. 하지만 getTime 메서드 명은 자칫 java.sql.Time을 반환하는 것처럼 지어져 반환 타입을 예측하기에 어렵습니다. 이 부분은 개발자의 실수를 유발할 수 있는 부분입니다. ~~(getDate 으로 지었으면 어땠을까?)~~

무엇보다 테스트는 실패합니다.

``` java
expected: <2020-01-01> but was: <2020-02-01>
Comparison Failure: 
Expected :2020-01-01
Actual   :2020-02-01 <- 왜 2월인가?
```

### 1.1.1. 혼란스러운 월 계산 - 1월 = 0

이미 아시는 분들도 있겠지만, 테스트가 통과하기 위해선 월 지정을 1이 아닌 0으로 지정해줘야 합니다.

``` java
calendar.set(2020, 0, 1);
```

여기서 월의 시작이 0인 이유는 배열 계산을 쉽게 하기 위함입니다. 예를 들어 월을 문자열로 반환하기 위해 1월을 0으로 정해줌으로써 배열을 쉽게 관리할 수 있습니다.

``` java
String[] monthNames = new String[12]
String january = monthNames[0];
```
> [stackoverflow - Why is January month 0 in java calendar](https://stackoverflow.com/questions/344380/why-is-january-month-0-in-java-calendar)

이는 개발적인 관점으로 JDK 1.0 부터 등장한 Date 클래스에서 부터 1월을 0으로 표현했고, JDK 1.1 부터 포함된 Calendar 클래스 또한 이러한 관례를 답습하고 있다는걸 알 수 있습니다.

안타깝게도 처음 날짜 API를 접하는 개발자들에겐 1월을 0으로 당연하게 생각하기엔 어려움이 있습니다. 이러한 혼란을 없애기 위해 Calendar 클래스엔 월과 관련된 int 상수 필드를 제공해주고 있습니다.

``` java
public class Calendar {
	public final static int JANUARY = 0;
	public final static int FEBRUARY = 1;
	...
	public final static int DECEMBER = 11;
	public final static int UNDECIMBER = 12;
}
```

따라서 월 지정을 하는 코드에선 개발자의 실수를 방지하기 위해 Calendar의 월 상수를 사용하는 방식과 명시적으로 **월 - 1** 코드를 작성하는 경우가 존재합니다. 

``` java
calendar.set(2020, 1 -1, 1);
calendar.set(2020, Calendar.JANUARY, 1);
```

### 1.1.2. 남용되고 있는 int 상수 필드

Calendar 클래스의 월 상수들을 살펴보면 특이하게도 13 월을 의미하는 UNDECIMBER 상수가 존재합니다.

``` java
	public final static int UNDECIMBER = 12;
```

실제로 에티오피아에는 보편적으로 사용하는 그레고리력(Gregorian Calendar)이 아닌 콥트 달력(Coptic Calendar)과 율리우스력(Julian Calendar)을 사용하기 때문에 13월 1일이 존재합니다. Calendar.UNDECIMBER 월 상수 때문에 자칫 13월 1일 존재하는 것처럼 보이지만, Calendar.UNDECIMBER 상수는 음력 달력의 계산법을 지원하기 위한 특수한 월로써 Date 객체에선 13월을 표기하진 않습니다.

> 현대 달력은 보편적으로 [그레고리력(Gregorian Calendar)](https://en.wikipedia.org/wiki/Gregorian_calendar)를 사용하여 12월까지 존재하지만, [율리우스력(Julian Calendar)](https://en.wikipedia.org/wiki/Julian_calendar)이나 [콥트 달력(Copic Calendar)](https://en.wikipedia.org/wiki/Coptic_calendar)에는 13월이 존재합니다.

이외에도 Calendar 클래스에는 다양한 계산 상수의 타입을 int로 사용하고 있습니다. 이처럼 월과 계산 상수를 각각의 목적에 맞게 모듈화하지 않고, 같은 타입으로 사용했다는 점은 컴파일 시점에 에러를 찾기 어렵다는 단점이 있습니다.

``` java
// 억지지만...
// 다음 코드는 에러가 없고, 심지어 `2020-02-01`라는 동일한 값이 나온다.
calendar.set(2020, Calendar.FEBRUARY, Calendar.SUNDAY);
calendar.set(2020, Calendar.SUNDAY, Calendar.FEBRUARY);
```

이는 개발자 커뮤니티에서 Calendar 클래스를 사용하기 어렵다고 느끼는 이유이기도 합니다. 또한 계산 상수는 다양하고 API 문서를 보기전 까진 상세히 어떠한 기능인지 파악하기 어렵습니다. 예를 들어 월 + 1를 구하는 기능을 개발한다고 가정해봅시다.


``` java
public Date addMonth(Date date, int plus){
    Calendar calendar = Calendar.getInstance();
    calendar.setTime(date);
    
    // 다음 계산 상수들 중에 Month + 1 을 의미하는 상수는?
    [1] calendar.add(Calendar.MONTH, plus);
    [2] calendar.add(Calendar.WEEK_OF_MONTH, plus);
    [3] calendar.add(Calendar.DAY_OF_MONTH, plus);
    [4] calendar.add(Calendar.DAY_OF_WEEK_IN_MONTH, plus);
    return calendar.getTime();
}
```

다음 월 계산 상수엔 4가지의 선택지가 존재합니다. 다음 계산 상수에서 `월 + 1` 계산 상수는 `Calendar.MONTH`을 사용해야 합니다. 이처럼 날짜 계산을 하기 위한 다양한 날짜 계산 상수의 숙지가 필요합니다. 앞서 설명했던 것과 마찬가지로 컴파일 시점에서 에러가 나타나지 않습니다. 자칫 잘못된 계산 상수를 사용하여 코드 수정이 필요하다면, 어떤 부분에서 잘못되었는지 파악하기가 매우 어렵습니다.

### 1.2. Date와 Calendar의 애매한 역할

여기서 기본 날짜 API에 대해 사용하는 데 있어 익숙지 않은 분들이라면 의문점이 생기실 수 있습니다.

도대체 왜, 날짜 계산을 하기 위해 Date 클래스뿐만 아니라 Calendar 클래스를 같이 사용하고 있는지 의아해할 수 있습니다. 이는 Date와 Calendar의 클래스의 애매한 역할 때문입니다.

초기 JDK 1.0에선 Date 클래스가 날짜 연산을 지원하는 유일한 클래스였지만 JDK 1.1 이후부터 Calendar 클래스가 포함되면서 날짜간의 연산, 국제화 지원 등 날짜 계산을 Calendar 클래스에서 주로 담당하게 되었습니다. 따라서 본의 아니게 Date 클래스의 메서드들은 사용하지 않게(@Deprecated) 되었고, 앞서 예제 코드만 보더라도 특정 시간대의 날짜를 생성한다거나, 년/월/일 같은 날짜 단위의 계산은 Date 클래스만으로는 수행하기 어려워서 Calendar 클래스와 같이 사용해야 합니다.

하지만 Calendar 클래스의 생성 비용이 비싼 편일뿐더러, 날짜 계산을 위해 필연적으로 Calendar 객체를 생성해야 하므로 성능상으로도 좋지 않습니다. 이러한 불편함을 덜기 위해 [Apache Commons Lang 라이브러리](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)에 있는 DateUtils 클래스의 plusDays() 메서드나 plusMonth() 메서드 같은 메서드를 주로 활용하지만 DateUtils 클래스를 쓰더라도 중간 객체로 Calendar 인스턴스를 생성하는 것은 같기 때문에 성능상으론 좋지 않습니다.

### 1.3. 불편 클래스가 아니다. (Not immutable)

단순히 날짜 계산의 어려움은 API 보거나 [Apache Commons Lang 라이브러리](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)를 사용하면 됩니다. 하지만, 그보다 Date, Calendar 클래스의 가장 큰 문제는 불변이 아니라는 점입니다.

C#, Python 같은 언어에서는 날짜 클래스가 한번 생성된 이후에는 내부 속성을 바꿀 수 없도록 설계되었습니다. 이는 런타임 시에 객체가 공유되더라도 데이터가 변경될 위험이 없습니다. 하지만 Java의 Date, Calendar 클래스는 불변 객체가 아니므로 언제든 데이터가 변경될 위험이 따릅니다.

``` java
@Test
void alwayChangeBecauseNotImmutable() {
    Calendar calendar = Calendar.getInstance();
    //[1]
    calendar.set(Calendar.MONTH, Calendar.JANUARY);
    
    //[2]
    calendar.set(Calendar.MONTH, Calendar.FEBRUARY);
}
```

다음 코드를 보면 Calendar에 날짜를 지정하게 되면 같은 인스턴스 공유하기 때문에 최종적으로 마지막에 변경된 날짜가 저장하게 됩니다. 이와 마찬가지로 Date 클래스도 불변 클래스가 아닐뿐더러 언제든 데이터를 바꿀 수 있는 setter 메서드가 존재합니다.

> Date 클래스는 불변 객체여야 했다. - Joshua Bloch (Effective Java 저자)

이러한 단점들로 인해 Java 진영에선 안전하게 기본 날짜 타입을 사용하기 위해 다양한 시도가 있었습니다. 그중에서도 아래 코드처럼 객체를 복사해서 반환하는 기법을 주로 사용했습니다.

``` java
@Getter
public class BasicVO {
    private final Date startDate;
    private final Date endDate;
    
    public BasicVO(Date startDate, Date endDate){
        this.startDate = startDate;
        this.endDate = endDate;
    }
}
```

### 1.4. 하위 클래스의 문제점
### 1.4.1. Naming

이처럼 문제가 많은 java.util.Date 클래스를 상속한 하위 클래스에도 문제가 존재합니다. 우선 java.sql.Date 클래스는 상위 클래스인 java.util.Date 클래스와 이름이 같습니다. 

``` java
package java.sql;
public class Date extends java.util.Date { ... }

package java.util;
public class Date 
    implements java.io.Serializable, Cloneable, Comparable<Date> { ... }   
```

이를 두고 Java 플랫폼 설계자는 클래스 이름을 지으면서 깜빡 존 듯하다는 조롱까지 나왔습니다. java.sql.Date 클래스는 Comparable 인터페이스에 대한 정의를 클래스 선언에서 하지 않았기 때문에 Comparable과 관련된 Generics 선언을 복잡하게 만들었습니다.

### 1.4.2. 동등성 (equality)

이보다 더 문제는 java.sql.TimeStamp 클래스에 있습니다.

바로 이 클래스는 java.util.Date 클래스에 나노초(nanosecond) 필드를 더한 클래스인데 equals() 선언의 대칭성을 어겼습니다.

``` java
@Test
void shouldNotBeUsedEqualsWhenComapareToChildObject() {
    Date date = new Date();
    Timestamp timestamp = new Timestamp(date.getTime());
    
    assertTrue(date.equals(timestamp)); // true
    assertFalse(timestamp.equals(date)); // false
}
```

따라서 a.equals(b)가 true라도 b.equals(a)는 false인 경우가 생길 수 있습니다.

### 1.5. 찾기 어려운 버그들

마지막으로 찾기 어려운 버그들이 존재합니다.

``` java
@Test
void shouldSetGmtWhenWrongTimeZoneId(){
	TimeZone zone = TimeZone.getTimeZone("Seoul/Asia");
	assertThat(zone.getID()).isEqualTo("GMT");
}
```

다음 코드는 시간대의 ID를 'Asia/Seoul'를 실수로 'Seoul/Asia'로 잘못 지정한 코드지만 `Timezone.getTimeZone` 메서드를 자세히 살펴보면 타임존의 ID를 찾을 수 없을 경우엔 `GMT`를 반환하게 됩니다.

``` java
private static TimeZone getTimeZone(String ID, boolean fallback) {
    TimeZone tz = ZoneInfo.getTimeZone(ID);
    if (tz == null) {
        tz = parseCustomTimeZone(ID);
        if (tz == null && fallback) {
            tz = new ZoneInfo(GMT_ID, 0);
            //  ^-- [1] ID에 해당되는 타임존을 찾을 수 없으면 GMT 반환
        }
    }
    return tz;
}
```

이러한 특성 때문에 컴파일 시점뿐만 아니라 런타임 시점에서도 오류가 발생하지 않고 마치 `GMT` 시간대가 지정된 것처럼 테스트를 통과하게 됩니다. 결과적으로 찾기 어려운 버그가 생길 위험이 있습니다.

### 2. 새로운 Java의 날짜 API
### java.time.*

이 외에도 본 포스팅에선 다루지 않은 `java.text.SimpleDateFormat`의 Thread-safe 하지 않는 이슈가 존재합니다. 이처럼 Java의 기본 날짜 API는 설계부터가 잘못되어 사용하기에 불편하고 개발자의 실수를 유발할 수 있는 클래스로 평가하고 있으며, 심지어 구현조차 잘못된 클래스로 악평이 자자합니다. 이러한 부분들은 Date 클래스를 보면 대부분 메서드가 @Deprecated 되어 있다는 걸 알 수 있습니다. Java 진영에선 나쁘지만 삭제되지 않을 클래스에 대해 @Deprecated 주석을 사용하는 것으로 유명한데, 이를 근거로 Java에서도 본인들의 실수를 어느 정도 인정하는듯한 모습이라 할 수 있습니다.

이처럼 Java의 기본 날짜 API는 문서를 열심히 보기 전까지는 제대로 사용하기 어렵습니다. 따라서 JDK8 이전에는 대부분 [Joda-Time 라이브러리](https://www.joda.org/joda-time/)을 많이 사용했습니다.

- org.joda.time.LocalDate
- org.joda.time.LocalDateTime
- org.joda.time.LocalTime

> Joda-Time 라이브러리는 Java8 이전에서 사용하는 표준 날짜 및 시간 라이브러리라 할 수 있다. 또한, Hibernate를 지원하는 [joda-time-hibernate](https://www.joda.org/joda-time-hibernate/) 라이브러리도 존재한다.

### 2.1. 새로운 날짜의 표준 명세 - JSR-310

2014년에 최종 배포된 JDK8 부터 JSR-310이라는 새로운 표준 명세를 정의함과 동시에 Joda-Time의 유용한 기능들을 java.time 패키지에 포함하여 배포하게 되었습니다. 스프링 프레임워크는 4.0 버전부터 JSR-310을 기본으로 지원하고 있습니다.

``` text
java
  ㄴ time
      ㄴ LocalDate.class
      ㄴ LocalTime.class
      ㄴ LocalDateTime.class
      ㄴ ZonedDateTime.class
      ㄴ ZoneId.class
```

> - [Joda-Time - ISO8601 Java calendar system](https://www.joda.org/joda-time/cal_iso.html)
> - [UTC 와 표기법, 그리고 ISO 8601, RFC 3339 표준](https://ohgyun.com/416)

새로운 날짜 API, 일명 Time API는 앞에서 설명한 Joda-Time에 가장 많은 영향을 받았으며 그 밖에 Time and Money 라이브러리나 ICU 등 여러 오픈소스 라이브러리를 참고하여 개발이 되었습니다.

'java.time.*' 패키지로 시작하지만, 거의 Joda-Time과 유사한 모습을 보여 주고 있습니다. JDK8의 Time API 특징엔 다음과 같습니다.

- 기존 클래스를 대체한 새로운 Time 클래스들
    - TimeZone -> ZoneId
    - Date -> LocalDate, LocalDateTime
    - DateTime -> ZonedDateTime
- 문제점 개선
    - 찾기 어려운 버그 개선
    - 월 상수 모듈화
    - Immutable
- 직관적인 API 메서드 명명
- 높아진 시간 정밀성

### 2.2. 문제점 개선

우선, DateTime 클래스대신 ZoneDateTime 클래스가 사용됩니다.

``` java
@Test
void shouldOccurZoneRulesExceptionWhenUnknownTimeZone() {
    ZoneId zone = ZoneId.of("Seoul/Asia");
    ZonedDateTime zonedDateTime = ZonedDateTime.now(zone);
    // -> java.time.zone.ZoneRulesException: Unknown time-zone ID: Seoul/Asia
}
```

앞서 설명했던것 처럼 `Seoul/Asia`라는 잘못된 시간대 주입합니다. 이전에 TimeZone 클래스를 사용할 경우엔 잘못된 시간대 주입에도 불구하고 `GMT` 시간대를 반환되는 문제가 존재했습니다. 반면 JDK8 부터는 찾을 수 없는 시간대가 주입될 경우 `java.time.zone.ZoneRulesException` 예외가 반환되도록 수정되었습니다.

``` java
package java.time;
public enum Month implements TemporalAccessor, TemporalAdjuster {
    /**
     * The singleton instance for the month of January with 31 days.
     * This has the numeric value of {@code 1}.
     */
    JANUARY,
    
    ...
    
    /**
     * The singleton instance for the month of December with 31 days.
     * This has the numeric value of {@code 12}.
     */
    DECEMBER;
}
```

또한, Calendar 클래스의 상수의 문제점이 다음과 같이 Month라는 Enum 클래스로 제공하며, 0부터 시작되는 것이 아닌 1부터 시작되도록 변경하여 날짜 지정/계산을 할 때 잘못 지정하거나 혼동할 여지가 없도록 설계되었습니다.

``` java
// 2020-01-01
Calendar calendar = Calendar.getInstance();
calendar.set(2020, 1 -1, 1);
Date basicNewYear = calendar.getTime();

LocalDate newYear = LocalDate.of(2020, 1, 1);
```

따라서 기존에 월-1을 하는 코드도 필요가 없을뿐더러 잘못된 월 지정에 대해 DateTimeException을 반환하도록 설계되었습니다.

무엇보다 언제나 데이터의 변경 위험이 있었던 부분은 불변 클래스로 정의하여 한번 생성된 날짜 데이터에 대해 변경되지 못하도록 구성하였습니다.

``` java
package java.time;
public final class LocalDate
    implements Temporal, TemporalAdjuster, ChronoLocalDate, Serializable {
    ...
}
```

### 2.2. 누구나 쉽게 알아볼 수 있는 메서드
### 정적 팩토리 메서드 패턴 (static factory method) 적극 활용

Time API엔 인스턴스 생성을 할 때 생성자 방식 대신, 정적 메서드 패턴을 활용하여 인스턴스를 생성하게 됩니다.

``` java
LocalDate.of(...);
LocalDate.from(...);
```

정적 팩토리 메서드는 가독성 있는 이름을 따로 붙일 수 있고, 생성자와는 달리 한번 생성된 객체를 재활용할 수 있습니다.

### 2.3. 높아진 시간 정밀성

그 밖에도 기존 Calendar, Date, Joda-Time의 시간 클래스가 밀리초(millisecond) 단위의 정밀성을 가졌다면, JSR-310의 클래스는 나노초까지 다룰 수 있습니다. 따라서 시계의 개념도 도입되어서 현재 시간과 관련된 기능을 테스트할 때도 유용합니다.

> java.time.Clock 클래스의 하위 클래스로 SystemClock, FixedClock 등이 제공된다.

이미 Spring 프레임워크 4.0 버전 부터는 JSR-310을 기본적으로 지원하고 있습니다. ZoneDateTime 등의 타입이 Controller의 메서드 파라미터로 선언되면 사용자가 입력한 문자열을 날짜 객체로 변환해줍니다.

``` java
@InitBinder(value = {"search"})
public void dateBinder(WebDataBinder dataBinder, HttpServletRequest request) {
    SimpleDateFormat dateFormat = new SimpleDateFormat(DateUtil.getDatePattern());
    dateFormat.setLenient(false);
    dataBinder.registerCustomEditor(Date.class, new CustomDateFormatEditor(dateFormat, true, request.getSession()));
}
```

따라서 날짜 문자열 타입을 Date 타입으로 바인딩하기 위해 존재했던 코드가 필요 없습니다.

### 결론
### 더 오래되고 아직까지도 존재하는 Date 클래스가 더 좋은게 아닌가?
### 묻거나 따지지말고 새로운 프로젝트에선 Java8 날짜 API를 도입하자.

Java SE 8부터는 사용자에게 java.time (JSR-310) 으로 마이그레이션해야합니다. - [Oracle 발췌](https://www.oracle.com/technical-resources/articles/java/jf14-date-time.html)

### 참고

- [Baeldung - Java Date to LocalDate and LocalDateTime](https://www.baeldung.com/java-date-to-localdate-and-localdatetime)
- [Oracle - How and When To Deprecate APIs](https://docs.oracle.com/javase/7/docs/technotes/guides/javadoc/deprecation/deprecation.html)
- [Oracle - Java-Date-Time package](https://docs.oracle.com/javase/tutorial/datetime/index.html)
- [stackoverflow - What does Calendar.UNDECIMBER do](https://stackoverflow.com/questions/44716517/what-does-calendar-undecimber-do)
- [stackoverflow - How to make a class immutable in Java](https://stackoverflow.com/questions/31846965/how-to-make-a-class-immutable-in-java-with-date-field-in-it)
- [stackoverflow - Whats the difference between Java Time LocalDateTime](https://stackoverflow.com/questions/53643505/whats-the-difference-between-java-time-localdatetime-java-util-date-and-java-u)
- [stackoverflow - Should i use java util date or switch to Java Time LocalDate](https://stackoverflow.com/questions/28730136/should-i-use-java-util-date-or-switch-to-java-time-localdate)
- https://hamait.tistory.com/205
- https://d2.naver.com/helloworld/645609