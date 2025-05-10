# Chapter 12. 새로운 날짜와 시간 API

## 12.1 자바 8 이전의 날짜와 시간 API의 문제점

기존 자바의 날짜와 시간 API는 여러 문제점을 갖고 있다:

- `Date` 클래스는 특정 시점을 날짜가 아닌 밀리초 단위로 표현하며 직관적이지 못하다.
  ```java
  Date date = new Date(117, 8, 21);
  // 결과값 - Thu Sep 21 00:00:00 CET 2017
  ```
- `Date` 클래스는 자체적으로 시간대 정보를 알고 있지 않다.
- `Date`를 deprecated 시키고 등장한 `Calendar` 클래스 또한 쉽게 에러를 일으키는 설계 문제를 갖고 있다.
- `Date`와 `Calendar` 두 가지 클래스가 등장하면서 개발자들에게 혼란만 가중되었다.
- 날짜와 시간을 파싱하는데 등장한 `DateFormat`은 `Date`에만 지원되었으며, 스레드에 안전하지 못했다.
- `Date`와 `Calendar`는 모두 가변 클래스이므로 유지보수가 아주 어렵다.

## 12.2 LocalDate, LocalTime, LocalDateTime, Instant, Duration, Period 클래스

`java.time` 패키지는 새로운 날짜와 시간에 관련된 클래스를 제공한다.

### 12.2.1 LocalDate와 LocalTime의 사용

`LocalDate` 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다. `LocalDate` 객체는 어떤 시간대 정보도 포함하지 않는다. 정적 팩토리 메서드 `of`로 `LocalDate` 인스턴스를 만들 수 있다.

```java
LocalDate date = LocalDate.of(2017, 9, 11); // 2017-09-11
int year = date.getYear(); // 2017
Month month = date.getMonth(); // SEPTEMBER
int day = date.getDayOfMonth(); // 11
DayOfWeek dow = date.getDayOfWeek(); // MONDAY
int len = date.lengthOfMonth(); // 해당 월의 일수 (30)
boolean leap = date.isLeapYear(); // 윤년 여부 (false)
LocalDate now = LocalDate.now(); // 현재 날짜 정보
```

`LocalDate`가 제공하는 `get` 메서드에 `TemporalField`를 전달해서 정보를 얻는 방법도 있다. `TemporalField`는 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스다.

```java
// public int get(TemporalField field)
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);
```

`ChronoField`는 `TemporalField`의 구현체이며 `ChronoField`의 열거자 요소를 이용해서 원하는 정보를 쉽게 얻을 수 있다.

시간에 대한 정보는 `LocalTime` 클래스로 표현할 수 있다. `LocalTime`도 정적 메서드 `of`로 인스턴스를 만들 수 있다.

```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
int hour = time.getHour(); // 13
int minute = time.getMinute(); // 45
int second = time.getSecond(); // 20
```

`parse` 메서드를 통해 날짜와 시간 문자열로 `LocalDate`와 `LocalTime`의 인스턴스를 만들 수 있다.

```java
LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");
```

### 12.2.2 날짜와 시간 조합

`LocalDateTime`은 `LocalDate`와 `LocalTime`을 쌍으로 갖는 복합 클래스다. 날짜와 시간을 모두 표현할 수 있으며 정적 메서드 `of`로 인스턴스를 만들 수 있다.

```java
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

### 12.2.3 Instant 클래스: 기계의 날짜와 시간

`java.time.Instant` 클래스는 기계적인 관점에서 시간을 표현한다. `Instant` 클래스는 유닉스 에포크 시간(Unix epoch time)(1970년 1월 1일 0시 0분 0초 UTC)을 기준으로 특정 지점까지의 시간을 초로 표현한다. 팩토리 메서드 `ofEpochSecond`에 초를 넘겨주어 인스턴스를 생성할 수 있다. `Instant` 클래스는 나노초의 정밀도를 제공한다.

```java
Instant.ofEpochSecond(3); // 에포크 시간 이후 3초
Instant.ofEpochSecond(3, 0); // 에포크 시간 이후 3초
Instant.ofEpochSecond(2, 1_000_000_000); // 에포크 시간 이후 3초(2초 이후의 1억 나노초)
Instant.ofEpochSecond(4, -1_000_000_000); // 에포크 시간 이후 3초(4초 이전의 1억 나노초)
```

`Instant` 클래스도 사람이 확인할 수 있도록 시간을 표현해주는 정적 팩토리 메서드 `now`를 제공한다. 하지만 사람이 읽을 수 있는 시간정보를 얻으려면 `Instant`를 다른 클래스로 변환해야 한다.

### 12.2.4 Duration과 Period 정의

지금까지 살펴본 모든 클래스는 `Temporal` 인터페이스를 구현한다. `Temporal` 인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.

`Duration` 클래스를 사용하면 두 시간 객체 사이의 지속시간을 만들 수 있다. `Duration.between` 정적 팩토리 메서드를 사용하면 두 시간 객체 사이의 지속시간을 만들 수 있다.

```java
Duration d1 = Duration.between(time1, time2);
Duration d2 = Duration.between(dateTime1, dateTime2);
Duration d3 = Duration.between(instant1, instant2);
```

`Duration` 클래스는 초와 나노초로 시간 단위를 표현하므로 `between` 메서드에 `LocalDate`를 전달할 수 없다. 년, 월, 일로 시간을 표현할 때는 `Period` 클래스를 사용하자. `Period` 클래스의 팩토리 메서드 `between`을 이용하면 두 `LocalDate`의 차이를 확인할 수 있다.

```java
Period tenDays = Period.between(
    LocalDate.of(2017, 9, 11),
    LocalDate.of(2017, 9, 21)
);
```

지금까지 살펴본 모든 클래스는 불변이다. 불변 클래스는 함수형 프로그래밍, 스레드 안정성과 도메인 모델의 일관성을 유지하는데 좋다.

## 12.3 날짜 조정, 파싱, 포매팅

### 12.3.1 날짜 조정하기

날짜와 시간 API의 모든 클래스는 불변이다. `withAttribute` 메서드를 사용하면 일부 속성이 수정된 상태의 새로운 객체를 반환받을 수 있다.

```java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date3 = date1.withDayOfMonth(25); // 2017-09-25
```

`get`과 `with` 메서드로 `Temporal` 객체의 필드값을 읽거나 고칠 수 있으며 `Temporal` 객체가 지정된 필드를 지원하지 않으면 `UnsupportedTemporalTypeException`이 발생한다.

### 12.3.2 TemporalAdjusters 사용하기

간단한 날짜 기능이 아닌 더 복잡한 날짜 조정 기능이 필요할 때 `with` 메서드에 `TemporalAdjuster`를 전달하는 방법으로 문제를 해결할 수 있다. 날짜와 시간 API는 다양한 상황에서 사용할 수 있도록 다양한 `TemporalAdjuster`를 제공한다.

```java
import static java.time.temporal.TemporalAdjusters.*;

LocalDate date1 = LocalDate.of(2014, 3, 18); // 2014-03-18 (화)
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); // 2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth()); // 2014-03-31
```

다음은 자주 사용되는 `TemporalAdjuster` 메서드들이다:

| 메서드 | 설명 |
|-------|------|
| dayOfWeekInMonth | 서수 요일에 해당하는 날짜를 반환하는 TemporalAdjuster를 반환함 |
| firstDayOfMonth | 현재 달의 첫 번째 날짜를 반환 |
| firstDayOfNextMonth | 다음 달의 첫 번째 날짜를 반환 |
| firstDayOfNextYear | 내년의 첫 번째 날짜를 반환 |
| firstInMonth | 올해의 첫 번째 날짜를 반환 |
| lastDayOfMonth | 현재 달의 마지막 날짜를 반환 |
| lastDayOfNextMonth | 다음 달의 마지막 날짜를 반환 |
| lastDayOfNextYear | 내년의 마지막 날짜를 반환 |
| lastDayOfYear | 올해의 마지막 날짜를 반환 |
| lastInMonth | 현재 달의 마지막 요일에 해당하는 날짜를 반환 |
| next/previous | 현재 달에서 현재 날짜 이후로 지정한 요일이 처음으로 나타나는 날짜를 반환하는 TemporalAdjuster를 반환함 |
| nextOrSame | 현재 날짜 이후로 지정한 요일이 처음으로 나타나는 날짜를 반환하는 TemporalAdjuster를 반환함 |
| previousOrSame | 현재 날짜 이후로 지정한 요일이 이전으로 나타나는 날짜를 반환하는 TemporalAdjuster를 반환함 |

필요한 기능이 존재하지 않으면 커스텀 `TemporalAdjuster`를 구현하여 사용할 수 있다.

```java
@FunctionalInterface
public interface TemporalAdjuster {
    Temporal adjustInto(Temporal temporal);
}
```

### 12.3.3 날짜와 시간 객체 출력과 파싱

날짜와 시간 관련 작업에서 포매팅과 파싱은 필수적이다. `java.time.format` 패키지가 이를 지원한다. 가장 중요하게 알아야 할 클래스는 `DateTimeFormatter`다. 정적 팩토리 메서드와 상수를 이용해서 손쉽게 포매터를 만들 수 있다.

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18
```

또한 특정 패턴으로 포매터를 만들 수 있다:

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date = LocalDate.of(2014, 3, 18);
String formattedDate = date.format(formatter); // 18/03/2014
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

`LocalDate`의 `format` 메서드는 요청 형식의 패턴에 해당하는 문자열을 생성한다. 그리고 정적 메서드 `parse`는 같은 포매터를 적용해서 생성된 문자열을 파싱함으로써 다시 날짜를 생성한다.

또한, `DateTimeFormatterBuilder` 클래스를 이용하면 원하는 포매터를 직접 만들 수 있다. 이 클래스로 대소문자를 구분하는 파싱, 관대한 규칙을 적용하는 파싱, 패딩, 포매터의 선택사항 등을 활용할 수 있다.

## 12.4 다양한 시간대와 캘린더 활용 방법

새로운 날짜와 시간 API의 큰 편리함 중 하나는 시간대를 간단하게 처리할 수 있다는 점이다. 기존의 `java.util.TimeZone`을 대체할 수 있는 `java.time.ZoneId` 클래스가 새롭게 등장했다. `ZoneId`를 이용하면 서머타임 같은 복잡한 사항이 자동으로 처리된다. 또한 `ZoneId`는 불변 클래스다.

### 12.4.1 시간대 사용하기

표준 시간이 같은 지역을 묶어서 시간대(time zone) 규칙 집합을 정의한다. `ZoneRules` 클래스에는 약 40개 정도의 시간대가 있다. `ZoneId`의 `getRules()`를 이용해서 해당 시간대의 규정을 획득할 수 있다.

```java
ZoneId romeZone = ZoneId.of("Europe/Rome");
```

지역 ID는 '{지역}/{도시}' 형식으로 이루어 진다. 지역집합 정보는 IANA Time Zone Database에서 제공하는 정보를 사용한다. `getDefault()` 메서드를 이용하면 기존의 `TimeZone` 객체를 `ZoneId` 객체로 변환할 수 있다.

```java
ZoneId zoneId = TimeZone.getDefault().toZoneId();
```

`ZoneId`는 `LocalDate`, `LocalTime`, `LocalDateTime`과 같이 `ZonedDateTime` 인스턴스로 변환할 수 있다.

```java
LocalDate date = LocalDate.of(2014, 3, 18);
ZonedDateTime zdt = date.atStartOfDay(romeZone);
```
