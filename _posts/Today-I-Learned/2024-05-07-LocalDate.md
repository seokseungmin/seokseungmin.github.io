---
title: "LocalDate"
excerpt: "[TIL] LocalDate"

categories:
  - Today I Learned
tags:
  - [Today I Learned]

permalink: /categories/LocalDate/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## LocalDate

날짜만 사용할 사용

```java
LocalDate today = LocalDate.now();              // 현재 날짜 출력 (현재 날짜)
LocalDate date = LocalDate.of(현재연도, 현재월, 현재일);    // 특정 날짜 출력 (현재 날짜)

date.getYear();         // 연도 : 현재연도
date.getMonth();        // 월 : 현재월
date.getMonthValue();   // 월 : 현재월
date.getDateOfMonth();  // 일 : 현재일
date.getDayOfYear();    // 올해 며칠째 : 현재 날짜의 연중 일 수
date.getDayOfWeek();    // 요일 : 현재 요일
date.isLeapYear();      // 운년여부 : 현재 연도가 윤년인지 여부

// LocalDate -> String
date.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));

// String -> LocalDate
LocalDate.parse("형식에 맞는 문자열");

// LocalDate -> LocalDateTime
date.atTime(현재시, 현재분);
```

## LocalTime

시간만 사용할 사용

```java
LocalTime now = LocalTime.now();                // 현재 시간 출력 (현재 시간)
LocalTime time = LocalTime.of(현재시, 현재분, 현재초, 현재나노초);  // 특정 시간 출력 (현재 시간)

time.getHour();     // 시 : 현재시
time.getMinute();   // 분 : 현재분
time.getSecond();   // 초 : 현재초
time.getNano();     // 나노초 : 현재나노초
```

## LocalDateTime

날짜와 시간 모두 사용할 사용

```java
LocalDateTime now = LocalDateTime.now();                        // 현재 날짜와 시간 출력 (현재 날짜 및 시간)
LocalDateTime time = LocalDateTime.of(현재연도, 현재월, 현재일, 현재시, 현재분);    // 특정 날짜와 시간 출력 (현재 날짜 및 시간)
// LocalDate + LocalTime 조합
LocalDateTime time2 = LocalDateTime.of(LocalDate.now(), LocalTime.now());

// LocalDateTime -> String
time.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));

// String -> LocalDateTime
LocalDateTime.parse("형식에 맞는 문자열");
LocalDateTime.parse("형식에 맞는 문자열", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));

// LocalDateTime -> LocalDate
LocalDate.from(time);
```

## 날짜 차이 계산

Period 클래스

```java
LocalDate now = LocalDate.now();
LocalDate target = LocalDate.of(현재연도, 현재월, 현재일);

Period period = Period.between(now, target);

period.getYears();      // 현재와 타겟 날짜 간의 연 수
period.getMonths();     // 현재와 타겟 날짜 간의 월 수
period.getDays();       // 현재와 타겟 날짜 간의 일 수
```

## 시간 기준으로 계산하기 (ChronoUnit)

```java
LocalDate now = LocalDate.now();
LocalDate target = LocalDate.of(현재연도, 현재월, 현재일);

ChronoUnit.DAYS.between(now, target);   // 현재와 타겟 날짜 간의 일 수
```

## 시간 차이 계산

Duration 클래스

```java
LocalTime now = LocalTime.now();  // 현재 시간 출력 (현재 시간)
LocalTime target = LocalTime.of(현재시, 현재분, 현재초);  // 특정 시간 출력 (현재 시간)

Duration duration = Duration.between(now, target);

duration.getSeconds();      // 현재와 타겟 시간 간의 초 수
duration.getNano();         // 현재와 타겟 시간 간의 나노초 수
```

## YearMonth

년과 월만 사용할 경우 사용할 수 있는 클래스 (yyyy-MM)

```java
YearMonth date = YearMonth.of(현재연도, 현재월);     // 현재 연도와 월 출력 (현재 연도와 월)

date.getYear();             // 현재 연도
date.getMonth();            // 현재 월
date.getMonthValue());      // 현재 월
date.atEndOfMonth();        // 현재 월의 마지막 날짜
date.lengthOfMonth();       // 현재 월의 일 수
date.plusMonths(1);         // 현재 월에서 한 달 후
date.minusMonths(1);        // 현재 월에서 한 달 전
```

```java
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.LocalDateTime;
import java.time.Period;
import java.time.Duration;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.YearMonth;

public class Main {
    public static void main(String[] args) {
        // 현재 날짜 및 시간
        LocalDate today = LocalDate.now();
        LocalTime now = LocalTime.now();
        LocalDateTime currentDateTime = LocalDateTime.now();
        
        // 특정 날짜 및 시간
        LocalDate date = LocalDate.of(2024, 5, 7);
        LocalTime time = LocalTime.of(14, 30);
        LocalDateTime dateTime = LocalDateTime.of(2024, 5, 7, 14, 30);
        
        // LocalDate 메서드
        int year = today.getYear();                 // 연도 : 2024
        int month = today.getMonthValue();          // 월 : 5
        int dayOfMonth = today.getDayOfMonth();     // 일 : 7
        int dayOfYear = today.getDayOfYear();       // 올해 며칠째 : 128
        LocalDate tomorrow = today.plusDays(1);     // 내일 날짜
        
        // LocalTime 메서드
        int hour = now.getHour();                   // 시 : 14
        int minute = now.getMinute();               // 분 : 30
        int second = now.getSecond();               // 초 : 15
        
        // LocalDateTime 메서드
        LocalDateTime nextMonth = currentDateTime.plusMonths(1);  // 한 달 후
        
        // 날짜 형식 지정
        String formattedDate = today.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));  // 2024-05-07
        String formattedDateTime = currentDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")); // 2024-05-07 14:30:15
        
        // 날짜 차이 계산
        Period period = Period.between(today, date);  // 현재 날짜와 date 간의 기간
        long daysBetween = ChronoUnit.DAYS.between(today, date); // 현재 날짜와 date 간의 일 수
        
        // 시간 차이 계산
        Duration duration = Duration.between(now, time);  // 현재 시간과 time 간의 기간
    }
}
```

8버전에서는 새로운 날짜와 시간을 다루는 클래스인 LocalDate, LocalTime, LocalDateTime이 도입되었습니다(java.time).<br>
이제는 이러한 클래스를 사용하여 날짜와 시간을 다루는 것이 권장됩니다.<br>
이전에 사용되던 Date와 Calendar 클래스는 사용을 지양해야 합니다(java.util).<br>

**참고 블로그:**
[europani.github.io](https://europani.github.io/java/2021/10/05/021-localDateTime.html)
