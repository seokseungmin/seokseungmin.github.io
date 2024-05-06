---
title: "InputMismatchException"
excerpt: "[TIL] InputMismatchException"

categories:
  - Today I Learned
tags:
  - [Today I Learned]

permalink: /categories/InputMismatchException/ # url

toc: true
toc_sticky: true

date: 2024-05-06
last_modified_at: 2024-05-06
---


## InputMismatchException

정수를 입력해야 하는데 사용자가 문자를 입력하는 경우 InputMismatchException이 발생합니다.<br>
이 예외는 Scanner를 사용하여 값을 읽을 때, 예상한 유형과 다른 유형의 입력이 들어왔을 때 발생합니다.<br>
이 문제를 해결하기 위해 while 루프를 사용하여 사용자로부터 올바른 입력을 받을 때까지 계속해서 입력을 요청합니다.<br>
try-catch 블록 내에서 InputMismatchException을 처리하여 사용자가 올바른 형식의 입력을 하지 않은 경우 메시지를 출력하고,<br>
Scanner의 nextLine() 메서드를 사용하여 입력 버퍼를 비워줍니다. 그 후, 사용자로부터 새로운 입력을 받습니다.<br>

```java
Scanner scanner = null;
int number = 0;
		
while(true) { 
    scanner = new Scanner(System.in);

    try {  
        System.out.print("정수를 입력하세요: "); 
        number = scanner.nextInt();
        break;
    } catch (InputMismatchException e) {
        System.out.println("잘못된 입력입니다. 정수만 입력해주세요.");
        scanner.nextLine(); // 입력 버퍼 비우기
    }
}
		
System.out.println("입력한 정수: " + number);
scanner.close();
```

이렇게 하면 사용자가 정수 이외의 값을 입력하더라도 프로그램이 멈추지 않고 계속해서 사용자로부터 올바른 입력을 받을 수 있습니다.

**참고 블로그:**
[Say It!, IT-story를 말하다](https://sayit.tistory.com/entry/InputMismatchException-java)
