---
title: "BufferedReader, StringTokenizer, Scanner"
excerpt: "[TIL] BufferedReader, StringTokenizer, Scanner"

categories:
  - Today I Learned
tags:
  - [Today I Learned]

permalink: /categories/BufferedReaderStringTokenizerScanner/ # url

toc: true
toc_sticky: true

date: 2024-05-22
last_modified_at: 2024-05-22
---

# BufferedReader와 Scanner의 차이점 및 사용법

## 개요
자바에서 입력을 처리할 때 `BufferedReader`와 `Scanner` 클래스를 자주 사용합니다. 이 글에서는 `BufferedReader`를 중심으로 `Scanner`와의 차이점 및 사용법을 정리합니다.

## 본론

### BufferedReader 사용법

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
```

#### 분석

1. **System.in**:
   - 콘솔에서 데이터를 입력받기 위한 `InputStream` 타입의 필드입니다. `System` 클래스의 `in` 정적 필드로, 기본적으로 바이트 스트림으로 입력을 처리합니다.
   - `System.in`으로 받은 `InputStream` 객체는 `read` 함수를 통해 1바이트만 읽을 수 있어, 한글과 같은 2바이트 문자는 읽을 수 없습니다.

2. **InputStreamReader**:
   - `InputStream` 객체를 입력으로 받아, 바이트 스트림을 문자 스트림으로 변환합니다. 따라서 `new InputStreamReader(System.in)`은 바이트 데이터를 문자 데이터로 읽을 수 있게 합니다.
   - 하지만, 배열 크기를 일일이 지정해야 하는 불편함이 있습니다.

3. **BufferedReader**:
   - `InputStreamReader`를 입력받아 문자 스트림을 더 효율적으로 처리합니다. `BufferedReader`는 개행 문자가 입력되기 전까지의 모든 텍스트를 저장하고, 스트림이 다 차거나 `null`이 아니라면 그 값을 계속 갖고 있습니다.
   - 많은 데이터를 입력받을 경우 `Scanner`보다 메모리적으로 더 효율적입니다.

### 활용 예제

#### String 입력

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
String s = reader.readLine(); // 한 줄을 읽어 String 타입으로 값을 가져옵니다.
```

#### String 배열 입력

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
String[] strArrN = reader.readLine().split(" "); // 한 줄을 읽어 " "을 기준으로 나눠 String 배열로 만듭니다.
```

#### int 입력

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
int i = Integer.parseInt(reader.readLine()); // 한 줄을 읽어 숫자로 변환하여 int로 저장합니다.
```

#### int 배열 입력 및 정렬

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
String[] strArrN = reader.readLine().split(" ");
int[] arrAList = new int[strArrN.length];
for(int i = 0; i < strArrN.length; i++) {
    arrAList[i] = Integer.parseInt(strArrN[i]);
}
Arrays.sort(arrAList); // 오름차순으로 정렬
```

### Scanner 클래스

#### Scanner란?

`Scanner` 클래스는 기본 타입과 문자열을 구문 분석할 수 있는 클래스입니다. 예를 들어, 숫자나 문자열 등을 분석하여 입력값을 처리합니다.

#### Scanner 사용법

```java
Scanner sc = new Scanner(System.in);
int a = sc.nextInt(); // 입력받은 int형 숫자를 변수에 저장
String b = sc.next(); // 입력받은 String형 문자열을 변수에 저장
long c = sc.nextLong(); // 입력받은 long형 숫자를 변수에 저장
```

#### BufferedReader와 Scanner의 차이점

- **속도**:
  - `Scanner`는 입력 즉시 프로그램에 전송되고, `BufferedReader`는 버퍼를 사용해 입력을 받아 효율적으로 처리합니다. 따라서 `BufferedReader`가 더 빠릅니다.

### StringTokenizer 클래스

#### StringTokenizer란?

`StringTokenizer` 클래스는 지정된 구분자로 문자열을 분리하여 토큰으로 나누는 클래스입니다.

#### StringTokenizer 사용법

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
StringTokenizer st = new StringTokenizer(br.readLine()); // 한 줄을 읽어 구분자로 분리할 준비를 합니다.
String b = st.nextToken(); // 첫 번째 토큰을 변수에 저장합니다.
```

## 마무리

### 정리

- **InputStream(= System.in)**:
  - 읽어오는 함수: `read()`
  - 단위: byte
  - 단점: 한글을 읽을 수 없다.

- **InputStreamReader**:
  - 읽어오는 함수: `read()`
  - 단위: char
  - 단점: 배열 크기 지정이 번거롭다.

- **BufferedReader**:
  - 읽어오는 함수: `readLine()`
  - 단위: String
  - 단점: 앞선 경우를 모두 담아야 해서 코드가 복잡해질 수 있다.

- **Scanner**:
  - 장점: 다양한 데이터 타입 지원, 사용 편리성
  - 단점: 속도가 느리다.

코딩 테스트에서는 `BufferedReader`와 `StringTokenizer`를 많이 사용하게 되는데, 이는 `Scanner`보다 속도가 빠르기 때문입니다. `BufferedReader`와 함께 `StringTokenizer`를 사용하면 입력 데이터를 효율적으로 분리할 수 있습니다.
