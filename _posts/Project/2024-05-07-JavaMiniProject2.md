---
title: "Java Mini Project2"
excerpt: "[제로베이스] 결제 금액 캐시백 계산 프로그램"

categories:
  - Project
tags:
  - [Project]

permalink: /categories/JavaMiniProject2/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## 결제 금액 캐시백 계산 프로그램
수행 목적 : Scanner의 입력함수와 조건문을 통한 캐시백 계산 로직 작성<br>
간략 소개 : 직불카드로 결제를 하게되면 이에 대한 캐시백을 제공해 줍니다. <br>
주어진 캐시백 금액을 계산하는 프로그램을 작성해 보세요.<br>

주어진 캐시백 적립 조건에 맞게 캐시백 계산<br>
결제 금액을 입력하면, 이에 대한 캐시백 계산 후 결과 출력<br>

## 필수 준수사항

[캐시백 계산 조건]
- 결재 금액의 10%를 적립한다.
- 캐시백포인트 단위는 백원단위이다.(100원, 200원, 300원 등)
- 한건의 캐시백 포인트는 최대 300원을 넘을 수 없습니다.

```java
import java.util.InputMismatchException;
import java.util.Scanner;

public class Project2 {
    public static void main(String[] args) {
        System.out.println("[캐시백 계산]");

        System.out.printf("결제 금액을 입력해 주세요. (금액): ");
        Scanner scanner = new Scanner(System.in);
        int payment = 0;

        try {
            payment = scanner.nextInt();
            int cashback = getCashback(payment);
            System.out.println("결제 금액은 " + payment + "원 이고, 캐시백은 " + cashback + " 원 입니다.");
        } catch (InputMismatchException e) {
            System.out.println("잘못된 입력입니다. 숫자를 입력해주세요.");
        } finally {
            scanner.close();
        }
    }

    private static int getCashback(int payment) {
        int cashback;
        // 결재 금액의 10%를 적립한다.
        cashback = (int) (payment * (10.0 / 100.0));

        switch (cashback / 100) {
            case 1:
                cashback = 100;
                break;
            case 2:
                cashback = 200;
                break;
            case 3:
            default:
                cashback = 300;
                break;
        }
        return cashback;
    }
}
```
