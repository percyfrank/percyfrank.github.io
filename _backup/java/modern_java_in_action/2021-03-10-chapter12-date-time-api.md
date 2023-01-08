---
title: "[Modern Java in Action] 12장. 새로운 날짜와 시간 API"
excerpt: "과거의 Date와 Calendar 및 DateFormat 클래스는 가변 객체이기 때문에 스레드 안전하지 않았다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-03-10
last_modified_at: 2021-03-10
---

## 1. 과거의 날짜와 시간 API

과거의 Date와 Calendar 및 DateFormat 클래스는 사용할 때 다소 직관적이지 않아 불편했으며, 가변 객체이기 때문에 스레드 안전하지 않았다. 그 결과 Java 8 이전까지 개발자들은 서드파티 라이브러리를 사용하여 날짜와 시간 관련 기능들을 구현했다. Java 8 부터는 ``java.time`` 패키지를 통해 개선된 날짜와 시간 API를 제공한다.

<br>

## 2. LocalDate 및 LocalTime

> LocalDateExample.java

```java
LocalDate date = LocalDate.of(2014, 3, 18);
LocalDate date2 = LocalDate.now();

int year = date.getYear(); // 2014
Month month = date.getMonth(); // MARCH
int day = date.getDayOfMonth(); // 18

DayOfWeek dow = date.getDayOfWeek(); // TUESDAY
int len = date.lengthOfMonth(); // 31 (days in March)
boolean leap = date.isLeapYear(); // false (not a leap year)
```

* ``java.time`` 패키지에서 제공하는 날짜 및 시간 관련 객체들은 기본적으로 불변이다.

> LocalDateExample.java

```java
int y = date.get(ChronoField.YEAR);
int m = date.get(ChronoField.MONTH_OF_YEAR);
int d = date.get(ChronoField.DAY_OF_MONTH);
```

* 인자로 TemporalField 인터페이스를 전달해 원하는 날짜 관련 값들을 가져올 수 있다.
  * ChronoField 열거 클래스에 미리 정의되어 있다.

> LocalTimeExample.java

```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
int hour = time.getHour(); // 13
int minute = time.getMinute(); // 45
int second = time.getSecond(); // 20
```

> ParseExample.java

```java
LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");
```

* 문자열을 입력받아 원하는 객체로 파싱할 수 있으며, 추후 서술할 DateTimeFormatter를 사용하면 더 유연하게 파싱할 수 있다.

<br>

## 3. LocalDateTime

LocalDateTime은 LocalDate와 LocalTime을 조합한 클래스이다. 타임 존이 없는 날짜와 시간을 표현할 수 있다.

> LocalDateTime.java

```java
LocalDateTime dt1 = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45, 20); // 2014-03-18T13:45
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

* LocalDate나 LocalTime 객체에서 ``atTime()`` 혹은 ``atDate()`` 등의 메서드를 통해 LocalDateTime으로도 변경이 가능하다.
  * LocalDateTime에서 ``toLocalDate()`` 혹은 ``toLocalTime()`` 등을 통해 반대로 변환할 수 있다.

<br>

## 4. Instant

LocalDateTime이 사람을 위한 날짜 및 시간 API라면, Instant는 기계를 위한 API다. Instant는 시간을 계속되는 타임라인에 위치한 하나의 점으로 표현하는데, Unix epoch 시간으로부터 몇 초가 지났는지를 표현한다.

> InstantExample.java

```java
Instant instant = Instant.ofEpochSecond(3);
Instant instant2 = Instant.ofEpochSecond(3, 1_000_000_000); //1 second after 2 seconds
Instant instant3 = Instant.ofEpochSecond(3, -1_000_000_000); //1 second before 3 seconds
Instant now = Instant.now();
```

* 두 번째 인자를 통해 나노초 만큼의 정밀도를 지원해준다.
* Instant는 오로지 초와 나노초로 구성된 정보만을 담고 있어서 인간에게 유용한 정보를 주지 못한다.
  * Instant 객체에 ``get(ChronoField.DAY_OF_MONTH)``를 호출하면 예외가 발생한다.

<br>

## 5. Duration 및 Period

Duration 및 Period는 Temporal 인터페이스를 구현한 클래스로서, 시간의 특정한 부분을 모델링하는 객체의 값을 읽거나 조작하는 행위를 정의한다.

> DurationExample.java

```java
Duration d1 = Duration.between(LocalTime.of(13, 45, 10), time);
Duration d2 = Duration.between(instant, now);
Duration d3 = Duration.between(time, LocalDateTime.now());
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);
```

> PeriodExample.java

```java
Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1);
Period tenDays2 = Period.between(LocalDate.of(2017, 9, 11), LocalDate.of(2017, 9, 21));
```

* 두 클래스는 비슷한 메서드들을 제공한다.

<br>

## 6. 날짜 조정과 파싱 및 포매팅

> ManipulateExample.java

```java
LocalDate date1 = LocalDate.of(2018, 9, 21);
LocalDate date2 = date1.withYear(2011);
LocalDate date3 = date2.withDayOfMonth(25);
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2);

LocalDate date5 = date1.plusWeeks(1);
LocalDate date6 = date5.minusYears(6);
LocalDate date7 = date6.plus(6, ChronoUnit.MONTHS);
```

* ChronoUnit 열거형은 TemporalUnit 인터페이스 구현체를 제공한다.
  * Temporal 인터페이스는 TemporalUnit만큼 시간(날짜)를 움직인다.

### 6.1. TemporalAdjusters

Date 관련 객체에 존재하는 ``with()`` 메서드에 TemporalAdjusters 인터페이스 구현체를 전달하면 더욱 유연하게 날짜를 조작할 수 있다. TemporalAdjusters 인터페이스는 특정한 날짜를 연산하는데 필요한 조작 방법을 커스텀하게 정의한다.

> TemporalAdjustersExample.java

```java
LocalDate date = LocalDate.of(2014, 3, 18);
LocalDate date2 = date.with(nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date.with(lastDayOfMonth());
```

* TemporalAdjusters에 미리 정의되어 있는 다양한 메서드들로 특정 조건의 날짜를 간편하게 조작할 수 있다.

> TemporalAdjustersExample.java

```java
TemporalAdjuster nextWorkingDay = temporal -> {
    DayOfWeek dow = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK));
    int dayToAdd = 1;
    if (dow == DayOfWeek.FRIDAY) {
        dayToAdd = 3;
    }
    if (dow == DayOfWeek.SATURDAY) {
        dayToAdd = 2;
    }
    return temporal.plus(dayToAdd, ChronoUnit.DAYS);
};

LocalDate nextWorkingDay = beforeWorkingDay.with(nextWorkingDay);
```

* 별도의 TemporalAdjusters 인터페이스 구현체나 람다 표현식을 넘겨주어 원하는 날짜를 조작할 수도 있다.

### 6.2. 포매팅

> ParsingExample.java

```java
LocalDate date = LocalDate.of(2014, 3, 19);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE);

LocalDate date2 = LocalDate.parse("20140318", DateTimeFormatter.ISO_LOCAL_DATE);

DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("dd/MM/YY", Locale.ITALIAN);
String formattedDate = date.format(dateTimeFormatter);
```

* DateTimeFormatter는 과거 DateFormat 클래스와 달리 스레드 안전하다.
* 이미 정의된 상수 혹은 정적 팩토리 메서드를 통해 원하는 포맷 형식을 사용한다.

> ParsingExample.java

```java
DateTimeFormatter complexFormatter = new DateTimeFormatterBuilder()
     .appendText(ChronoField.DAY_OF_MONTH)
     .appendLiteral(". ")
     .appendText(ChronoField.MONTH_OF_YEAR)
     .appendLiteral(" ")
     .appendText(ChronoField.YEAR)
     .parseCaseInsensitive()
     .toFormatter(Locale.ITALIAN);
```

* 원한다면 빌더를 통해 자신만의 DateTimeFormatter를 유연하게 설정할 수 있다.

<br>

## 7. 다양한 시간대와 캘린더 활용 방법

ZoneId 클래스는 기존의 TimeZone 클래스를 대체하여 일광 절약 시간제(DST) 등 다양한 시간대를 제공해준다. 해당 클래스 또한 불변이다.

> TimeZoneExample.java

```java
ZoneId romeZone = ZoneId.of("Europe/Rome");
ZoneId defaultZone = TimeZone.getDefault().toZoneId();

LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
ZoneDateTime zdt1 = date.atStartDay(romeZone);
ZoneDateTime zdt2 = LocalDateTime.now().atZone(defaultZone);

Instant instant = LocalDateTime.now().toInstant(defaultZone);
LocalDateTime timeFromInstant = LocalDateTime.ofInstant(instant, romeZone);
```

* ZoneDateTime은 LocalDateTime과 ZoneId를 포괄하고 있다.
* Instant와 LodalDateTime간의 변환을 위한 메서드에도 ZoneId를 활용할 수 있다.

> OffsetDateTimeExample.java

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
OffsetDateTime offsetDateTime = OffsetDateTime.of(date, newYorkOffset);
```

* UTC/Greenwich의 고정 오프셋을 사용하여 시간대를 표현할 수 있다.
* ZoneOffset은 ZoneId의 하위 클래스로서 런던의 Greenwich와 특정 시간의 차이를 표현한다.
* 일광 절약 시간제 등을 표현할 수 없기 때문에 대부분의 경우 사용이 권장되지 않는다.

Java 8은 추가적인 달력 체계를 제공한다. JapaneseDate, ThaiBuddhistDate, HijrahDate 등. 이들은 ChronoLocalDate 인터페이스를 구현하며 임의의 연대표를 표현한다.

> CalendarExample.java

```java
LocalDate date = LocalDate.now();
JapaneseDate japaneseDate = JapaneseDate.from(date);

Chronology japaneseChronology = Chronology.ofLocale(Locale.JAPAN);
ChronoLocalDate now = japaneseChronology.dateNow();
```

* ``from()`` 정적 팩토리 메서드에 LocalDate를 받아 생성한다.
* 혹은 명시적으로 특정 지역의 달력 체계를 생성하고 해당 Local에 대한 ChronoLocalDate 객체를 얻을 수 있다.
* 그러나 일반적인 경우 ChronoLocalDate 대신 LocalDate 사용이 권장된다.
  * 대부분의 어플리케이션은 다양한 달력 체계를 제공하지 않고 표준 달력 체계를 사용하기 때문에 개발자가 혼란을 겪을 수 있다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
