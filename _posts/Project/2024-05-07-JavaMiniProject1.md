---
title: "Java Mini Project1"
excerpt: "[제로베이스] 구구단 출력"

categories:
  - Java
tags:
  - [Java]

permalink: /categories/JavaMiniProject1/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## 필수 준수 사항
다중 반복문을 이용하여 코딩<br>
콘솔화면에 내용이 맞도록 format함수 이용하여 코딩<br>
제목 및 1단부터 9단까지 표시(반드시, 예시와 동일한 레이아웃으로 작성)<br>

## 권장 사항
String.format 함수를 학습해 보세요.<br>
반복문에 대한 초기값은 주어진 조건에 맞게 작성해 보세요.<br>
코드 작성시, 복잡하게 작성하기 보다는 최대한 필요한 코드를 통한 심플하게 작성해 주세요.<br>

```java
public class Main {
    public static void main(String[] args) {

        System.out.println("[구구단 출력]");
        gugudan();

    }

    private static void gugudan() {
        for (int i = 1; i <= 9; i++) {
            for (int j = 1; j <= 9; j++) {
                System.out.print(String.format("%02d", j) + " * " + String.format("%02d", i) + " = " + String.format("%02d", i * j));
                System.out.print("    ");
            }
            System.out.println();
        }
    }
}
```
