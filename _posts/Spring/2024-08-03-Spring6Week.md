---
title: "스프링 핵심가이드 6주차"
excerpt: "[제로베이스] 스프링 핵심가이드"

categories:
  - Spring
tags:
  - [Spring]

permalink: /categories/Spring6Week/ # url

toc: true
toc_sticky: true

date: 2024-08-03
last_modified_at: 2024-08-03
---

[스프링부트 핵심가이드 프로젝트 따라하면서 짠 코드](https://github.com/seokseungmin/valid_exception)<br>

본 게시글은 '스프링 부트 핵심 가이드' 책의 내용을 정리한 것입니다.<br>
저자 : 장정우<br>
출판사 : 위키북스<br>

# 유효성 검사와 예외처리

애플리케이션의 비즈니스 로직이 올바르게 동작하려면 데이터를 사전 검증하는 작업이 필요합니다.
이것을 유효성 검사 또는 데이터 검증이라 부릅니다.
유효성 검사의 예로는 여러 계층에서 들어오는 데이터에 대해 의도한 형식대로 값이 들어오는지 체크하는 과정이 있습니다.
이 같은 유효성 검사(validation)는 프로그래밍에서 매우 중요한 부분이며, 자바에서 가장 신경 써야 하는 것 중 하나로 NullPointException
예외가 있습니다.

# 일반적인 애플리케이션 유효성 검사의 문제점
일반적으로 사용되는 데이터 검증 로직에는 몇가지 문제점이 있습니다.
계층별로 진행하는 유효성 검사는 검증 로직이 각 클래스별로 분산돼 있어 관리하기가 어렵습니다.
그리고 검증 로직에 의외로 중복이 많아 여러 곳에 유사한 기능의 코드가 존재할 수 있습니다.
마지막으로 검증해야 할 값이 많다면 검증하는 코드가 길어집니다.
이러한 문제로 코드가 복잡해지고 가독성이 떨어집니다.

이 같은 문제를 해결하기 위해 자바 진영에서는 2009년부터 Bean Validation 이라는 유효성 검사 프레임워크를 제공합니다.
Bean Validation은 어노테이션을 통해 다양한 데이터를 검증하는 기능을 제공합니다.
Bean Validation을 사용한다는 것은 유효성 검사를 위한 로직을 DTO 같은 도메인 모델과 묶어서 각 계층에서 사용하면서
검증 자체를 도메인 모델에 얹는 방식으로 수횅한다는 의미입니다.
또한 Bean Validation은 어노테이션을 사용한 검증 방식이기 때문에 코드의 간결함도 유지할 수 있습니다.

# Hibernate Vaildator
Hibernate Vaildator는 Bean Validation 명세의 구현체입니다.
스프링 부트에서는 Hibernate Vaildator를 유효성 검사 표준으로 채택해서 사용하고 있습니다.
Hibernate Vaildator는 JSP-303 명세의 구현체로서 도메인 모델에서 어노테이션을 통한 필드값 검증을 가능하게 도와줍니다.

# 스프링 부트에서의 유효성 검사
지금부터 애플리케이션에 유효성 검사 기능을 추가하겠습니다.
기본 프로젝트 뼈대는 7장에서 사용한 패키지와 클래스 구조를 그대로 가져와 만들겠습니다.


#### 프로젝트 생성
groupId: com.springboot  
artifactId: valid_exception

#### 의존성 선택
Lombok, Spring Configuration Processor, Spring Web, Spring Data JPA, MariaDB Driver, Valid
Swagger 의존성 추가:
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.2'

Swagger 설정파일, logback 설정파일 추가:
- config/SwaggerConfig.java
- logback-spring.xml

### 스프링 부트의 유효성 검사
유효성 검사는 각 계층으로 데이터가 넘어오는 시점에 해당 데이터에 대한 검사를 실시합니다.
스프링부트 프로젝트에서는 계층 간 데이터 전송에 대체로 DTO 객체를 활용하고 있기 때문에
다음과 같이 유효성 검사를 DTO 객체를 대상으로 수행하는 것이 일반적입니다.

클라이언트 -> ( 컨트롤러 -> 서비스 -> 리포지토리 ) -> 데이터베이스
[ -> 사이에서 도메인 모델이 유효성검사]

이번 장의 실습을 위한 DTO와 컨트롤러를 생성하겠습니다.
먼저 ValidRequestDto라는 이름의 DTO 객체를 다음과 같이 생성합니다.


```java
package com.springboot.valid_exception.data.dto;

import com.springboot.valid_exception.config.annotation.Telephone;
import com.springboot.valid_exception.group.ValidationGroup1;
import com.springboot.valid_exception.group.ValidationGroup2;
import jakarta.validation.constraints.*;
import lombok.*;

@Data@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidatedRequestDto {
    @NotBlank
    private String name;

    @Email
    private String email;

    @Pattern(regexp = "^01(?:0|1|[6-9])[.-]?(\\d{4})[.-]?(\\d{4})$")
    private String phoneNumber;

    @Min(value = 20)
    @Max(value = 40)
    private int age;

    @Size(min = 0, max = 40)
    private String description;

    @Positive
    private int count;

    @AssertTrue
    private boolean booleanCheck;
}
```

예제를 보면 각 필드에 어노테이션이 선언된 것을 볼 수 있습니다.
각 어노테이션은 유효성 검사를 위한 조건을 설정하는 데 사용됩니다.
대표적인 어노테이션은 다음과 같습니다.

### 문자열 검증
- @Null : null 값만 허용합니다.
- @NotNull : null을 허용하지 않습니다 "", " "는 허용합니다.
- @NotEmpty : null, ""을 허용하지 않습니다. " "는 허용합니다.
- @NotBlank : null, "", " "을 허용하지 않습니다.

### 최댓값/최솟값 검증
- BigDecimal, BigInteger, int, long 등의 타입을 지원합니다.
- @DemicalMax(value = "$numberString") : $numberString보다 작은 값을 허용합니다.
- @DemicalMin(value = "$numberString") : $numberString보다 큰 값을 허용합니다.
- @Min(value = $number) : $number 이상의 값을 허용합니다.
- @Max(value = $number) : $number 이하의 값을 허용합니다.

### 값의 범위 검증
- BigDecimal, BigInteger, int, long 등의 타입을 지원합니다.
- @Positive : 양수를 허용합니다.
- @PositiveOrZero: 0을 포함한 양수를 허용합니다.
- @Negative : 음수를 허용합니다.
- @NegativeOrZero : 0을 포함한 음수를 허용합니다.

### 시간에 대한 검증
- Date, LocalDate, LocalDateTime 등의 타입을 지원합니다.
- @Future : 현재보다 미래의 날짜를 허용합니다.
- @FutureOrPresent : 현재를 포함한 미래의 날짜를 허용합니다.
- @Past :  현재보다 과거의 날짜를 허용합니다.
- @PastOrPresent : 현재를 포함한 과거의 날짜를 허용합니다.

### 이메일 검증
- @Email : 이메일 형식을 검사합니다. ""는 허용합니다.

### 자릿수 범위 검증
- BigDecimal, BigInteger, int, long 등의 타입을 지원합니다.
- @Digits(integer = $number1, fraction = $number2): $number1의 정수 자릿수와 $number2의 소수 자릿수를 허용합니다.

### Boolean 검증
- @AssertTrue : true인지 체크합니다. null 값은 체크하지 않습니다.
- @AssertFalse : false인지 체크합니다. null 값은 체크하지 않습니다.
 
### 문자열 길이 검증
- @Size(min = $number1, max = $number2): $number1 이상 $number2 이하의 범위를 허용합니다.

### 정규식 검증
- @Pattern(regexp = "$expression"): 정규식을 검사합니다. 정규식은 자바의 java.util.regex.Pattern 패키지의 컨벤션을 따릅니다.

- 유효성 검사에 사용하는 어노테이션은 인텔리제이 IDEA에서도 확인할 수 있습니다.
- 화면 우측의 [Bean Validation] 탭을 클릭하면 목록을 확인할 수 있습니다.

- 인텔리제이 IDEA에서는 자동으로 우측에 [Bean Validation] 탭을 추가하는 기능이 있는데, 만약 화면에서 이를
- 확인할 수 없다면 메뉴에서 [View] -> [Toll Windows] -> [Bean Validation]을 차례로 선택해 탭을 추가할 수 있습니다.
- 다음으로 앞에서 생성한 DTO를 사용하는 컨트롤러 객체를 생성하겠습니다.
- 다음과 같이 ValidationController를 생성합니다.

```java
  package com.springboot.valid_exception.data.controller;

import com.springboot.valid_exception.data.dto.ValidRequestDto;
import com.springboot.valid_exception.data.dto.ValidatedRequestDto;
import com.springboot.valid_exception.group.ValidationGroup1;
import com.springboot.valid_exception.group.ValidationGroup2;
import jakarta.validation.Valid;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RequestMapping("/validation")
public class ValidationController {

    private final Logger LOGGER = LoggerFactory.getLogger(ValidationController.class);

        @PostMapping("/valid")
    public ResponseEntity<String> checkValidationByValid(@Valid @RequestBody ValidatedRequestDto validRequestDto) {
        LOGGER.info(validRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validRequestDto.toString());
    }
}
```

예제에서 checkValidationValid() 메서드는 ValidRequestDto 객체를 RequestBody 값으로 받고 있습니다.
이 경우 9번 줄과 같이 @Valid 어노테이션을 지정해야 DTO 객체에 대해 유효성 검사를 수행합니다.

동작을 확인하기 위해 애플리케이션을 실행해 Swagger 페이지에 접속합니다.
checkValidationByValid() 메서드를 호출하기 위해 각 값을 그림과 같이 입력합니다.

```
{
  "name": "Flature",
  "email": "flature@wikibooks.co.kr",
  "phoneNumber": "010-1234-5678",
  "age": 30,
  "description": "Validation 실습 데이터입니다.",
  "count": 1,
  "booleanCheck": true
}
```

위의 값은 위에서 설정한 유효성 검사를 통과할 수 있는 값들입니다.
먼저 age는 @Min(value=20), @Max(value=40)으로 값이 20살 이상, 40살 이하인 데이터만 받겠다는 것을 의미합니다.
booleanCheck는 @AssertTrue를 통해 값이 true인지 체크합니다.
count에는 @Positive가 설정돼 있으므로 0이 아닌 양수가 값으로 들어오는지 체크합니다.
description은 @Size를 통해 문자열의 길이를 제한했습니다.
@Email 어노테이션을 설정한 email 필드에서는 값이 '@' 문자가 있는지 확인합니다(실습에서는 이 정도로만 설정하지만
실무에서는 도메인을 검사하거나 비정상적인 이메일인지 검토하는 추가 설정이 필요할 수 있습니다). name은 @NotBlank로 null값이나
"", " " 모두 허용하지 않게 설정해서 값을 의무적으로 받도록 설정했습니다.
phoneNumber는 @Pattern을 통해 정규식을 설정했습니다. regexp 속성의 값을 "^01(?:0|1|[6-9])[.-]?(\\d{4})[.-]?(\\d{4})$"로
설정하면 휴대전화 번호 형식인지 검증할 수 있습니다.

Swagger에서 애플리케이션을 호출하면 별 문제없이 '200 OK'로 응답하는 것을 볼 수 있습니다.
이번에는 앞에서 설정한 규칙에서 벗어나는 값으로 변경해 호출해보겠습니다.
age를 -1로 설정하고 호출하면 다음과 같이 400 에러가 발생합니다.

```
Code	Details
400
Undocumented
Error: response status is 400

Response body
Download
{
  "timestamp": "2024-08-02T22:43:32.668+00:00",
  "status": 400,
  "error": "Bad Request",
  "path": "/validation/valid"
}
Response headers
 connection: close 
 content-type: application/json 
 date: Fri,02 Aug 2024 22:43:32 GMT 
 transfer-encoding: chunked
```

다음과 같이 애플리케이션의 로그에 로그가 출력되어 문제가 발생한 지점을 확인할 수 있습니다.
그러나 아직 별도의 예외 처리를 하지 않았기 때문에 에러가 발생했을 때 클라이언트 어디에서 에러가 발생했는지 확인할 수가 없습니다.

```
2024-08-03 07:43:32.655 [http-nio-8080-exec-2] WARN  o.s.w.s.m.s.DefaultHandlerExceptionResolver - Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public org.springframework.http.ResponseEntity<java.lang.String> com.springboot.valid_exception.data.controller.ValidationController.checkValidationByValid(com.springboot.valid_exception.data.dto.ValidatedRequestDto): [Field error in object 'validatedRequestDto' on field 'age': rejected value [-1]; codes [Min.validatedRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] ]
```

2개 이상의 유혀성 검사를 통과하지 못한 경우에는 다음과 같이 검사를 실패한 개수를 로그에 포함시킵니다.
이번에는 count의 값도 -1로 설정해서 유효성 검사를 실패하게 했습니다.

```
2024-08-03 07:46:16.659 [http-nio-8080-exec-1] WARN  o.s.w.s.m.s.DefaultHandlerExceptionResolver - Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public org.springframework.http.ResponseEntity<java.lang.String> com.springboot.valid_exception.data.controller.ValidationController.checkValidationByValid(com.springboot.valid_exception.data.dto.ValidatedRequestDto) with 2 errors: [Field error in object 'validatedRequestDto' on field 'age': rejected value [-1]; codes [Min.validatedRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] [Field error in object 'validatedRequestDto' on field 'count': rejected value [-1]; codes [Positive.validatedRequestDto.count,Positive.count,Positive.int,Positive]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.count,count]; arguments []; default message [count]]; default message [0보다 커야 합니다]] ]
```

출력 결과를 보면 'with 2 errors'라는 표현으로 두 요소에서 에러가 발생했다는 것이 명시돼 있습니다.
이후 내용에서는 각각 'default message'로 어느 부분이 잘못됐는지 더욱 쉽게 확인할 수 있습니다.

### 스터디 가이드
정규식(Regular Expression)이란 특정한 규칙을 가진 문자열 집합을 표현하기 위해 쓰이는 형식입니다. 전화번호, 주민등록번호,
이메일과 같이 특정 형식의 값을 검증해야 할 때가 있습니다.
이러한 값은 정규식을 통해 쉽게 검증할 수 있습니다.
정규식은 다음과 같은 요소를 사용합니다.
- ^: 문자열의 시작
- $: 문자열의 종료
- .: 임의의 한 문자
- *: 앞 문자가 없거나 무한정 많음
- +: 앞 문자가 하나 이상
- ?: 앞 문자가 없거나 하나 존재
- [,]: 문자의 집합이나 범위를 나타내며, 두 문자 사이는 - 기홀로 범위를 표현
- {,}: 횟수 또는 범위를 의미
- (,): 괄호 안의 문자를 하나의 문자로 인식
- |: 패턴 안에서 OR 연산을 수행
- \: 정규식에서 역슬래시는 확장문자로 취급하고, 역슬래시 다음에 특수문자가 오면 문자로 인식
- \b: 단어의 경계
- \B: 단어가 아닌 것에 대한 경계
- \A: 입력의 시작 부분
- \G: 이전 매치의 끝
- \Z: 종결자가 있는 경우 입력의 끝
- \z: 입력의 끝
- \s: 공백 문자
- \S: 공백 문자가 아닌 나머지 무자(^\s와 동일)
- \w: 알파벳이나 숫자
- \W: 알파벳이나 숫자나 아닌 문자(^\w와 동일)
- \d: 숫자 [0-9]와 동일하게 취급
- \D: 숫자를 제외한 모든 문자(^0-9와 동일)

정규식은 익숙하지 않은 문자의 조합으로 구성되기 때문에 다음과 같은 사이트에서 직접 정규식을 만들어보며
연습하는 것을 권장합니다.
- https://regexr.com/
- https:regex101.com/

# @Validated 활용
앞의 예제에서는 유효성 검사를 수행하기 위해 @Valid 어노테이션을 선언했습니다.
@Valid 어노테이션은 자바에서 지원하는 어노테이션이며, 스프링도 @Validated라는 별도의 어노테이션으로 유효성 검사를 지원합니다.
@Validated은 @Valid 어노테이션의 기능을 포함하고 있기 때문에 @Validated로 변경할 수 있습니다.
또한 @Validated 유효성 검사를 그룹으로 묶어 대상을 특정할 수 있는 기능이 있습니다.
검증 그룹은 별다른 내용이 없는 마커 인터페이스를 생성해서 사용합니다.
실습을 위해 다음과 같이 data 패키지 내에 group 패키지를 생성하고  ValidationGroup1과 ValidationGroup2라는
인터페이스를 생성합니다.
두 인터페이스 모두 내부 코드는 없으며, 인터페이스만 생성해서 그룹화하는 용도로 사용합니다.
각 인터페이스는 다음과 같이 구성돼 있습니다.

```java
package com.springboot.valid_exception.group;

public interface ValidationGroup1 {
}
```

```java
package com.springboot.valid_exception.group;

public interface ValidationGroup2 {
}
```

검증 그룹 설정은 DTO 객체에서 합니다.
다음과 같이 새로운 DTO 객체를 생성합니다.

```java
package com.springboot.valid_exception.data.dto;

import com.springboot.valid_exception.config.annotation.Telephone;
import com.springboot.valid_exception.group.ValidationGroup1;
import com.springboot.valid_exception.group.ValidationGroup2;
import jakarta.validation.constraints.*;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidatedRequestDto {
    @NotBlank
    private String name;

    @Email
    private String email;

    @Pattern(regexp = "^01(?:0|1|[6-9])[.-]?(\\d{4})[.-]?(\\d{4})$")
    private String phoneNumber;

    @Min(value = 20, groups = ValidationGroup1.class)
    @Max(value = 40, groups = ValidationGroup1.class)
    private int age;

    @Size(min = 0, max = 40)
    private String description;

    @Positive(groups = ValidationGroup2.class)
    private int count;

    @AssertTrue
    private boolean booleanCheck;
}
```

```
@Min(value = 20, groups = ValidationGroup1.class)
@Max(value = 40, groups = ValidationGroup1.class)
private int age;

@Positive(groups = ValidationGroup2.class)
private int count;
```

수정된 부분은 다음과 같습니다.
@Min, @Max어노테이션의 groups 속성을 사용해 ValidationGroup1 그룹을 설정하고 
@Positive(groups = ValidationGroup2.class)를 사용해 ValidationGroup2를 설정했습니다.
이 설정을 통해 어느 그룹에 맞춰 유효성 검사를 실시할 것인지 지정한 것입니다.
실제로 그룹을 어떻게 설정해서 유효성 검사를 실시할지 결정하는 것은 @Validated 어노테이션에서 합니다.
유효성 검사 그룹을 설정하기 위해 컨트롤러 클래스에 다음과 같이 메서드를 추가하겠습니다.


```java
package com.springboot.valid_exception.data.controller;

import com.springboot.valid_exception.data.dto.ValidatedRequestDto;
import com.springboot.valid_exception.group.ValidationGroup1;
import com.springboot.valid_exception.group.ValidationGroup2;
import jakarta.validation.Valid;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RequestMapping("/validation")
public class ValidationController {

    private final Logger LOGGER = LoggerFactory.getLogger(ValidationController.class);

    @PostMapping("/valided")
    public ResponseEntity<String> checkValidation(@Validated @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }

    @PostMapping("/valided/group1")
    public ResponseEntity<String> checkValidation1(@Validated(ValidationGroup1.class) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }

    @PostMapping("/valided/group2")
    public ResponseEntity<String> checkValidation2(
            @Validated(ValidationGroup2.class) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }

    @PostMapping("/valided/all-group")
    public ResponseEntity<String> checkValidation3(@Validated({ValidationGroup1.class, ValidationGroup2.class}) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }
}
```

다음 예제 에서는 @Validated 어노테이션에 속성을 지정하지 않고
public ResponseEntity<String> checkValidation(@Validated @RequestBody ValidatedRequestDto validatedRequestDto) 
@Validated(ValidationGroup2.class) @RequestBody ValidatedRequestDto validatedRequestDto) 여기서는 ValidationGroup2를 그룹으로 지정했습니다.
마지막으로 public ResponseEntity<String> checkValidation3(@Validated({ValidationGroup1.class, ValidationGroup2.class}) @RequestBody ValidatedRequestDto validatedRequestDto)
두 그룹을 모두 지정했습니다.

다시 Swagger 페이지에서 각 메서드를 호출하겠습니다.
먼저 checkValidation() 메서드를 호출하겠습니다. 호출할 때 전달하는 데이터는 다음과 같습니다.

```
{
  "name": "Flature",
  "email": "flature@wikibooks.co.kr",
  "phoneNumber": "010-1234-5678",
  "age": -1,
  "description": "Validation 실습 데이터입니다.",
  "count": -1,
  "booleanCheck": true
}
```

위 데이터는 age와 count 변수에 대한 유효성 검사를 통과하지 못하는 데이터입니다.
하지만 첫 번째 메서드를 호출했을 경우 정상적으로 통과하는 것을 볼 수 있습니다.
@Validated 어노테이션에 특정 그룹을 지정하지 않는 경우에는 groups 속성을 설정하지 않은 필드에 대해서만 유효성 검사를 실시하게 됩니다.

데이터는 그래도 이용하면서 checkValidation1() 메서드를 호출하면 다음과 같은 로그를 확인할 수 있습니다.

```
2024-08-03 08:14:40.049 [http-nio-8080-exec-3] WARN  o.s.w.s.m.s.DefaultHandlerExceptionResolver - Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public org.springframework.http.ResponseEntity<java.lang.String> com.springboot.valid_exception.data.controller.ValidationController.checkValidation(com.springboot.valid_exception.data.dto.ValidatedRequestDto) with 2 errors: [Field error in object 'validatedRequestDto' on field 'count': rejected value [-1]; codes [Positive.validatedRequestDto.count,Positive.count,Positive.int,Positive]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.count,count]; arguments []; default message [count]]; default message [0보다 커야 합니다]] [Field error in object 'validatedRequestDto' on field 'age': rejected value [-1]; codes [Min.validatedRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] ]
```

검사 오류가 발생할 수 있는 두 변수 중에서 ValidationGroup1을 그룹으로 설정한 age에 대한 에러가 발생하는 것을 볼 수 있습니다.
마찬가지로 checkValidation2() 메서드를 호출하면 age에서는 오류가 발생하지 않고 count에 대한 오류만 발생하게 됩니다.

그리고 마지막 메서드를 호출하면 다음과 같이 검사 오류 로그가 출력됩니다.
전 오류 로그는 나오지 않았습니다.

```
2024-08-03 08:20:12.996 [http-nio-8080-exec-7] INFO  c.s.v.d.c.ValidationController - ValidatedRequestDto(name=Flature, email=flature@wikibooks.co.kr, phoneNumber=010-1234-5678, age=-1, description=Validation 실습 데이터입니다., count=-1, booleanCheck=true)
```

한번더 메서드를 호출하겠습니다. 이번에는 호출 데이터를 다음과 같이 변경합니다.

```
{
  "name": "Flature",
  "email": "flature@wikibooks.co.kr",
  "phoneNumber": "010-1234-5678",
  "age": 30,
  "description": "Validation 실습 데이터입니다.",
  "count": 30,
  "booleanCheck": false
}
```

위 데이터는 age와 count는 검사를 통과하고 booleanCheck 변수에서 검사를 실패하는 데이터입니다.
한편 checkValidation3() 메서드를 호출하면 정상적으로 응답이 오는 것을 볼 수 있습니다.
정리하면 다음과 같습니다.

- @Validated 어노테이션에 특정 그룹을 설정하지 않은 경우에는 groups가 설정되지 않은 필드에 대해 유효성 검사를 수행
- @Validated 어노테이션에 특정 그룹을 설정하는 경우에는 지정된 그룹으로 설정된 필드에 대해서만 유효성 검사를 수행

이처럼 그룹을 지정해서 유효송 검사를 실시하는 경우에는 어떤 상황에 사용할지를 적절하게 설계해야 의도대로
유효성 검사를 실시할 수 있습니다.
만약 이를 제대로 설게하지 않으면 비효율적이거나 생산적이지 못한 패턴을 의미하는 안티 패턴이 발생하게 됩니다.


# 커스텀 Validation 추가
실무에서는 유효성 검사를 실시할 때 자바 또는 스프링의 유효성 검사 어노테이션에서 제공하지 않는 기능을 써야 할 때도 있습니다.
이 경우 ConstraintValidator와 커스텀 어노테이션을 조합해서 별도의 유효성 검사 어노테이션을 생성할 수 있습니다.
동일한 정규식을 계속 쓰는 @pattern 어노테이션의 경우가 가장 흔한 사례입니다.

이번에는 전화번호 형식이 일치하는지 확인하는 간단한 유효성 검사 어노테이션을 생성해 보겠습니다.
먼저 ConstraintValidator 인터페이스를 구현하는 클래스를 생성해야 합니다.
다음과 같이 TelephoneValidator 클래스를 생성합니다.

```java
package com.springboot.valid_exception.config.annotation;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class TelephoneValidator implements ConstraintValidator<Telephone, String> {

    public boolean isValid(String value, ConstraintValidatorContext context) {
        if(value == null){
            return false;
        }
        return value.matches("^01(?:0|1|[6-9])[.-]?(\\d{4})[.-]?(\\d{4})$");
    }
}
```

TelephoneValidator 클래스를 ConstraintValidator 인터페이스의 구현체로 정의합니다.
인터페이스를 선언할 때는 어떤 어노테이션 인터페이스인지 타입을 지정해야 합니다(어노테이션 인터페이스에 대해서는 바로
이어서 소개하겠습니다). ConstraintValidator 인터페이스는 invalid() 메서드를 정의하고 있습니다.
이 메서드를 구현하려면 직접 유효성 검사 로직을 작성해야 합니다.
null에 대한 허용 여부 로직을 추가하고, 다음줄에서는 지정한 정규식과 비교해서 알맞은 형식을 띠고 있는지 검사해야 합니다.
이 로직에서 false가 리턴되면 MethodArgumentNotValidException 예외가 발생합니다.

그럼 ConstraintValidator 인터페이스에서 정의한 Telephone 인터페이스르 살펴보겠습니다.
Telephone 인터페이스는 다음과 같이 작성할 수 있습니다.

```java
package com.springboot.valid_exception.config.annotation;

import jakarta.validation.Constraint;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = TelephoneValidator.class)
public @interface Telephone {
    String message() default "전화번호 형식이 일치하지 않습니다.";

    Class[] groups() default {};

    Class[] payload() default {};
}
```

@Target 어노테이션은 이 어노테이션을 어디서 선언할 수 있는지 정의하는데 사용됩니다.
예제에서는 필드에서 선언할 수 있게 설정돼 있습니다.
그 외에 사용할 수 있는 ElementType은 다음과 같습니다.
- ElementType.PACKAGE
- ElementType.TYPE
- ElementType.CONSTRUCTOR
- ElementType.FIELD
- ElementType.METHOD
- ElementType.ANNOTATION_TYPE
- ElementType.LOCAL_VARIABLE
- ElementType.PARAMETER
- ElementType.TYPE_PARAMETER
- ElementType.TYPE_USE

@Retention 어노테이션은 이 어노테이션이 실제로 적용되고 유지되는 범위를 의미합니다.
@Retention의 적용 범위는 RetentionPolicy를 통해 지정하며, 지정 가능한 항목은 다음과 같습니다.

- RetentionPolcicy.@Retention: 컴파일 이후에도 JVM에 의해 계속 참조합니다.
리플렉션이나 로깅에 많이 사용되는 정책입니다.
- RetentionPolcicy.SOURCE: 컴파일 전까지만 유지됩니다. 컴파일 이후에는 사라집니다.

@Constraint 어노테이션을 활용해 앞에서 소개한 TelephoneValidator와 매핑하는 작업을 수행합니다.
@Telephone 인터페이스 내부에는 message(), groups(), payload() 요소를 정의해야 합니다.
(이와 관련된 내용은 ConstraintHelper에서 살펴보겠습니다). 각 항목은 다음과 같은 의미를 가지고 있습니다.

- message(): 유효성 검사가 실패할 경우 반환되는 메시지를 의미합니다.
- groups(): 유효성 검사를 사용하는 그룹으로 설정합니다.
- payload(): 사용자가 추가 정보를 위해 전달하는 값입니다.

  별도의 groups()와 payload() 요소는 정의하지 않고 기본 메시지에 대해서만 요소를 설정했습니다.
  이렇게 어노테이션 인터페이스를 설정하고 나면 다음과 같이 인텔리제이 IDEA의 [Bean Validation] 탭에 앞에서
  생성한 @Telephone이 추가된 것을 볼 수 있습니다.

  이제 직접 생성한 새로운 유효성 검사 어노테이션을 적용해 보겠습니다.
  다음과 같이 ValidatedRequestDto 클래스에서 phoneNumber 변수의 어노테이션을 변경합니다.

```java
package com.springboot.valid_exception.data.dto;

import com.springboot.valid_exception.config.annotation.Telephone;
import com.springboot.valid_exception.group.ValidationGroup1;
import com.springboot.valid_exception.group.ValidationGroup2;
import jakarta.validation.constraints.*;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidatedRequestDto {
    
    @NotBlank
    private String name;

    @Email
    private String email;

    @Telephone
    private String phoneNumber;

    @Min(value = 20)
    @Max(value = 40)
    private int age;

    @Size(min = 0, max = 40)
    private String description;

    @Positive
    private int count;

    @AssertTrue
    private boolean booleanCheck;
}
```

@Pattern 어노테이션을 @Telephone 어노테이션으로 변경한 코드입니다.
다시 애플리케이션을 실행하고 Swagger 페이지에서 테스트를 진행하겠습니다.
테스트 데이터는 다음과 같습니다.

```
{
  "name": "Flature",
  "email": "flature@wikibooks.co.kr",
  "phoneNumber": "12345678",
  "age": 30,
  "description": "Validation 실습 데이터입니다.",
  "count": 30,
  "booleanCheck": false
}
```

이 내용으로 메서드를 호출하면 다음과 같이 유효성 검사에서 형식 오류를 감지하는 것을 볼 수 있습니다.
별도의 그룹을 지정하지 않았기 때문에 checkValidation() 메서드를 호출했을 때 오류가 발생합니다.

```
2024-08-03 08:50:47.768 [http-nio-8080-exec-7] WARN  o.s.w.s.m.s.DefaultHandlerExceptionResolver - Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public org.springframework.http.ResponseEntity<java.lang.String> com.springboot.valid_exception.data.controller.ValidationController.checkValidation(com.springboot.valid_exception.data.dto.ValidatedRequestDto) with 2 errors: [Field error in object 'validatedRequestDto' on field 'phoneNumber': rejected value [12345678]; codes [Telephone.validatedRequestDto.phoneNumber,Telephone.phoneNumber,Telephone.java.lang.String,Telephone]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.phoneNumber,phoneNumber]; arguments []; default message [phoneNumber]]; default message [전화번호 형식이 일치하지 않습니다.]] [Field error in object 'validatedRequestDto' on field 'booleanCheck': rejected value [false]; codes [AssertTrue.validatedRequestDto.booleanCheck,AssertTrue.booleanCheck,AssertTrue.boolean,AssertTrue]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.booleanCheck,booleanCheck]; arguments []; default message [booleanCheck]]; default message [true여야 합니다]] ]
```

# 예외 처리
