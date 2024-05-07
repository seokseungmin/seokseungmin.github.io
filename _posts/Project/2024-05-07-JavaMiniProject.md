---
title: "Java Mini Project4"
excerpt: "[제로베이스] 주민등록번호 생성 프로그램"

categories:
  - Project
tags:
  - [Project]

permalink: /categories/JavaMiniProject4/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## 주민등록번호 생성 프로그램
수행 목적 : Scanner의 입력함수와 조건문 및 Random클래스를 통한 주민번호 생성 로직 작성<br>
간략 소개 : 주민번호는 출생년도와 출생월과 성별에 대한 내용을 포함하여 만들어지는 숫자로 된 체계입니다.<br>
이에 2020년도 부터 생성 조건이 변경되었습니다. 이를 조건에 맞게 생성하는 프로그램을 작성해 보세요.<br>
입력값은 2020년도 이후로 입력한다는 전제로 작성해 주세요.<br>

## 필수 준수사항
주민등록번호 생성 로직에 맞게 주민등록번호 생성<br>
입력값은 생년, 월, 일, 성별과 임의의 번호를 통해서 생성<br>
임의번호는 Random함수의 nextInt()함수를 통해서 생성 (임의 번호 범위는 1 ~ 999999사이의 값으로 설정)<br>

```java
import java.util.InputMismatchException;
import java.util.Random;
import java.util.Scanner;

public class Project4 {
    public static void main(String[] args) {
        System.out.println("[주민등록번호 계산]");
        int birthYear;
        int birthMonth;
        int birthDay;
        char gender;

        Scanner scanner = new Scanner(System.in);
        Random random = new Random();
        int randomValue = random.nextInt(999999) + 1;
        StringBuilder stringBuilder = new StringBuilder();

        try {
            System.out.print("출생년도를 입력해 주세요. (yyyy):");
            birthYear = scanner.nextInt();
            System.out.print("출생월을 입력해 주세요.(mm):");
            birthMonth = scanner.nextInt();
            System.out.print("출생일을 입력해 주세요.(dd):");
            birthDay = scanner.nextInt();
            System.out.print("성별을 입력해 주세요.(m/f)");
            gender = scanner.next().charAt(0);
            if (gender != 'm' && gender != 'f') {
                throw new genderTypeException("성별 양식 오류발생.");
            }
            getResidentRegistrationNumber(gender, birthYear, stringBuilder, birthMonth, birthDay, randomValue);
        } catch (InputMismatchException e) {
            System.out.println("입력 형식이 잘못되었습니다. 숫자를 입력해주세요.");
        } catch (genderTypeException e) {
            System.out.println("성별을 양식에 맞게 기입해주세요(m/f).");
            System.out.println(e.getMessage());
            e.printStackTrace();
        } finally {
            scanner.close();
        }
    }

    private static void getResidentRegistrationNumber(char gender, int birthYear, StringBuilder stringBuilder, int birthMonth, int birthDay, int randomValue) {
        int genderNum = getGenderNum(gender);
        String birthYearString = Integer.toString(birthYear).substring(2);
        stringBuilder.append(birthYearString).append(birthMonth).append(birthDay).append("-").append(genderNum).append(randomValue);
        System.out.print(stringBuilder);
    }

    private static int getGenderNum(char gender) {
        int genderNum = 0;
        if (gender == 'm') {
            genderNum = 3;
        } else if (gender == 'f') {
            genderNum = 4;
        }
        return genderNum;
    }

    public static class genderTypeException extends RuntimeException {

        // 1. 매개 변수가 없는 기본 생성자
        genderTypeException() {

        }

        // 2. 예외 발생 원인(예외 메시지)을 전달하기 위해 String 타입의 매개변수를 갖는 생성자
        genderTypeException(String message) {
            super(message); // RuntimeException 클래스의 생성자를 호출합니다.
        }
    }
}
```
