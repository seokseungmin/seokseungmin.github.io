---
title: "Java Mini Project8"
excerpt: "[제로베이스] 연소득 과세금액 계산 프로그램"

categories:
  - Project
tags:
  - [Project]

permalink: /categories/JavaMiniProject8/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## 연소득 과세금액 계산 프로그램

수행 목적 : Scanner의 입력함수와 조건문 및 반복문 과 배열, 함수를 통한 과세 로직 작성. <br>
간략 소개 : 대한민국 헌법은 국민의 의무와 권리를 규정하고 있습니다. 이중 납세의 의무는 국민의 3대 의무중 하나입니다. <br>
모든 국민은 1년 동안 열심히 번 소득에 대해서 세금을 납부하여야 합니다.<br>
이런 소득에 대한 소득세율표가 있습니다. <br>
주어진 표를 기준으로 해서 소득에 대한세금을 구하는 프로그램을 작성해 보세요.<br>

## 필수 준수사항
### 설명)
1,000만원 소득인 경우는 과세표준이 1,200만원 이하 이기 때문에 세율을 6%로 계산한 결과인 60만원의 세금이 부과됨.<br>
1,500만원 소득의 경우는 과세표준 구간이 15%세율이기 때문에 15%로 계산하는 게 아니라 1,200만원까지는 6%의 세율로 계산하고 그 외만 15%로 계산해서 합계를 냄.<br> 
이때, 누진공제 금액을 이용할 수 있는데 1,500만원에 15%세율로 계산한 금액 225만원에 누직 공제 금액을 빼면 세금계산과 동일한 금액입니다.<br>

```java
import java.util.InputMismatchException;
import java.util.Scanner;

public class Project8 {
    public static void main(String[] args) {
        taxableAmountProgram();
    }

    private static void taxableAmountProgram() {
        System.out.println("[과세금액 계산 프로그램]");

        try {
            System.out.print("연소득을 입력해 주세요.:");
            Scanner scanner = new Scanner(System.in);
            int annual_income = scanner.nextInt();

            double taxValue = 0;
            double deductionValue = 0;
            int i = 0;
            double finalValue = 0;

            if (annual_income <= 12000000) {
                taxValue = annual_income * 0.06;
                deductionValue = annual_income * 0.06;
                finalValue = annual_income;
                i = 0;
            } else if (annual_income <= 46000000) {
                taxValue = (12000000 * 0.06) + (annual_income - 12000000) * 0.15;
                deductionValue = annual_income * 0.15 - 1080000;
                finalValue = (annual_income - 12000000);
                i = 1;
            } else if (annual_income <= 88000000) {
                taxValue = (12000000 * 0.06) + (34000000 * 0.15) + (annual_income - 46000000) * 0.24;
                deductionValue = annual_income * 0.24 - 5220000;
                finalValue = (annual_income - 46000000);
                i = 2;
            } else if (annual_income <= 150000000) {
                taxValue = (12000000 * 0.06) + (34000000 * 0.15) + (42000000 * 0.24) + (annual_income - 88000000) * 0.35;
                deductionValue = annual_income * 0.35 - 14900000;
                finalValue = (annual_income - 88000000);
                i = 3;
            } else if (annual_income <= 300000000) {
                taxValue = (12000000 * 0.06) + (34000000 * 0.15) + (42000000 * 0.24) + (62000000 * 0.35) + (annual_income - 150000000) * 0.38;
                deductionValue = annual_income * 0.38 - 19400000;
                finalValue = (annual_income - 150000000);
                i = 4;
            } else if (annual_income <= 500000000) {
                taxValue = (12000000 * 0.06) + (34000000 * 0.15) + (42000000 * 0.24) + (62000000 * 0.35) + (150000000 * 0.38) + (annual_income - 300000000) * 0.40;
                deductionValue = annual_income * 0.40 - 25400000;
                finalValue = (annual_income - 300000000);
                i = 5;
            } else if (annual_income <= 1000000000) {
                taxValue = (12000000 * 0.06) + (34000000 * 0.15) + (42000000 * 0.24) + (62000000 * 0.35) + (150000000 * 0.38) + (200000000 * 0.40) + (annual_income - 500000000) * 0.42;
                deductionValue = annual_income * 0.42 - 35400000;
                finalValue = (annual_income - 500000000);
                i = 6;
            } else {
                taxValue = (12000000 * 0.06) + (34000000 * 0.15) + (42000000 * 0.24) + (62000000 * 0.35) + (150000000 * 0.38) + (200000000 * 0.40) + (500000000 * 0.42) + (annual_income - 1000000000) * 0.45;
                deductionValue = annual_income * 0.45 - 65400000;
                finalValue = (annual_income - 1000000000);
                i = 7;
            }

            double[] taxValArr = {12000000, 34000000, 42000000, 62000000, 150000000, 200000000, 500000000, 1000000000};
            int[] taxPercentArr = {6, 15, 24, 35, 38, 40, 42, 45};

            for (int x = 0; x <= i; x++) {
                if (x != i) {
                    System.out.printf("%10.0f * %2d%% = %.0f\n", taxValArr[x], taxPercentArr[x], taxValArr[x] * (taxPercentArr[x] * 0.01));
                } else {
                    System.out.printf("%10.0f * %2d%% = %.0f\n", finalValue, taxPercentArr[x], finalValue * (taxPercentArr[x] * 0.01));
                }
            }

            System.out.println();
            System.out.printf("[세율에 의한 세금]:\t\t\t\t\t%.0f\t\t\n", taxValue);
            System.out.printf("[누진공제에 계산에 의한 세금]:\t%.0f\t\t\n", deductionValue);
        } catch (InputMismatchException e) {
            System.out.println("숫자를 입력해 주세요.");
            e.printStackTrace();
        }
    }
}
```
