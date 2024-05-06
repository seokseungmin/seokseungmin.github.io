---
title: "CustomException"
excerpt: "[TIL] CustomException"

categories:
  - Today I Learned
tags:
  - [Today I Learned]

permalink: /categories/CustomException/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

# 커스텀 예외(Custom Exception)

일반 예외로 선언할 경우 Exception을 상속받아 구현

실행 예외로 선언할 경우에는 RuntimeException을 상속받아 구현

사용자 정의 예외 클래스는 컴파일러가 체크하는 일반 예외로 선언할 수도 있고, 컴파일러가 체크하지 않는 실행 예외로 선언할 수도 있습니다.
- 일반 예외로 선언할 경우 Exception을 상속하면 되고, 실행 예외로 선언할 경우에는 RuntimeException을 상속하면 됩니다.
- 사용자 정의 예외 클래스 이름은 Exception으로 끝나는 것을 권장합니다.
- 사용자 정의 예외 클래스 작성 시 생성자는 두 개를 선언하는 것이 일반적입니다.
  1. 매개 변수가 없는 기본 생성자
  2. 예외 발생 원인(예외 메시지)을 전달하기 위해 String 타입의 매개변수를 갖는 생성자

예외 메시지의 용도는 catch {} 블록의 예외처리 코드에서 이용하기 위해서입니다.

## 예시

RuntimeException을 상속받아 작성한 사용자 정의 예외입니다.

```java
public class CustomException extends RuntimeException {

    // 1. 매개 변수가 없는 기본 생성자
    CustomException() {

    }

    // 2. 예외 발생 원인(예외 메시지)을 전달하기 위해 String 타입의 매개변수를 갖는 생성자
    CustomException(String message) {
        super(message); // RuntimeException 클래스의 생성자를 호출합니다.
    }
}
```

## 예외 발생시키기

```java
throw new 예외();
throw new 예외("메시지");
```

## 커스텀 예외 활용

강제로 예외를 발생시켜 메소드를 호출한 곳에서 커스텀 예외를 처리합니다.

```java
public static void main(String[] args) {
    try{
        test();
    } catch (CustomException e) {
        System.out.println("커스텀 예외 테스트");
    }
}

public static void test() throws CustomException {
    throw new CustomException("예외 테스트 입니다.");
}
```

## 예외 정보 얻기

try 블록에서 예외가 발생되면 예외 객체는 catch 블록의 매개 변수에서 참조하게 되므로 매개 변수를 이용하면 예외 객체의 정보를 알 수 있습니다.
모든 예외 객체는 Exception 클래스를 상속하기 때문에 Exception이 가지고 있는 메소드들을 모든 예외 객체에서 호출할 수 있습니다.
Exception는 Throwable를 상속받고 Throwable에 객체의 정보를 얻을 수 있는 메소드를 가지고 있습니다.
주로 자주 사용되는 메소드는 getMessage()와 printStackTrace()입니다.

```java
public class Exception extends Throwable {

    public Exception() {
        super();
    }

    public Exception(String message) {
        super(message);
    }
    
    // (. . .) 생략
}

public class Throwable implements Serializable {

    public String getMessage() {
        return detailMessage;
    }    
    
    public void printStackTrace() {
        printStackTrace(System.err);
    }
    
    // (. . .) 생략
}
```

### getMessage()

String 타입의 매개변수 메시지를 갖는 생성자를 이용하였다면, 메시지는 자동적으로 예외 객체 내부에 저장되게 되는데 이 메시지를 리턴하는 함수입니다.
예외 메시지의 내용에는 왜 예외가 발생했는지에 대한 간단한 설명이 포함됩니다.

### printStackTrace()

예외 발생 코드를 추적해서 모두 콘솔에 출력합니다.
어떤 예외가 어디에서 발생했는지 상세하게 출력해주기 때문에 프로그램을 테스트하면서 오류를 찾을 때 활용합니다.

## 커스텀 예외 활용

```java
public static void main(String[] args) {

    try{
        test();
    } catch (CustomException e) {
        System.out.println("커스텀 예외 테스트");
        System.out.println(e.getMessage());
        e.printStackTrace();
    }
}

public static void test() throws CustomException {
    throw new CustomException("예외 테스트 입니다.");
}
```

**참고 블로그:**
[sayit.tistory](https://veneas.tistory.com/entry/Java-%EC%BB%A4%EC%8A%A4%ED%85%80-%EC%98%88%EC%99%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0Custom-Exception)
