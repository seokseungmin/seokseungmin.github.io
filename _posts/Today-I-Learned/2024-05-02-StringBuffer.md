---
title: "StringBuffer"
excerpt: "[TIL] StringBuffer"

categories:
  - Today I Learned
tags:
  - [Today I Learned]

permalink: /categories/StringBuffer/ # url

toc: true
toc_sticky: true

date: 2024-05-02
last_modified_at: 2024-05-02
---

개요
---

String의 내부 값은 변경할 수 없다. <br>
아래 코드에서 immutable 객체를 수정한 것처럼 보이지만, "abcdef"라는 새 객체를 생성한 것이다.<br>

```java
String immutable = "abc";
immutable = immutable + "def";
```

내부값이 변경 가능한 문자열을 만드려면 StringBuilder 또는 StringBuffer을 사용해야 한다.<br>

```java
StringBuffer sb = new StringBuffer("abc");
sb.append("def");
```

StringBuffer의 append 함수를 통해 내부 값을 변경하고 있다.<br>
String과 String Buffer, String Builder로 문자열을 수정할 때 어떤 차이점 있는지 알아보자.<br>

String vs String Buffer vs String Builder
---
String 객체는 내부값이 변하지 않기 때문에, 문자열을 수정할 때마다 새로운 객체를 생성한다.<br>
따라서 문자열 값만 변경하는 String Builder나 String Buffer에 비해 상대적으로 성능이 떨어진다.<br>
하지만 불변성 덕분에 여러 스레드간에 안전하게 공유할 수 있다는 장점이 있다.<br>

String Buffer는 append를 통해 내부 값을 수정할 수 있고, 스레드에 안전하다.
String Builder는 스레드에 안전하지 않지만, 안전한 구현인 String Buffer에 비해 빠르다. 하지만 성능에서 큰 차이는 없다.
상황에 맞춰 여러 스레드 사이에서 문자열을 수정할 일이 있으면 String Buffer를 사용하고, 단일 스레드에서 문자열 수정은 String Builder를 사용한다

### append 사용예시

```java
StringBuffer sb = new StringBuffer("abc");
StringBuffer sb2 = sb.append(true);
sb.append('d').append(10.0);

// sb = "abctrue10.0"
// sb2 = "abctrue10.0"
```

### insert사용예시

```java
StringBuffer insert(int pos, boolean b)
StringBuffer insert(int pos, char c)
StringBuffer insert(int pos, char[ ] str)
StringBuffer insert(int pos, double d)
StringBuffer insert(int pos, float f)
StringBuffer insert(int pos, int i)
StringBuffer insert(int pos, long l)
StringBuffer insert(int pos, Object obj)
StringBuffer insert(int pos, String str)
두 번째 매개변수로 받은 값을 문자열로 변환화여 지정된 위치에 추가한다. pos는 0부터 시작
StringBuffer sb = new StringBuffer("0123456");
sb.insert(4,'.');

// sb = 0123.456
```

### delete 사용예시

```java
StringBuffer delete(int start, int end)
시작위치부터 끝위치 사이에 있는 문자 제거 (끝 위치의 문자는 제외)
StringBuffer sb = new StringBuffer("0123456");
StringBuffer sb2 = sb.delete(3,6);

sb = "0126"
sb2 = "0126"
```

### deleteCharAt 사용예시

```java
StringBuffer deleteCharAt(int index)
지정된 위치의 문자 제거
StringBuffer sb = new StringBuffer("0123456");
sb.deleteCharAt(3);

sb = "012456"
```

**참고 블로그:**
- [자바 문자열 이것은 알고 쓰자](https://velog.io/@paulhana6006/%EC%9E%90%EB%B0%94-%EB%AC%B8%EC%9E%90%EC%97%B4-%EC%9D%B4%EA%B1%B0%EB%8A%94-%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%902)
- [자바의 정석 StringBuffer클래스의 메서드](https://velog.io/@rnfaos77/%EC%9E%90%EB%B0%94%EC%9D%98-%EC%A0%95%EC%84%9D-StringBuffer%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98-%EB%A9%94%EC%84%9C%EB%93%9C)

**참고 하면 좋을 블로그:**
- [JAVA ☕ String, StringBuffer, StringBuilder 차이점, 성능 비교](https://inpa.tistory.com/entry/JAVA-%E2%98%95-String-StringBuffer-StringBuilder-%EC%B0%A8%EC%9D%B4%EC%A0%90-%EC%84%B1%EB%8A%A5-%EB%B9%84%EA%B5%90)
