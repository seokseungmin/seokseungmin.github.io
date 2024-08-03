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
애플리케이션을 개발할 때는 불가피하게 많은 오류가 발생하게 됩니다.
자바에서는 이러한 오류를 try/catch, throw 구문을 활용해 처리합니다.
스플링부트에서는 더욱 편리하게 예외처리를 할 수 있는
기능을 제공합니다. 이번 절에서는 예외 처리의 기초를 소개하고 스프링 부트에서 적용할 수 있는 예외 처리 방식을 알아보겠습니다.

### 예외와 에러
프로그래밍에서 예외(exception)란 입력 값의 처리가 불가능하거나 참조된 값이 잘못된 경우 등 애플리케이션이 정상적으로 동작하지 못하는 상황을 의미합니다.
예외는 개발자가 직접 처리할 수 있는 것이므로 미리 코드 설계를 통해 처리할 수 있습니다.
다음으로 에러(error)가 있습니다.
많은 사람들이 예외와 비슷한 의미로 사용하고 있지만 소프트웨어 공학에서는 엄연히 다르게 사용되는 용어입니다.
에러는 주로 자바의 가상머신에서 발생시키는 것으로서 예외와 달리 애플리케이션 코드에서 처리할 수 있는 것이
거의 없습니다.
대표적인 예로 메모리 부족(OutOfMemory), 스택 오버플로(StackOverFlow) 등이 있습니다.
이러한 에러는 발생 시점에 처리하는 것이 아니라 미리 애플리케이션의 코드를 살펴보면서 문제가 발생하지 않도록 예방해서 원천적으로 차단해야 합니다.

# 예외 클래스
자바의 예외 클래스는 다음과 같은 상속 구조를 갖추고 있습니다.

- java.lang.Object
  - Throwable
    - Exception (Checked Exception)
      - IOException
      - SQLException
      - CustomException
      - ... (기타 Checked Exception들)
    - RuntimeException (Unchecked Exception)
      - NullPointerException
      - IllegalArgumentException
      - IndexOutOfBoundException
      - ... (기타 Unchecked Exception들)
    - Error (Unchecked Exception)
      - IOError 
      - OutOfMemoryError
      - StackOverflowError
      - ... (기타 Error들)


모든 예외 클래스는 Throwable 클래스를 상속받습니다.
그리고 가장 익숙하게 볼 수 있는 Exception 클래스는 다양한 자식 클래스를 가지고 있습니다.
이 클래스는 크게 Checked Exception과 Unchecked Exception으로 구분할 수 있습니다.

| 구분               | Checked Exception       | Unchecked Exception                                     |
|------------------|------------------------|---------------------------------------------------------|
| 처리여부          | 반드시 예외 처리 필요      | 명시적 처리를 강제하지 않음                               |
| 확인시점          | 컴파일 단계              | 실행 중 단계                                           |
| 대표적인 예외클래스 | IOException, SQLException | RuntimeException, NullPointerException, IllegalArgumentException, IndexOutOfBoundException, SystemException |

Checked Exception은 컴파일 단계에서 확인 가능한 예외 상황입니다.
이러한 예외는 IDE에서 캐치해서 반드시 예외 처리를 할 수 있게 표시해줍니다.
반면 Unchecked Exception은 런타임 단계에서 확인되는 예외 상황을 나타냅니다.
즉, 문법상 문제는 없지만 프로그램이 동작하는 도중 예기치 않은 상황이 생겨 발생하는 예외를 의미합니다.

간단히 분류하자면 RuntimeException을 상속받는 Exception 클래스는 Unchecked Exception이고그렇지 않은 Exception 클래스는 Checked Exception입니다.

# 예외 처리 방법
예외가 발생했을 때 이를 처리하는 방법은 크게 세 가지가 있습니다.
- 예외 복구
- 예외 처리 회피
- 예외 전환

먼저 예외 복구 방법은 에외 상황을 파악해서 문제를 해결하는 방식입니다.
대표적인 방법이 try/catch구문입니다. try블록에는 예외가 발생할 수 있는 코드를 작성합니다.
대체로 외부 라이브러리를 사용하는 경우에는 try 블록을 사용하라는 IDE의 알림이 발생하지만 개발자가 직접
작성한 로직은 예외 상황을 예측해서 try 블록에 포함시켜야 합니다.
그리고 catch 블록을 통해 try 블록에서 발생하는 예외 상황을 처리하는 내용을 작성합니다.
이때 catch 블록은 여러 개를 작성할수 있습니다.
이 경우 예외상황이 발생하면 애플리케이션에서는 여러 개의 catch 블록을 순차적으로 거치면서 예외 유형과 매칭되는
블록을 찾아 예외 처리 동작을 수행합니다.

```java
int a = 1;
String b = "a";

try{
  System.out.println(a + Integer.parseInt(b));
} catch (NumberFormatException e){
  b = "2";
  System.out.println(a + Integer.parseInt(b));
}
```

또 다른 예외 처리 방법 중 하나는 예외처리를 회피하는 방법입니다.
이 방법은 예외가 발생한 시점에서 바로 처리하는 것이 아니라 에외가 발생한 메서드를 호출한 곳에서 처리 할 수 있게
전가하는 방식입니다.
이때 throw 키워드를 사용헤 어떤 예외가 발생했는지 호출부에 내용을 전달할 수 있습니다.


```java
int a = 1;
String b = "a";

try{
  System.out.println(a + Integer.parseInt(b));
} catch (NumberFormatException e){
  throw new NumberFormatException("숫자가 아닙니다.");
}
```

마지막으로 예외 전환 방법이 있습니다. 이 방법은 앞의 두 방식을 적절하게 섞은 방식입니다.
예외가 발생헀을 때 어떤 예외가 발생했느냐에 따라 호출부로 예외 내용을 전달하면서 좀 더 적합한
예외 타입으로 전달할 필요가 있습니다.
또는 애플리케이션에서 예외 처리를 좀 더 단순하게 하기 위해 래핑(Wrapping)해야 하는 경우도 있습니다.
이런 경우에는 try/catch 방식을 사용하면서 catch블록에서 throw 키워드를 사용해 다른 예외 타입으로 전달하면 됩니다.
이 방식은 앞으로 나올 커스텀 예외를 만드는 과정에서 사용되는 방법이므로 별도로 예제를 보여드리지 않겠습니다.

# 스프링 부트의 예외 처리 방식
웹 서비스 애플리케이션에서는 외부에서 들어오는 요청에 담긴 데이터 처리하는 경우가 많습니다.
그 과정에서 예외가 발생하면 예외를 복구해서 정상으로 처리하기보다는 요청을 보낸 클라이언트에 어떤 문제가 발생했는지 상황을 전달하는 경우가 많습니다.
이번 절에서는 이를 반영해서 에외 상황을 복구하는 방법보다는 스프링 부트에서 사용하는 예외 처리 방법을 중심으로
설명하고 실습하겠습니다.

예외가 발생했을 때 클라이언트 오류 메시지를 전달하려면 각 레이어에서 발생한 예외를 엔드포인트 레벨인
컨트롤러로 전달해야 합니다. 이렇게 전달받은 예외를 스프링 부트에서 처리하는 방식으로 크게 두가지가 있습니다.

- @(Rest)ControllerAdvice와 @ExceptionHandler를 통해 모든 컨트롤러의 예외를 처리
- @ExceptionHandler를 통해 특정 컨트롤러의 예외를 처리

# Tip
@ControllerAdvice 대신 @RestControllerAdvice를 사용하면 결괏값을 JSON 형태로 반환할 수 있습니다.
먼저 @RestControllerAdvice를 활용한 핸들러 클래스를 생성하겠습니다.
다음과 같이 CustomExceptionHandler 클래스를 생성합니다.

```
package com.springboot.valid_exception.common.exception;

import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class CustomExceptionHandler {

    private final Logger LOGGER = LoggerFactory.getLogger(CustomExceptionHandler.class);

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<Map<String, String>> handleException(RuntimeException e, HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;

        LOGGER.error("Advice 내 handlerException 호출, {},{}", request.getRequestURI(), e.getMessage());

        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }

    @ExceptionHandler(CustomException.class)
    public ResponseEntity<Map<String, String>>handleException(CustomException e, HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();

        LOGGER.error("Advice 내 handleException 호출, {},{}", request.getRequestURI(), e.getMessage());

         Map<String, String> map = new HashMap<>();
         map.put("error type", e.getHttpStatusType());
         map.put("code", Integer.toString(e.getHttpCode()));
         map.put("message", e.getMessage());

         return new ResponseEntity<>(map, responseHeaders, e.getHttpStatus());
    }
}
```

에제에서 사용한 @RestControllerAdvice와 이 예제에서는 사용하지 않은 @ControllerAdvice는 스프링에서 제공하는
어노테이션입니다. 이 어노테이션은 @Controller나 @RestController에서 발생하는 예외를 한 곳에서 관리하고
처리할 수 있게 하는 기능을 수행합니다.
즉, 다음과 같이 별도 설정을 통해 예외를 관제하는 범위를 지정할 수 있습니다.
@RestControllerAdvice(basePackages = "com.springboot.valid_exception")

지정된 @ExceptionHandler는 @Controller나 @RestController가 적용된 빈에서 발생하는 예외를 잡아 처리하는 메서드를 정의할 때 사용합니다. 어떤 예외 클래스를 처리할지는 value 속성으로 등록합니다.
value 속성은 배열의 형식으로도 전달받을 수 있어 여러 예외 클래스를 등록할 수도 있습니다.
위 예제에서는 RuntimeException이 발생하면 처리하도록 코드를 작성했으므로 RuntimeException에 포함되는
각종 예외가 발생할 경우를 포착해서 처리하게 됩니다.

클라이언트에게 오류가 발생했다는 것을 알리는 응답 메시지를 구성해서 리턴합니다.
컨트롤러의 메서드에 다른 타입의 리턴이 설정돼 있어도 핸들러 메서드에서 별도의 리턴 타입을 지정할 수 있습니다.

이 예제를 테스틀하기 위해 예외를 발생시킬 수 있는 컨트롤러를 생성하겠습니다.
다음과 같이 ExceptionController를 생성합니다.

```java
package com.springboot.valid_exception.data.controller;

import com.springboot.valid_exception.common.Constants;
import com.springboot.valid_exception.common.exception.CustomException;
import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/exception")
public class ExceptionController {

    private final Logger LOGGER = LoggerFactory.getLogger(ExceptionController.class);

    @GetMapping
    public void getRuntimeException() {
        throw new RuntimeException("getRuntimeException 메서드 호출");
    }

    @ExceptionHandler(value = RuntimeException.class)
    public ResponseEntity<Map<String, String>> handleException(RuntimeException e, HttpServletRequest request) {
        HttpHeaders responseHeader = new HttpHeaders();
        responseHeader.setContentType(MediaType.APPLICATION_JSON);
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;

        LOGGER.error("클래스 내 handleException 호출, {}, {}", request.getRequestURI(), e.getMessage());

        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeader, httpStatus);
    }

    @GetMapping("/custom")
    public void getCustomException() throws CustomException {
        throw new CustomException(Constants.ExceptionClass.PRODUCT, HttpStatus.BAD_REQUEST, "getCustomException 메서드 호출");
    }
}
```

위 예제의 getRuntimeException() 메서드는 컨트롤러로 요청이 들어오면 RuntimeException을 발생시킵니다.
다음과 같이 Swagger 페이지에서 이 메서드를 호출해 봅시다.

호출하면 400 에러와 함께 다음과 같이 에러 메시지가 Body 값에 담겨 응답이 돌아옵니다.

```
Code	Details
400
Undocumented
Error: response status is 400

Response body
Download
{
  "code": "400",
  "error type": "Bad Request",
  "message": "getRuntimeException 메서드 호출"
}
Response headers
 connection: close 
 content-type: application/json 
 date: Sat,03 Aug 2024 06:57:01 GMT 
 transfer-encoding: chunked
```

핸들러 메서드는 Map 객체에 응답할 메시지를 구성하고 ResponseEntity에 HttpHeader, HttpStatus, Body값을 담아
전달합니다.
이 처럼 컨트롤러에서 던진 예외는 @ControllerAdvice 또는 @RestControllerAdvice가 선언돼 있는 핸들러 클래스에서
매핑된 예외 타입을 찾아 처리하게 됩니다.
두 어노테이션은 별도 범위 설정이 없으면 전역 범위에서 예외를 처리하기 때문에 특정 컨트롤러에서만 동작하는
@ExceptionHandler 메서드를 생성해서 처리할 수도 있습니다.
ExceptionController에 다음과 같이 메서드를 추가로 생성해 봅시다.
위 코드는 이미 추가된 상태입니다. 그래서 생략하겠습니다.

컨트롤러 클래스 내에 @ExceptionHandler 어노테이션을 사용한 메서드를 선언하면 해당 클래스에 국한해서 예외 처리를 할 수 있습니다. 핸들러 메서드에서 각 로그 메시지에 다음과 같은 내용이 콘솔에 출력되는 것을 볼 수 있습니다.

```
2024-08-03 16:03:43.796 [http-nio-8080-exec-6] ERROR c.s.v.d.c.ExceptionController - 클래스 내 handleException 호출, /exception, getRuntimeException 메서드 호출
```

출력 결과에서 '클래스 내 handleException 호출'이라는 메시지를 볼 수 있습니다. 만약 `@ControllerAdvice`와 컨트롤러 내에 동일한 예외 타입을 처리한다면 좀 더 우선순위가 높은 클래스 내의 핸들러 메서드가 사용되는 것을 볼 수 있습니다. 우선순위를 비교하는 방법은 총 두 가지가 있습니다.

### @ControllerAdvice vs @RestControllerAdvice

```plaintext
@ControllerAdvice                                      @RestControllerAdvice
---------------------------------------------------    ---------------------------------------------------
|                                                   |    |                                               |
| - @ExceptionHandler(Exception.class) < (우선순위 높음) |    | - @ExceptionHandler(NullPointerException.class) |
|                                                   |    |                                               |
---------------------------------------------------    ---------------------------------------------------
```

### 구체적인 예외 클래스 우선순위
만약 컨트롤러 또는 `@ControllerAdvice` 클래스 내에 동일하게 핸들러 메서드가 선언된 상태에서 다음과 같이 `Exception` 클래스와 그보다 좀 더 구체적인 `NullPointerException` 클래스가 각각 선언된 경우에는 구체적인 클래스가 지정된 쪽이 우선순위를 갖게 됩니다.

### 글로벌 예외 처리 vs 컨트롤러 예외 처리
`@ControllerAdvice`의 글로벌 예외 처리와 컨트롤러 내의 예외 처리에 동일한 타입의 예외를 처리하게 되면 범위가 좁은 컨트롤러의 핸들러 메서드가 우선순위를 갖게 됩니다.

```plaintext
@ControllerAdvice                                      @Controller
---------------------------------------------------    ---------------------------------------------------
|                                                   |    |                                               |
| - @ExceptionHandler(Exception.class) < (우선순위 높음) |    | - @ExceptionHandler(Exception.class)           |
|                                                   |    |                                               |
---------------------------------------------------    ---------------------------------------------------
 글로벌 예외처리                                       컨트롤러 예외 처리
```

따라서, 예외 처리 우선순위를 결정할 때는 예외 클래스의 구체성과 핸들러 메서드의 위치를 고려해야 합니다.

# 커스텀 예외
애플리케이션을 개발하다 보면 점점 예외로 처리할 영역이 늘어나고, 예외 상황이 다양해지면서 사용하는
예외 타입도 많아집니다.
대부분의 상황에서는 자바에서 이미 적절한 상황에 사용할 수 있도록 제공하는 표준 예외(Standard Exception)를 사용하면 해결됩니다.
사실 애플리케이션의 예외 처리에는 표준 예외만 사용해도 모든 상활들을 처리할 수 있습니다.
그런데 왜 커스텀 예외(Custom Exception)를 만들어 사용할까요?

커스텀 예외를 만들어서 사용하면 네이밍에 개발자의 의도를 담을 수 있기 때문에 이름만으로도 어느 정도 예외 상황을 짐작할 수 있습니다.
앞에서 언급했듯이 표준 예외에서도 다양한 예외 상황을 처리할 수 있는 클래스를 제공하고 있지만
표준 예외에서 제공하는 클래스는 해당 예외 타입의 이름만으로 이해하기 어려운 경우가 있습니다.
그래서 표준 예외를 사용할 때는 예외 메시지를 상세히 작성해야 하는 번거로움이 있습니다.

또한 커스텀 예외를 사용하면 애플리케이션에서 발생하는 예외를 개발자가 직접 관리하기가 수월해집니다.
표준 에외를 상속받은 커스텀 예외들을 개발자가 직접 코드로 관리하기 때문에 책임 소재를 애플리케이션
내부로 가져올 수 있게 됩니다.
마지막으로 커스텀 예외를 사용하면 예외 상황에 대한 처리도 용이합니다.
앞에서 @ControllerAdvice와 @ExceptionHandler에 대해 알아봤는데, 이러한 어노테이션을 사용해
애플리케이션에서 발생하는 예외 상황들을 한 곳에서 처리할 수 있었습니다.
예를 들어, RuntimeException에 대해 @ControllerAdvice의 내부에서 표준 예외 처리를 하는 로직을 
작성한 경우 개발자가 의도한 RuntimeException 부분이 아닌 의도하지 않은 부분에서
발생하는 에러들이 존재할 수 있습니다.
표준 예외를 사용하면 이처럼 의도하지 않은 예외 상황도 정해진 예외 처리 코드에서 처리하기 때문에
어디에서 문제가 발생했는지 확인하기가 어렵습니다.
그러나 커스텀 예외로 관리하면 의도하지 않았던 부분에서 발생한 예외는 개발자가 관리하는 예외 처리 코드가 처리하지 않으므로 개발 과정에서 혼동할 여지가 줄어듭니다.

### 스터디 가이드
커스텀 예외의 효과에 대해서는 개발자들의 의견이 분분합니다. 우선 커스텀 예외를 만들어 사용해보는 것을 시작으로
어떤 방식이 효과적인지 직접 고민하고 자신만의 논리를 구축하길 바랍니다.
