---
title: "스프링 핵심가이드 2주차"
excerpt: "[제로베이스] 스프링 핵심가이드"

categories:
  - Spring
tags:
  - [Spring]

permalink: /categories/Spring2Week/ # url

toc: true
toc_sticky: true

date: 2024-07-29
last_modified_at: 2024-07-29
---

본 게시글은 '스프링 부트 핵심 가이드' 책의 내용을 정리한 것입니다.<br>
저자 : 장정우<br>
출판사 : 위키북스<br>
 
# 4장 스프링 부트 애플리케이션 개발하기

## 4.1 프로젝트 생성

### 4.1.1 인텔리제이 IDEA에서 프로젝트 생성하기
**IntelliJ IDEA Ultimate 버전**

IntelliJ IDEA Ultimate 버전에 내장된 Spring Initializr를 사용하여 외부에서 프로젝트를 생성할 필요 없이 곧바로 스프링 프로젝트를 생성할 수 있습니다.

- **Name**: 프로젝트의 이름을 설정합니다.
- **Location**: 프로젝트 생성할 위치를 지정합니다.
- **Language**: JVM 상에서 동작하는 언어를 선택합니다 (예: Java).
- **Group**: 이 프로젝트를 정의하는 고유한 식별자 정보인 그룹을 설정합니다.
- **Type**: 빌드 툴을 선택합니다. 과거에는 Maven을 많이 사용했으나 요즘은 비교적 최신인 Gradle을 많이 이용합니다.
- **Artifact**: 세부 프로젝트를 식별하는 정보를 기입합니다. 프로젝트 명과 동일하게 작성해도 무관합니다.
- **Package name**: 그룹과 Artifact를 설정하면 자동으로 입력됩니다.
- **Project SDK**: JDK 버전을 설정합니다.
- **Java**: 자바 언어 버전을 설정합니다.
- **Packaging**: 애플리케이션을 쉽게 배포하고 동작하게 할 파일들의 패키징 옵션입니다.
  - **Jar / War**: 자바 언어의 툴에서 사용하는 아카이브 파일. (애플리케이션의 배포와 동작을 위해 사용)

프로젝트에 사용할 의존성 추가

### 4.1.2 스프링 공식 사이트에서 프로젝트 생성하기 (Spring Initializr)
- [Spring Initializr](https://start.spring.io/)
- 의존성 추가 후 'GENERATE'로 다운로드 후 압축을 풀고 인텔리제이 IDEA로 실행.

## 4.2 pom.xml (Project Object Model) 살펴보기

Maven의 핵심 부분인 pom.xml 파일은 Maven 프로젝트의 기본 설정 파일로서, 이 파일에는 프로젝트 구성, 의존성, 플러그인, 목표 빌드 프로파일 등의 프로젝트와 관련된 모든 정보가 포함됩니다.

### 4.2.1 빌드 관리 도구

빌드 관리 도구는 JVM이나 WAS가 프로젝트를 인식하고 실행할 수 있게 우리가 작성한 소스코드와 프로젝트에 사용된 파일들(.xml, .jar, .properties)을 빌드하는 도구입니다. 개발 규모가 커질수록 관리할 라이브러리가 많아지고 라이브러리 간 버전 호환성을 체크해야 하는 어려움이 발생하는데, 빌드 관리 도구를 이용하면 이 같은 문제를 해결할 수 있습니다.

### 4.2.2 메이븐 (Maven)

아파치 메이븐은 자바 기반 프로젝트를 빌드하고 관리하는 데 사용하는 도구로 메이븐의 가장 큰 특징은 pom.xml 파일에 필요한 라이브러리를 추가하면 해당 라이브러리에 필요한 라이브러리까지 함께 내려받아 관리한다는 점입니다.

- **프로젝트 관리**: 프로젝트 버전과 아티팩트 관리.
- **빌드 및 패키징**: 의존성 관리, 설정된 패키지 형식으로 빌드 수행.
- **테스트**: 빌드를 수행하기 전에 단위 테스트를 통해 작성된 애플리케이션 코드의 정상 동작 여부 확인.
- **배포**: 빌드가 완료된 패키지를 원격 저장소에 배포.

#### 4.2.2.1 메이븐의 생명주기

- **클린 생명주기 (clean)**: 이전 빌드가 생성한 모든 파일을 제거합니다.
- **기본 생명주기**
  - **validate**: 프로젝트를 빌드하는 데 필요한 모든 정보를 사용할 수 있는지 검토합니다.
  - **compile**: 프로젝트의 소스코드를 컴파일합니다.
  - **test**: 단위 테스트 프레임워크를 사용해 테스트를 실행합니다.
  - **package**: 컴파일한 코드를 가져와서 JAR 등의 형식으로 패키징을 수행합니다.
  - **verify**: 패키지가 유효하며 일정 기준을 충족하는지 확인합니다.
  - **install**: 프로젝트를 사용하는 데 필요한 패키지를 로컬 저장소에 설치합니다.
  - **deploy**: 프로젝트를 통합 또는 릴리스 환경에서 다른 곳에 공유하기 위해 원격 저장소에 패키지를 복사합니다.
- **사이트 생명주기**
  - **site**: 메이븐의 설정 파일 정보를 기반으로 프로젝트의 문서 사이트를 생성합니다.
  - **site-deploy**: 생성된 사이트 문서를 웹 서버에 배포합니다.

각 생명주기 단계는 이전 단계가 성공적으로 완료된 후에 실행됩니다.

## 4.3 Hello World 출력하기

### 4.3.1 컨트롤러 작성하기

**controller 패키지 및 HelloController 클래스 생성**

스프링 MVC(Model-View-Controller)의 컨트롤러는 사용자 요청 처리에 있어 핵심 역할을 수행하며, MVC 패턴에서 "Controller" 부분을 차지합니다.

#### 컨트롤러 코드 작성

```java
package com.example.hello.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello() {
        return "Hello World";
    }
}
```

#### 컨트롤러에서 수행되는 주요 작업

1. 사용자의 요청을 받음.
2. 요청에 맞는 모델(Model)의 메서드나 기능을 호출.
3. 모델의 반환 값을 처리하여 사용자에게 표시할 결과를 생성.
4. 결과를 뷰에 전달하여 사용자에게 웹 페이지를 표시.

### 4.3.2 애플리케이션 실행하기

스프링 부트 애플리케이션을 실행하려면, `SpringBootApplication`이 선언된 메인 클래스를 실행하면 됩니다. 정상적으로 실행되면 콘솔에 "Tomcat started on port(s): 8080 (http) with context path"와 같은 메시지가 표시됩니다.

### 4.3.3 웹 브라우저를 통한 동작 테스트

웹 브라우저에서 `http://localhost:8080/hello`에 접속하여 "Hello World"가 출력되는지 확인합니다.

### 4.3.4 Talend API Tester를 통한 동작 테스트

웹 브라우저를 통한 동작 테스트는 간편하지만 상세한 응답을 확인할 수 없다는 단점이 있습니다. 따라서 Talend API Tester 크롬 확장 프로그램 또는 포스트맨(Postman) 등의 프로그램을 사용합니다.

#### Talend API Tester 사용 예시

1. Talend API Tester 크롬 확장 프로그램을 설치하고 실행합니다.
2. 새로운 GET 요청을 생성합니다.
3. URL에 `http://localhost:8080/hello`를 입력합니다.
4. **Send** 버튼을 클릭하여 요청을 보냅니다.

**요청 예시:**

```
GET /hello HTTP/1.1
Host: localhost:8080
```

**응답 예시:**

```
HTTP/1.1 200 OK
Content-Type: text/plain;charset=UTF-8
Content-Length: 11
Date: Mon, 29 Jul 2024 12:34:56 GMT

Hello World
```

- **Body**: "Hello World"
- **Headers**:
  - Content-Type: text/plain;charset=UTF-8
  - Content-Length: 11
  - Date: Mon, 29 Jul 2024 12:34:56 GMT

위와 같이 Talend API Tester를 사용하면 요청과 응답의 상세한 내용을 확인할 수 있습니다.

# 05장: API를 작성하는 다양한 방법

## Spring Boot Controller, Service, and RESTful Annotations

Controller는 클라이언트로부터 요청이 오면 해당 요청을 받고 Service 계층에서 처리 후, 다시 응답을 반환하는 계층이다.

## @RestController와 @RequestMapping

### @RestController
보통은 Controller 계층의 클래스에 @Controller를 붙여 작성한다. 그럼 @RestController는 무엇일까?

@RestController는 @ResponseBody + @Controller 라고 생각하면 될 것 같다. @Controller는 ViewResolver가 작동해 뷰를 반환하지만 @RestController는 클라이언트 측에 데이터를 전송할 때 사용하므로 MessageConverter가 작동한다.

### @RequestMapping
@RequestMapping 어노테이션을 별 설정 없이 선언하면 HTTP의 모든 요청을 받는다.

하지만 클래스에 다음과 같이 설정하면 클래스에 작성하는 메서드 앞에 자동적으로 해당 URL이 붙는다.

```java
@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {
    //http://localhost:8080/api/v1/get-api/hello
    @RequestMapping(value = "hello", method = RequestMethod.GET)
    public String getHello() {
        return "Hello";
    }
}
```

대부분의 상황에서는 이렇게 작성하진 않고 아래와 같이 작성한다.

```java
@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {
    //http://localhost:8080/api/v1/get-api/hello
    @GetMapping("hello")
    public String getHello() {
        return "Hello";
    }
}
```

## URI와 URL의 차이
URL은 우리가 흔히 말하는 웹 주소를 의미하며 리소스가 어디에 있는지 알려주기 위한 경로를 의미한다.  
URI는 특정 리소스를 식별할 수 있는 식별자를 의미한다.  
웹에서는 URL을 통해 리소스가 어느 서버에 위치해 있는지 알 수 있으며, 그 서버에 접근해서 리소스를 접근하기 위해서는 대부분 URI가 필요하다.

## @GetMapping

HTTP Method 중 GET은 서버에 있는 자원을 요청할 때 사용한다. 이때 중요한 것은 GET은 Body가 없지만 여러가지 파라미터를 전송할 수 있다. 여러가지 파라미터는 URL에 작성하여 전송하게 되는데, 크게 2가지 방법이 있다.

### @PathVariable을 활용
아래 예시 코드를 보자.

```java
//http://localhost:8080/api/v1/get-api/variable1/{String값}
@GetMapping("/variable1/{variable}")
public String getVariable(@PathVariable String variable) {
    return variable;
}
```

위와 같은 방식으로 URL에 직접 값을 넣어서 전송할 수 있다. `localhost:8080/api/v1/get-api/variable1/hello` 와 같은 형태로 전송한다면 메서드의 `variable`에는 `hello`라는 값이 들어가게 된다.

하지만 위와 같이 작성할 땐 지켜야 할 규칙이 있다.

- @GetMapping 어노테이션의 값으로 URL을 입력할 때 중괄호를 사용해 어느 위치에서 값을 받을지 정해야 한다.
- 메서드 파라미터를 받는 위치에 @PathVariable을 명시하며 @GetMapping 어노테이션과 @PathVariable에 저장된 변수의 이름을 동일하게 맞춰야 한다.

만약 @GetMapping 어노테이션에 지정한 변수의 이름과 @PathVariable 변수 이름을 동일하게 맞추기 어렵다면 아래와 같이 @PathVariable 뒤에 괄호를 열어 @GetMapping 어노테이션의 변수명을 지정해야 한다.

```java
@GetMapping(value = "/variable2/{variable}")
public String getVariable2(@PathVariable("variable") String var) {
    return var;
}
```

### @RequestParam을 활용
@PathVariable과 같이 직접적으로 값을 담을 수도 있지만 쿼리 파라미터 형식으로 값을 담을 수도 있다. 쿼리 파라미터란 '?'을 기준으로 우측에 {키}={값} 형태로 구성된 요청이다. 이러한 쿼리 파라미터를 활용한 요청에서 @RequestParam을 사용한다.

```java
@GetMapping("/request1")
public String getReqeustParam1(
    @RequestParam String name,
    @RequestParam String email,
    @RequestParam String organization) {
    
    return name + " " + email + " " + organization;
}
```

위와 같이 @RequestParm이 지정된 변수는 쿼리 파라미터에서 Key에 해당한다. 따라서 위 코드의 요청 예시는 아래와 같다.

`localhost:8080/api/v1/get-api/request1?name=value1&email=value2&organization=value3`

기존 URL 뒤에 '?'로 시작하여 key=value 형태로 작성하고, 여러 개가 있을 땐 '&'로 연결한다.

또한 @RequestParam 뒤에 적는 이름을 동일하게 설정하기 어렵다면 @PathVariable처럼 value 요소로 매핑하면 된다.

### 쿼리스트링에 값을 특정할 수 없을 때
만약에 쿼리스트링에 어떤 값이 들어올지 모른다면 아래와 같이 Map 객체를 활용할 수 있다.

```java
@GetMapping(value = "/request2")
public String getRequestParam2(@RequestParam Map<String, String> param) {
    StringBuilder sb = new StringBuilder();
    
    param.entrySet().forEach(map -> {
        sb.append(map.getKey() + " : " + map.getValue() + "\n");
    });
    
    return sb.toString();
}
```

위와 같이 Map 객체를 활용하면 값에 상관없이 요청을 받을 수 있다.

## DTO 객체를 활용한 GET 메서드 구현
DTO란 Data Transfer Object의 약자로 다른 레이어 간 데이터 교환에 활용된다. 쉽게 설명하면 각 클래스 및 인터페이스를 호출하면서 전달하는 매개변수로 사용되는 데이터 객체이다. DTO는 데이터를 교환하는 용도로만 사용되는 객체이기 때문에 별도의 로직이 포함되지 않는다.

### MemberDto

```java
@Getter
@Setter
@ToString
public class MemberDto {
    private String name;
    private String email;
    private String organization;
}
```

DTO 객체는 POST 요청에서 객체로 받기 위해 사용하는 줄 알았는데 GET 요청에서도 사용할 수 있다는 걸 처음 알았다. 이렇게 선언하면 DTO 클래스에 선언된 필드는 컨트롤러의 메서드에서 쿼리 파라미터의 키와 매핑된다. 즉, 쿼리스트링의 키가 정해져 있지만 받아야 할 파라미터가 많을 경우에는 DTO 객체를 활용할 수 있다.

```java
@GetMapping("/request3")
public String getRequestparam3(MemberDto memberDto) {
    return memberDto.toString();
}
```

## @PostMapping
POST API는 웹 애플리케이션을 통해 데이터베이스 등의 저장소에 리소스를 저장할 때 사용되는 API이다. GET API에서는 URL 경로나 파라미터에 변수를 넣어 요청을 보냈지만 POST API에서는 저장하고자 하는 리소스나 값을 HTTP Body에 담아 서버에 전달한다.

### @RequestBody를 활용
Body 영역에 작성되는 값은 일정한 형태를 취한다. 일반적으로 JSON 형태로 전송된다.

#### Map 객체에 Mapping

```java
@PostMapping("/member")
public String postMember(@RequestBody Map<String, Object> postData) {
    StringBuilder ssb = new StringBuilder();
    
    postData.entrySet().forEach(map -> {
        sb.append(map.getKey() + " : " + map.getvalue() + "\n");
    });
    
    return sb.toString();
}
```

위에는 @RequestBody라는 어노테이션을 사용하였는데, 이는 HTTP의 Body 내용을 해당 어노테이션이 지정된 객체에 매핑하는 역할을 한다. Map 객체는 요청을 통해 어떤 값이 들어올지 특정하기 어려울 때 주로 사용한다. 이전까지는 DTO 객체와 같은 객체에 매핑 했는데 Map 형식에도 매핑이 된다는 걸 처음 알았다.

#### DTO 객체에 Mapping
요청 메시지가 들어갈 값이 정해져 있다면 DTO 객체를 매개변수로 삼아 작성할 수 있다.

```java
@PostMapping("/member2")
public String postMemberDto(@RequestBody MemberDto memberDto) {
    return memberDto.toString();
}
```

## @PutMapping
PUT API는 웹 애플리케이션 서버를 통해 데이터베이스 같은 저장소에 존재하는 리소스 값을 업데이트 하는데 사용한다. 기본적으로 요청하는 방식은 POST와 거의 동일하기 때문에 응답을 어떻게 구성하는지 살펴보겠다. (응답도 POST와 거의 동일하다)

### ResponseEntity를 활용
스프링 프레임워크에는 HttpEntity라는 클래스가 있다. HttpEntity는 Header와 Body로 구성된 HTTP 요청과 응답을 구성하는 역할을 수행한다.

#### HttpEntity

```java
public class HttpEntity<T> {
    private final HttpHeaders headers;
    @Nullable
    private final T body;
    
    ...
}
```

RequestEntity와 ResponseEntity는 HttpEntity를 상속받아 구현한 클래스이다. 그중 ResponseEntity는 서버에 들어온 요청에 대해 응답 데이터를 구성해서 전달할 수 있게 한다. 아래와 같이 ResponseEntity는 HttpEntity로부터 HttpHeaders와

 Body를 가지고 자체적으로 HttpStatus를 구현한다.

#### ResponseEntity

```java
public class ResponseEntity<T> extends HttpEntity<T> {
    private final Object status;
    
    ...
}
```

위 클래스를 활용하면 응답 코드 변경, Header와 Body를 쉽게 구성할 수 있다. 이는 PUT 이외에도 다른 메서드들에서도 모두 사용 가능하다.

### ResponseEntity를 활용한 PUT 구현

```java
@PutMapping("/member3")
public ResponseEntity<MemberDto> postMemberDto3(@RequestBody MemberDto memberDto) {
    return ResponseEntity
        .status(HttpStatus.ACCEPTED)
        .body(memberDto);
}
```

## @DeleteMapping
DELETE API는 웹 애플리케이션 서버를 거쳐 데이터베이스 등의 저장소에 있는 리소스를 삭제할 때 사용합니다. 컨트롤러를 통해 값을 받는 단계에서는 간단한 값을 받기 때문에 GET 메서드와 같이 URI에 값을 넣어 요청을 받는 형식으로 구현됩니다.

### @PathVariable을 사용한 예시 코드

```java
package com.example.hello.controller;

import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    
    // http://localhost:8080/api/v1/get-api/variable1/{id}
    @DeleteMapping("/api/v1/get-api/variable1/{id}")
    public String deleteResource(@PathVariable String id) {
        return "Deleted resource with id: " + id;
    }
}
```

위 예시 코드는 `@DeleteMapping` 어노테이션을 사용하여 특정 리소스를 삭제하는 REST API 엔드포인트를 정의합니다. `@PathVariable`을 사용하여 URI 경로에서 `id` 값을 받아옵니다.

### @RequestParam을 사용한 예시 코드

```java
package com.example.hello.controller;

import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    // http://localhost:8080/api/v1/get-api/resource?id=123
    @DeleteMapping("/api/v1/get-api/resource")
    public String deleteResourceById(@RequestParam String id) {
        return "Deleted resource with id: " + id;
    }
}
```

위 예시 코드는 `@RequestParam` 어노테이션을 사용하여 쿼리 파라미터로 `id` 값을 받아 삭제 요청을 처리하는 엔드포인트를 정의합니다.

### Talend API Tester를 통한 동작 테스트

1. Talend API Tester 크롬 확장 프로그램을 설치하고 실행합니다.
2. 새로운 DELETE 요청을 생성합니다.
3. URL에 `http://localhost:8080/api/v1/get-api/resource?id=123`을 입력합니다. 여기서 `id=123`은 삭제하려는 리소스의 실제 ID입니다.
4. **Send** 버튼을 클릭하여 요청을 보냅니다.

**요청 예시:**

```
DELETE /api/v1/get-api/resource?id=123 HTTP/1.1
Host: localhost:8080
```

**응답 예시:**

```
HTTP/1.1 200 OK
Content-Type: text/plain;charset=UTF-8
Content-Length: 28
Date: Mon, 29 Jul 2024 12:34:56 GMT

Deleted resource with id: 123
```

- **Body**: "Deleted resource with id: 123"
- **Headers**:
  - Content-Type: text/plain;charset=UTF-8
  - Content-Length: 28
  - Date: Mon, 29 Jul 2024 12:34:56 GMT

위와 같이 Talend API Tester를 사용하면 DELETE 요청과 응답의 상세한 내용을 확인할 수 있습니다.

