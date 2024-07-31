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
애플리케이션에서 자주 사용되는 정렬과 페이징 처리는 앞서 소개한 쿼리 메서드를 작성하는 방법을 기반으로 수행할 수 있습니다. 물론 기본 쿼리 메서드인 이름를 통한 정렬과 페이징 처리도 가능하지만 다른 방법들도 많이 쓰입니다. 이번 장에서는 기본적인 정렬과 페이징 처리 방법을 알아보겠습니다.

## 정렬 처리하기
일반적인 쿼리문에서 정렬을 사용할 때는 OREDER BY 구문을 사용합니다.
쿼리 메서드도 정렬 기능에 동일한 키워드가 사용됩니다.

쿼리 메서드의 정렬 처리
// Asc : 오름차순, Desc : 내림차순
List<Product> findByNameOrderByNumberAsc(String name);
List<Product> findByNameOrderByNumberDesc(String name);

기본 쿼리 메서드를 작성한 후 OrderBy 키워드를 삽입하고 정렬하고자 하는 칼럼과 오름차순/내림차순을 설정하면 정렬이 수행됩니다.
2번 줄의 쿼리 메서드를 해석하면 '상품정보를 이름으로 검색한 후 상품 번호로 오름차순 정렬을 수행' 한다는 뜻입니다. 오름차순으로 정렬하려면 Asc 키워드를 내림차순으로 정렬하려면 Desc 키워드를 사용합니다.

하이버네이트 로그를 살펴보면 order by 구문이 포함돼 있고 메서드에 이름에 나와 있는 것처럼 Number에 대해 오름차순으로 정렬하고 있습니다.
다른 쿼리 메서드들은 조건 구문에서 조건을 여러 개 사용하기 위해 And와 Or 키워드를 사용했습니다.
하지만 정렬 구문은 And나 Or 키워드를 사용하지 않고 우선순위를 기주으로 차례대로 작성하면 됩니다.

쿼리 메서드에서 여러 정렬 기준 사용
// And를 붙이지 않음
List<Product> findByNameOrderByPriceAscStockDesc(String name);
먼저 Price 기준으로 오름차순 정렬한 후 후순위로 재고수량을 기준으로 내림차순 정렬을 수행합니다. 이렇게 작성한 메서드가 호출되면 하이버네이트에서는 다음과 같이 쿼리를 작성합니다.
이미지 생략...
이렇게 쿼리 메서드의 이름에 정렬 키워드를 삽입해서 정렬을 수행하는 것도 가능하지만 메소드의 이름이 길어질수록 가독성이 떨어지는 문제가 생깁니다.
이를 해결하기 위해 매개변수를 활용해 정렬할 수도 있습니다.

매개변수를 활용한 쿼리 정렬
List<Product> findByName(String name, Sort sort);
앞서 소개한 정렬 키워드가 들어간 메서드와 거의 동일한 기능을 수행합니다.
다만 이 메서드는 이름에 키워드를 넣지 않고 Sort객체를 활용해 매개변수로 받아들인
정렬 기준을 가지고 쿼리문을 작성하게 됩니다.
Sort 객체를 테스트해보기 위해 다음과 같은 패키지를 생성한후 ProducRepositoryTest를 생성해서 다음과 같이 작성합니다.

```java
package com.springboot.advanced_jpa.data.repository;

import com.querydsl.core.Tuple;
import com.querydsl.jpa.impl.JPAQuery;
import com.querydsl.jpa.impl.JPAQueryFactory;
import com.springboot.advanced_jpa.config.QueryDSLConfig;
import com.springboot.advanced_jpa.data.entity.Product;
import com.springboot.advanced_jpa.data.entity.QProduct;
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.context.annotation.Import;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;

import java.time.LocalDateTime;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase.Replace;


@DataJpaTest
// 기본값은  Replace.ANY, 이 경우 임베디드 메모리 데이터베이스를 사용.
//  Replace.NONE으로 변경하면 애플리케이션에서 실제로 사용하는 데이터베이스로 테스트가 가능.
@AutoConfigureTestDatabase(replace = Replace.NONE)
@Import(QueryDSLConfig.class)
public class ProductRepositoryTest {

    @PersistenceContext
    EntityManager entityManager;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private JPAQueryFactory jpaQueryFactory;

    @Test
    void save() {
        //given
        Product product = new Product();
        product.setName("펜");
        product.setPrice(1000);
        product.setStock(1000);

        //when
        Product savedProduct = productRepository.save(product);

        //then
        assertEquals(product.getName(), savedProduct.getName());
        assertEquals(product.getPrice(), savedProduct.getPrice());
        assertEquals(product.getStock(), savedProduct.getStock());
    }

    @Test
    void sortingAndPagingTest() {
        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(1000);
        product1.setStock(100);
        product1.setCreatedAt(LocalDateTime.now());
        product1.setUpdatedAt(LocalDateTime.now());

        Product product2 = new Product();
        product2.setName("펜");
        product2.setPrice(5000);
        product2.setStock(300);
        product2.setCreatedAt(LocalDateTime.now());
        product2.setUpdatedAt(LocalDateTime.now());

        Product product3 = new Product();
        product3.setName("펜");
        product3.setPrice(500);
        product3.setStock(50);
        product3.setCreatedAt(LocalDateTime.now());
        product3.setUpdatedAt(LocalDateTime.now());

        productRepository.save(product1);
        productRepository.save(product2);
        productRepository.save(product3);

        System.out.println(productRepository.findByName("펜", getSort()));

        Page<Product> productPage = productRepository.findByName("펜", PageRequest.of(0, 2));
        System.out.println(productPage.getContent());


        List<Product> sortedProductsByParam = productRepository.findByNameParam("펜");
        System.out.println("sortedProductsByParam : " + sortedProductsByParam);

        List<Object[]> sortedProductsByParam2 = productRepository.findByNameParam2("펜");
        System.out.println("sortedProductsByParam2 : " + sortedProductsByParam2);
    }

    private Sort getSort() {
        return Sort.by(Sort.Order.asc("price"), Sort.Order.desc("stock"));
    }

    @Test
    void queryDslTest() {
        //given
        JPAQuery<Product> query = new JPAQuery<>(entityManager);
        QProduct qProduct = QProduct.product;

        List<Product> productList = query
                .from(qProduct)
                .where(qProduct.name.eq("펜"))
                .orderBy(qProduct.price.asc())
                .fetch();

        for (Product product : productList) {
            System.out.println("---------------");
            System.out.println("Product Number : " + product.getNumber());
            System.out.println("Product Name : " + product.getName());
            System.out.println("Product Price : " + product.getPrice());
            System.out.println("Product Stock : " + product.getStock());
            System.out.println();
            System.out.println("---------------");
        }
    }

    @Test
    void queryDslTest2() {
        //given
        JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(entityManager);
        QProduct qProduct = QProduct.product;

        List<Product> productList = jpaQueryFactory.selectFrom(qProduct)
                .where(qProduct.name.eq("펜"))
                .orderBy(qProduct.price.asc())
                .fetch();

        for (Product product : productList) {
            System.out.println("---------------");
            System.out.println();
            System.out.println("Product Number : " + product.getNumber());
            System.out.println("Product Name : " + product.getName());
            System.out.println("Product Price : " + product.getPrice());
            System.out.println("Product Stock : " + product.getStock());
            System.out.println();
            System.out.println("---------------");
        }
    }

    @Test
    void queryDslTest3() {
        //given
        JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(entityManager);
        QProduct qProduct = QProduct.product;

        List<String> productList = jpaQueryFactory
                .select(qProduct.name)
                .from(qProduct)
                .where(qProduct.name.eq("펜"))
                .orderBy(qProduct.price.asc())
                .fetch();

        for (String product : productList) {
            System.out.println("---------------");
            System.out.println();
            System.out.println("Product Name : " + product);
            System.out.println("---------------");

            List<Tuple> tupleList = jpaQueryFactory
                    .select(qProduct.name, qProduct.price)
                    .from(qProduct)
                    .where(qProduct.name.eq("펜"))
                    .orderBy(qProduct.price.asc())
                    .fetch();

            for (Tuple tuple : tupleList) {
                System.out.println("---------------");
                System.out.println("Product Name : " + tuple.get(qProduct.name));
                System.out.println("Product Price : " + tuple.get(qProduct.price));
                System.out.println("---------------");
            }
        }
    }

    @Test
    void queryDslTest4() {
        //given
        QProduct qProduct = QProduct.product;

        List<String> productList = jpaQueryFactory
                .select(qProduct.name)
                .from(qProduct)
                .where(qProduct.name.eq("펜"))
                .orderBy(qProduct.price.asc())
                .fetch();

        for (String product : productList) {
            System.out.println("---------------");
            System.out.println("Product Nane : " + product);
            System.out.println("---------------");
        }
    }

    @Test
    void findByNameTest() {
        List<Product> productList = productRepository.findByName("펜");

        for (Product product : productList) {
            System.out.println(product.getNumber());
            System.out.println(product.getName());
            System.out.println(product.getPrice());
            System.out.println(product.getStock());
        }
    }

    @Test
    void auditingTest() {
        Product product = new Product();
        product.setName("펜");
        product.setPrice(1000);
        product.setStock(100);

        Product savedProduct = productRepository.save(product);

        System.out.println("productName : " + savedProduct.getName());
        System.out.println("createdAt : " + savedProduct.getCreatedAt());
    }
}
```

sortingAndPagingTest() 메서드에 밑에 다음과 같이 작성해서 메소드에 전달할 수 있습니다.

쿼리 메서드에 Sort 객체 전달
productRepository.findByName("펜", Sort.by(Order.asc("price)));
productRepository.findByName("펜", Sort.by(Order.asc("price), Order.desc("stock)));

위 예제를 포함하여 앞으로 나올 테스트 코드의 결과값을 확인하고 싶다면
System.out.println()을 붙여주면 됩니다.
Sort 클래스는 내부 클래스로 정의돼 있는 Order 객체를 활용해 정렬 기준을 생성합니다.
Order 객체에는 asc와 desc 메서드가 포함돼 있어 이 메서드를 통해 오름차순/내림차순을 지정하며,
여러정렬 기준을 사용할 경우에는 2번 줄처럼 콤마(,)를 사용해 구분합니다.
이렇게 작성된 두 개의 메서드를 실행하면 하이버네이트에서는 다음과 같은 쿼리를 생성합니다.
쿼리 내용은 생략...

매개변수를 활용한 쿼리 메서드를 사용하면 쿼리 메서드를 정의하는 단계에서 코드가 줄어드는 장점이 있습니다. 그러나 호출하는 위치에서는 여전히 정렬 기준이 길어져 가독성이 떨어집니다.
해당 코드는 정렬 기준을 설정하기 위한 필수적인 구문이기 때문에 코드의 양을 줄이기는 어렵습니다.
하지만 Sort 부분을 하나의 메서드로 분리해서 쿼리 메서드를 호출하는 코드를 작성하는 방법도 가능합니다.
위에 올린 코드가 이 작업을 이미 처리한 코드입니다.

# 페이징 처리
페이징이란 데이터베이스의 레코드를 개수로 나눠 페이지를 구분하는 것을 의미합니다.
예를 들면, 25개의 레코드가 있다면 레코드를 7개씩, 총 4개의 페이지로 구분하고 그중에서 특정 페이지를 가져오는 것입니다. 흔히 볼 수 있는 웹 페이지에서 각 페이지를 구분해서 데이터를 제공할 때
그에 맞게 데이터를 요청하는 것이라고 생각하면 됩니다.
JPA에서는 이 같은 페이징 처리를 위해 Page와 Pageable을 사용합니다.
페이징 처리가 가능한 쿼리 메서드를 작성할 수 있습니다.

페이징 처리를 위한 쿼리 메서드 예시
Page<Product> findByName(String name, Pageable pageable);

페이징 쿼리 메서드를 호출하는 방법
Page<Product> productPage = productRepository.findByName("펜", PageRequest.of(0,2));

위 코드에서는 메서드를 호출할 때 리턴 타입으로 Page 객체를 받아야 하기 때문에 Page<Product>로 타입을 
선언해서 객체를 리턴받았습니다.
그리고 Pageable 파라미터를 전달하기 위해 PageRequest 클래스를 사용했습니다.
PageRequest는 Pageable의 구현체입니다.

일반적으로 PageRequest는 of 메서드를 통해 PageRequest 객체를 생성합니다.
of 메서드는 매개변수에 따라 다양한 형태로 오버로딩돼 있는데 다음과 같은 매개변수 조합을 지원합니다.


| of 메서드  | 매개변수 설명 | 비고 |
| --- | --- | --- |
| `of(int page, int size)` | `page`: 페이지 번호 (0부터 시작), `size`: 페이지당 레코드 수 | 기본적인 페이징 설정, 데이터를 정렬하지 않음 |
| `of(int page, int size, Sort sort)` | `page`: 페이지 번호, `size`: 페이지당 레코드 수, `sort`: 정렬 정보 | sort에 의해 정렬, 특정 정렬 조건 포함 |
| `of(int page, int size, Sort.Direction direction, String... properties)` | `page`: 페이지 번호, `size`: 페이지당 레코드 수, `direction`: 정렬 방향 (ASC 또는 DESC), `properties`: 정렬할 속성들 | Sort.by(direction, properties)에 의해 정렬, 다중 정렬 조건 포함 |

하이버네이트에서 생성하는 쿼리는 생략...
쿼리 로그를 보면 select 쿼리에 limit 쿼리가 포함돼 있는 것을 볼 수 있습니다.
만약 페이지 번호를 0이 아닌 1 이상의 숫자로 설정하면 offset 키워드로 포함되어 레콛 목록을 구분해서
가져오게 됩니다.
이렇게 리턴받은 객체를 출력하면 다음과 같은 출력 결과를 확인할 수 있습니다.

Page 1 of 2 containing com.springboot.advanced_jpa.data.entity.Product instances

Page 객체를 그대로 출력하면 해당 객체의 값을 보여주지 않고 위와 같이 몇 번째 페이지에 해당하는지만 확인할 수 있습니다. 각 페이지를 구성하는 세부적인 값을 보려면 다음과 같이 작성합니다.

Page 객체의 데이터 출력
Page<Product> productPage = productRepository.findByName("펜", PageRequest.of(0,2));
System.out.println(productPage.getContent());

getContent() 메서드를 사용해 출력하면 배열 형태로 값이 출력됩니다.
