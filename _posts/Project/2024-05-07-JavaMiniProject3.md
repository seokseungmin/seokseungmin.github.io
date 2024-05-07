---
title: "Java Mini Project3"
excerpt: "[제로베이스] 놀이동산 입장권 계산 프로그램"

categories:
  - Project
tags:
  - [Project]

permalink: /categories/JavaMiniProject3/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## 놀이동산 입장권 계산 프로그램
수행 목적 : Scanner의 입력함수와 다중 조건문을 통한 입장권 계산 로직 작성<br>
간략 소개 : 놀이동산의 입장권은 나이와 기타 우대사항에 따라 입장료가 달라집니다.<br>
문제에서 주어진 조건에 맞는 입장료를 구하는 프로그램을 작성해 보세요.<br>

## 필수 준수사항
놀이공원 입장료를 구하는 조건에 맞게 로직 작성<br>
입력내용은 나이, 입장시간, 국가유공자 여부, 복지카드 여부 순으로 입력<br>

놀이공원 입장료 할인은 일반 할인과 특별 할인이 있습니다.<br>
조건은 아래와 같습니다.<br>
입장료 할인은 중복할인 되지 않으며, 중복될 경우 가장 할인을 많이 받은 금액으로 정해집니다.<br>
- 3세미만이면 무료 입장<br>
- 복지카드와 국가유공자의 경우 일반 할인 적용<br>
- 13세미만이면 특별 할인 적용<br>
- 17시이후에 입장하면 특별 할인 적용<br>
기본 입장료 : 10,000원<br>
특별 할인의 경우 : 4,000원<br>
일반 할인의 경우 : 8,000원<br>

```java
import java.util.InputMismatchException;
import java.util.Scanner;

public class Project3 {
    public static void main(String[] args) {
        int age = 0;
        int entranceTime = 0;
        int entranceFee = 10000; // 기본 입장료
        boolean nationalMeritorious = false;
        boolean welfareCard = false;

        System.out.println("놀이동산 입장권 계산 프로그램");
        Scanner scanner = new Scanner(System.in);

        try {
            // 입력 받기
            System.out.print("나이를 입력해 주세요. (숫자): ");
            age = scanner.nextInt();

            System.out.print("입장 시간을 입력해 주세요. (숫자입력): ");
            entranceTime = scanner.nextInt();

            System.out.print("국가 유공자 여부를 입력해 주세요. (y/n): ");
            char nationalMeritoriousAvailability = scanner.next().charAt(0);
            if (nationalMeritoriousAvailability == 'y') {
                nationalMeritorious = true;
            }

            System.out.print("복지 카드 여부를 입력해 주세요. (y/n): ");
            char welfareCardAvailability = scanner.next().charAt(0);
            if (welfareCardAvailability == 'y') {
                welfareCard = true;
            }

            entranceFee(age, entranceFee, nationalMeritorious, welfareCard, entranceTime);
        } catch (InputMismatchException e) {
            System.out.println("입력 형식이 잘못되었습니다. 숫자를 입력해주세요.");
        }
    }

    private static void entranceFee(int age, int entranceFee, boolean nationalMeritorious, boolean welfareCard, int entranceTime) {
        // 할인 계산
        if (age < 3) {
            entranceFee = 0; // 3세 미만은 무료
        } else {
            if (nationalMeritorious || welfareCard) {
                entranceFee = 8000; // 국가 유공자나 복지카드 소지자는 일반 할인
            }
            if (age < 13 || entranceTime >= 17) {
                entranceFee = 4000; // 13세 미만이거나 17시 이후 입장은 특별 할인
            }
        }

        System.out.println("입장료: " + entranceFee);
    }
}
```
