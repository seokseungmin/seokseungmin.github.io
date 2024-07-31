---
title: "스프링 핵심가이드 4주차"
excerpt: "[제로베이스] 스프링 핵심가이드"

categories:
  - Spring
tags:
  - [Spring]

permalink: /categories/Spring4Week/ # url

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

[스프링부트 핵심가이드 프로젝트 따라하면서 짠 코드](https://github.com/seokseungmin/advanced_jpa/tree/main)<br>

본 게시글은 '스프링 부트 핵심 가이드' 책의 내용을 정리한 것입니다.<br>
저자 : 장정우<br>
출판사 : 위키북스<br>

# 8장 Spring Data JPA

#### 프로젝트 생성
groupId: com.springboot  
artifactId: advanced_jpa

#### 의존성 선택
Lombok, Spring Configuration Processor, Spring Web, Spring Data JPA, MariaDB Driver, H2 Database, JDBC API
Swagger 의존성 추가:
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.2'

Swagger 설정파일, logback 설정파일 추가:
- config/SwaggerConfig.java
- logback-spring.xml

# JPQL
JPQL은 JPA Query Language의 줄임말로 JPA에서 사용할 수 있는 쿼리를 의미합니다.
JPQL의 문법은 SQL과 매우 비슷해서 데이터베이스 쿼리에 익숙한 분들이라면 어렵지 않게 사용할 수 있습니다.
SQL과의 차이점은 SQL에서는 테이블이나 칼럼의 이름을 사용하는 것과 달리 JPQL은 엔티티 객체를 대상으로 수행하는
쿼리이기 때문에 매핑된 엔티티의 이름과 필드의 이름을 사용한다는 것입니다.

SELECT p FROM Product p WHERE p.number = ?1;

Product = 엔티티 타입, p.number = 엔티티 속성

# 쿼리 메서드 살펴보기
리포지토리는 JpaRepository를 상속받는 것만으로도 다양한 CRUD 메서드를 제공합니다.
하지만 이러한 기본 메서드들은 식별자 기반으로 생성되기 때문에 결국 별도의 메서드를 정의해서
사용하는 경우가 많습니다.
이때 간단한 쿼리문을 작성하기 위해 사용되는 것이 쿼리 메서드입니다.

## 식별자란?
식별자는 데이터베이스 테이블에서 각 행을 고유하게 식별할 수 있는 컬럼을 의미합니다.
보통 기본 키(Primary Key)로 사용되며, 엔터티에서는 주로 @Id 어노테이션을 통해 식별자로 지정됩니다. 
식별자는 각 엔터티 인스턴스를 고유하게 식별할 수 있기 때문에 CRUD 메서드에서 자주 사용됩니다.

# 쿼리 메서드의 생성
쿼리 메서드는 크게 동작을 결정하는 주제(Subject)와 서술어(Predicate)로 구분합니다.
'find...By','exists...By'와 같은 키워드로 쿼리의 주제를 정하며 'By'는 서술어의 시작을 나타내는 구분자 역할을 합니다.
서술어 부분은 검색 및 정렬 조건을 지정하는 영역입니다.
기본적으로 엔티티의 속성으로 정의할 수 있고, AND나 OR를 사용해 조건을 확장하는 것도 가능합니다.

리포지토리의 쿼리 메서드 생성 예
// (리턴타입) + {주제 + 서술어(속성)} 구조의 메서드
List<Person> findByLastnameAndEmail(String lastName, String email);
서술어에 들어가는 엔티티의 속성 식(Expression)은 위의 예시와 같이 엔티티에서 관리하고 있는 속성(필드)만 참조할 수 있습니다.

쿼리 메서드의 주제 키워드
쿼리 메서드의 주제 부분에 사용할 수 있는 주요 키워드는 다음과 같습니다.
- find...By
- read...By
- get...By
- query...By
- search...By
- stream...By

보다시피 조회하는 기능을 수행하는 키워드입니다. '...'으로 표시한 영역에는 도메인(엔티티)을 표현할 수 있습니다.
그러나 리포지토리에서 이미 도메인을 설정한 후에 메서드를 사용하기 때문에 중복으로 판단해 생략하기도 합니다.
리턴 타입으로는 Collection이나 Stream에 속학 하위 타입을 설정할 수 있습니다.

find...By 키워드를 활용한 쿼리 메서드

```java
package com.springboot.advanced_jpa.data.repository;


import com.springboot.advanced_jpa.data.entity.Product;
import com.springboot.advanced_jpa.data.repository.support.ProductRepositoryCustom;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface ProductRepository extends JpaRepository<Product, Long>, ProductRepositoryCustom {
    List<Product> findByName(String name);

    List<Product> findByName(String name, Sort sort);

    Page<Product> findByName(String name, Pageable pageable);

    @Query("SELECT p FROM Product p WHERE p.name = :name")
    List<Product> findByNameParam(@Param("name") String name);

    @Query("SELECT p.name, p.price, p.stock FROM Product p WHERE p.name = :name")
    List<Object[]> findByNameParam2(@Param("name") String name);
}
```

# exists...By
특정 데이터가 존재하는지 확인하는 키워드입니다.
리턴 타입으로는 boolean 타입을 사용합니다

exists...By 키워드를 활용한 쿼리 메서드
boolean existsByNumber(Long number);

# count...By
조회 쿼리를 수행한 후 쿼리 결과로 나온 레코드의 개수를 리턴합니다.

count...By 키워드를 활용한 쿼리 메서드
//count...By
long countByName(String name);

# delete...By, remove...By
삭제 쿼리를 수행합니다. 리턴 타입이 없거나 삭제한 횟수를 리턴합니다.

delete와 remove를 활용한 쿼리 메서드
// delete...By, remove...By
void deleteByNumber(Long number);
long removeByName(String name);

# ...First(number)...,...TOP<number>...
쿼리를 통해 조회된 결과값의 개수를 제한하는 키워드입니다.
두 키워드는 동일한 동작을 수행하며, 주제와 By 사이에 위치합니다.
일반적으로 이 키워드는 한 번의 동작으로 여러 건을 조회할 때 사용되며,
단 건으로 조회하기 위해서는<number>를 생략하면 됩니다.

First,Top 키워드를 활용한 쿼리 메서드
// ...First(number)...,...TOP<number>...
List<Product> findFirst5ByName(String name);
List<Product> findTop10ByName(String name);

# 쿼리 메서드의 조건자 키워드
JPQL의 서술어 부분에서 사용할 수 있는 몇가지 조건자 키워드를 소개하겠습니다.

## Is
값의 일치를 조건으로 사용하는 조건자 키워드입니다.
생략되는 경우가 많으며 Equals와 동일한 기능을 수행합니다.

Is, Equals 키워드를 사용한 쿼리 메서드
//findByNumber 메소드와 동일하게 동작
Product findByNumberIs(Long number);
Product findByNumberEquals(Long number);

# (Is)Not
값의 불일치를 조건으로 사용하는 조건자 키워드입니다.
Is는 생략하고 Not 키워드만 사용할 수도 있습니다.

Not 키워드를 사용한 쿼리 메서드
Product findByNumberIsNot(Long number)
Product findByNumberNot(Long number)

# (Is)Null, (Is)NotNull
값이 null인지 검사하는 조건자 키워드입니다.

Null 키워드를 사용한 쿼리 메서드
List<Product> findByUpdatedAtNull();
List<Product> findByUpdatedAtIsNull();
List<Product> findByUpdatedAtNotNull();
List<Product> findByUpdatedAtIsNotNull();

# (Is)True, (Is)False
boolean 타입으로 지정된 칼럼값을 확인하는 키워드입니다.
Product 엔티티에 boolean타입을 사용하는 칼럼이 없기 때문에 실제 코드에 반영하면 에러가 발생하므로 사용법만 참고합니다.

True, False 키워드를 사용한 쿼리 메서드
Product findByisActiveTrue();
Product findByisActiveIsTrue();
Product findByisActiveFalse();
Product findByisActiveIsFalse();

# And, Or
여러 조건을 묶을 때 사용합니다.

And, Or 키워드를 사용한 쿼리 메서드
Prouct findByNumberAndName(Long number, String name);
Product findByNumberOrName(Long number, String name);

# (Is)GreaterThan, (Is)LessThan, (Is)Between
숫자나 datetime 칼럼을 대상으로 한 비교 연산에 사용할 수 있는 조건자 키워드입니다.
GreaterThan, LessThan 키워드는 비교 대상에 대한 초과/미만의 개념으로 비교 연산을 수행하고, 경곗값을 포함하려면
Equal 키워드를 추가하면 됩니다.

GreaterThan, LessThan, Between 키워드를 사용한 쿼리 메서드
List<Product> findByPriceIsGreaterThan(Long price);
List<Product> findByPriceGreaterThan(Long price);
List<Product> findByPriceIsGreaterThanEqual(Long price);
List<Product> findByPriceIsLessThan(Long price);
List<Product> findByPriceLessThan(Long price);
List<Product> findByPriceIsLessThanEqual(Long price);
List<Product> findByPriceIsBetWeen(Long price);
List<Product> findByPriceBetWeen(Long price);

# (Is)StartingWith(=StartsWith), (Is)EndingWith(=EndsWith), (Is)containing(=Contains), (Is)Like
칼럼값에서 일부 일치 여부를 확인하는 조건자 키워드입니다.
SQL 쿼리문에서 값의 일부를 포함하는 값을 추출할 떄 사용하는 '%' 키워드와 동일한 역할을 하는 키워드입니다.
자동으로 생성되는 SQL문을 보면 Containing 키워드는 문자열의 양 끝, StartingWith 키워드는 문자열의 앞,
EndingWith 키워드는 문자열의 끝에 '%'가 배치됩니다. 여기에 별도로 고려해야 하는 키워드는 Like 키워드인데,
이 키워드는 코드 수준에서 메서드를 호출하면서 전달하는 값에 %를 명시적으로 입력해야 합니다.

부분 일치 키워드를 사용한 쿼리 메서드
List<Product> findByNameLike(String name);
List<Product> findByNameIsLike(String name);
List<Product> findByNameContains(Stirng name);
List<Product> findByNameContaining(Stirng name);
List<Product> findByNameIsContains(Stirng name);

List<Product> findByNameStartsWith(Stirng name);
List<Product> findByNameStartingWith(Stirng name);
List<Product> findByNameIsStartingWith(Stirng name);

List<Product> findByNameEndsWith(Stirng name);
List<Product> findByNameEndingWith(Stirng name);
List<Product> findByNameIsEndingWith(Stirng name);

# 정렬과 페이징 처리
