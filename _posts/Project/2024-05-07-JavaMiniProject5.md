---
title: "Java Mini Project5"
excerpt: "[제로베이스] 달력 출력 프로그램"

categories:
  - Project
tags:
  - [Project]

permalink: /categories/JavaMiniProject5/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## 달력 출력 프로그램
수행 목적 : Scanner의 입력함수와 조건문 및 반복문을 통한 달력 계산 로직 작성<br>
간략 소개 : 달력은 일반적인 전산시스템에서 많이 사용하는 컴포넌트입니다. <br>
입력받은 년도와 월을 통해 달력을 출력하는 프로그램을 작성해 보세요.<br>

## 필수 준수사항
입력받은 년도와 월을 통한 달력 생성<br>
입력값은 년도, 월을 입력<br>
날짜는 LocalDate클래스를 이용(Calendar와 Date클래스도 이용 가능)<br>
출력은 입력한 달을 기준으로 이전달, 입력달, 현재달 출력(3달 출력)<br>

```java
import java.time.LocalDate;
import java.util.Scanner;

public class Project5 {

    public static void makeCalender(int year, int month) {

        int date = 1;       // 일

        // 해당 년도 월의 첫 날을 가져옴
        LocalDate firstDayOfMonth = LocalDate.of(year, month, 1);

        // 해당 월이 시작하는 요일을 가져옴 (1: 월요일, ..., 7: 일요일)
        int startDayOfWeek = firstDayOfMonth.getDayOfWeek().getValue();

        // 해당 월의 마지막 날짜를 가져옴
        int lastDayOfMonth = firstDayOfMonth.lengthOfMonth();

        // 년도와 월 출력
        System.out.println("[" + year + "년 " + String.format("%02d", month) + "월]");
        System.out.println("일\t월\t화\t수\t목\t금\t토\t");

        // 1일 전까지의 공백 생성
        for (int i = 0; i < startDayOfWeek % 7; i++) {
            System.out.print("\t");
        }

        // 달력 일 출력
        for (int i = 1; i <= lastDayOfMonth; i++) {
            System.out.printf("%02d\t", date++);
            startDayOfWeek++;

            if (startDayOfWeek % 7 == 0) {                // 다음 주로 줄 바꿈
                System.out.println();
            }
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int year;
        int month;

        Scanner scanner = new Scanner(System.in);

        while (true) {
            System.out.println("[달력 출력 프로그램]");
            System.out.print("달력의 년도를 입력해 주세요. (yyyy) : ");
            year = scanner.nextInt();
            // 년도 음수 입력 방지
            if (year < 0) {
                System.out.println("년도 입력이 잘못되었습니다. 다시 입력해주세요.\n");
                continue;
            }

            System.out.print("달력의 월을 입력해 주세요. (mm) : ");
            month = scanner.nextInt();
            // 월 1~12 외 입력 방지
            if (month <= 0 || month > 12) {
                System.out.println("월 입력이 잘못되었습니다. 다시 입력해주세요.\n");
                continue;
            }
            break;
        }

        makeCalender(year, month - 1);
        makeCalender(year, month);
        makeCalender(year, month + 1);
    }
}
```
