---
title: "스프링 핵심가이드 3주차"
excerpt: "[제로베이스] 스프링 핵심가이드"

categories:
  - Spring
tags:
  - [Spring]

permalink: /categories/Spring3Week/ # url

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---

본 게시글은 '스프링 부트 핵심 가이드' 책의 내용을 정리한 것입니다.<br>
저자 : 장정우<br>
출판사 : 위키북스<br>

# 6장 데이터베이스 연동

### 1. 마리아DB 설치
- 다운로드
아래 사이트에서 다운로드 받으면 된다.

다운로드 링크: [mariadb.org/download](https://mariadb.org/download)

---

### 2. ORM
ORM (Object Relational Mapping)은 객체지향 언어에서 객체(클래스)와 RDB의 테이블을 자동으로 매핑하는 방법이다.

#### 장점
- 데이터베이스 쿼리를 객체지향적으로 조작 가능
- 쿼리문 작성이 줄어 개발 비용 감소
- 코드 가독성 향상
- 재사용 및 유지보수 편리
- 데이터베이스에 대한 종속성 감소

#### 단점
- 복잡한 서비스 구현에 한계
- 성능 문제 발생 가능
- 객체와 테이블 간의 불일치 문제

---

### 3. JPA
Java Persistence API는 자바 진영의 ORM 기술 표준 인터페이스 모음이다. JPA는 내부적으로 JDBC를 사용하며, 적절한 SQL을 생성해 데이터베이스를 조작하고 객체를 자동 매핑한다.

JPA의 대표적인 구현체:
- 하이버네이트 (Hibernate)
- 이클립스 링크 (EclipseLink)
- 데이터 뉴클리어스 (DataNucleus)

가장 많이 사용되는 구현체는 하이버네이트이다.

---

### 4. 하이버네이트
자바의 ORM 프레임워크로, JPA 인터페이스를 구현하는 JPA 구현체 중 하나이다. Spring Boot에서는 JPA를 더욱 편하게 사용하도록 모듈화한 Spring Data JPA를 활용하기 때문에 JPA를 직접 사용할 일은 거의 없다.

---

### 5. 영속성 컨텍스트
영속성 컨텍스트는 엔티티와 레코드의 괴리를 해소하고 객체를 보관하는 기능을 수행한다. 엔티티 객체가 영속성 컨텍스트에 들어오면 JPA는 매핑 정보를 데이터베이스에 반영한다.

엔티티 매니저는 영속성 컨텍스트에 접근하기 위한 수단으로 사용된다.

## 엔티티의 생명주기
엔티티 객체는 영속성 컨텍스트에서 다음과 같은 4가지 상태로 구분됨.

### 비영속(New)
영속성 컨텍스트에 추가되지 안은 엔티티 객체의 상태를 의미합니다.

### 영속(Managed)
영속성 컨텍스트에 의해 엔티티 객체가 관리되는 상태입니다.

### 준영속(Detached)
영속성 컨텍스트에 의해 관리되던 엔티티 객체가 컨텍스트와 분리된 상태입니다.

### 삭제(Removed)
데이터베이스에서 레코드를 삭제하기 위해 영속성 컨텍스트에 삭제 요청을 한 상태입니다.

---

### 6. 데이터베이스 연동
#### 프로젝트 생성
groupId: com.springboot  
artifactId: jpa

#### 의존성 선택
Lombok, Spring Configuration Processor, Spring Web, Spring Data JPA, MariaDB Driver, H2 Database 
Swagger 의존성 추가:
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.2'

Swagger 설정파일, logback 설정파일 추가:
- config/SwaggerConfig.java
- logback-spring.xml

#### JPA 설정 추가

```application.properties
spring.application.name=jpa

spring.datasource.url=jdbc:mariadb://localhost:3306/springboot
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver

spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```
create: 애플리케이션이 가동되고 SessionFactory가 실행될때 기존 테이블을 지우고 새로 생성합니다. <br>
이 책에서는 모든 예제를 create로 설정하고 진행했습니다.<br>

create-drop: create와 동일한 기능을 수행하나 애플리케이션을 종료하는 시점에 테이블을 지웁니다.<br>

update: SessionFactory가 실행될 때 객체를 검사해서 변경된 스키마를 갱신합니다.<br>
기존에 저장된 데이터는 유지됩니다.<br>

validate: update처럼 검사하지만 스키마는 건드리지 않습니다. <br>
검사 과정에서 데이터베이스의 테이블 정보와 객체의 정보가 다르면 에러가 발생합니다.<br>

none: ddl-auto 기능을 사용하지 않습니다.<br>

운영환경에서는 create, create-drop, update 기능은 사용하지 않습니다.<br>
데이터베이스에 축적된 데이터를 지워버릴 수도 있고,<br>
사람의 실수로 객체의 정보가 변경됐을 때 운영환경의 데이터베이스 정보까지 변경될 수 있기 때문입니다.<br>
운영환경에서는 대체로 validate나 none을 사용합니다.<br>
반면 개발환경에서는 create 또는 update를 사용하는 편입니다.<br>
show-sql은 로그에 하이버네이트가 생성한 쿼리문을 출력하는 옵션입니다.<br>
아무 설정이 없으면 저장에 용이한 형태로 출력되기 떄문에 사람이 보기에는 불편하게 한 줄로 출력됩니다.<br>
format_sql 옵션으로 사람이 보기 좋게 포매팅 할 수 있습니다.<br>

mariaDB에서 `springboot` 데이터베이스를 생성해야 한다.<br>

```sql
CREATE DATABASE springboot;
```

---

### 7. 엔티티 설계
엔티티는 데이터베이스의 테이블에 대응하는 클래스이다. 데이터베이스 테이블이 아래와 같다면:

| 칼럼명            | 데이터 타입  |
|-----------------|------------|
| 상품번호          | int        |
| 상품이름          | varchar    |
| 상품 가격         | int        |
| 상품 재고         | int        |
| 상품 생성 일자     | DateTime   |
| 상품 정보 변경 일자| DateTime   |

# 엔티티 클래스 
### Product 엔티티

## @Entity
해당클래스가 엔티티임을 명시하기 위한 어노테이션.<br>
클래스 자체는 테이블과 일대일로 매칭되며, 해당 클래스의 인스턴스는 매핑되는 테이블에서 하나의 레코드를 의미합니다.<br>

## @Table
엔티티 클래스는 테이블과 매핑되므로 특별한 경우가 아니면 @Table 어노테이션이 필요하지 않습니다.<br>
@Table 어노테이션을 사용할 때는 클래스의 이름과 테이블의 이름을 다르게 지정해야 하는 경우입니다.<br>
@Table 어노테이션을 명시하지 않으면 테이블의 이름과 클래스 이름이 동일하다는 의미이며,<br>
서로 다른 이름을 쓰려면 @Table(name = 값) 형태로 데이터베이스의 테이블명을 명시해야 합니다.<br>
대체로 자바의 명명법과 데이터베이스가 사용하는 명명법이 다르기 때문에 자주 사용됩니다.<br>

## @id
엔티티 클래스의 필드는 테이블의 칼럼과 매핑됩니다.<br>
@id 어노테이션이 선언된 필드는 테이블의 기본값 역할로 사용됩니다. <br>
모든 엔티티는 @Id 어노테이션이 필요합니다. 꼭 기억해 주세요.<br>

## @GeneratedValue
일반적으로 @id 어노테이션과 함께 사용됩니다.<br>
이 어노테이션은 해당 필드의 값을 어떤 방식으로 자동으로 생성할지 결정할 때 사용합니다.<br>
값 생성 방식은 다음과 같습니다.<br>

## GeneratedValue를 사용하지 않는 방식(직접 할당)
애플리케이션에서 자체적으로 고유한 기본값을 생성할 경우 사용하는 방식입니다.<br>
내부에 정해진 규칙에 의해 기본값을 생성하고 식별자로 사용합니다.<br>

## AUTO
@GeneratedValue의 기본 설정값<br>
기본값을 사용하는 데이터베이스에 맞게 자동 생성합니다.<br>

## IDENTITY
기본값 생성을 데이터베이스에 위임하는 방식입니다.<br>
데이터베이스의 AUTO_INCREMENT를 사용해 기본값을 생성합니다.<br>

## SEQUENCE
@SequenceGenerator 어노테이션으로 식별자 생성기를 설정하고 이를 통해 값을 자동 주입받습니다.<br>
SequenceGEnerator를 정의할 떄는 name, sequenceName, allocationSize를 활용합니다.<br>
@GeneratedValue에 생성기를 설정합니다.<br>

## TABLE
어떤 DBMS를 사용하더라도 동일하게 동작하기를 원할 경우 사용합니다.<br>
식별자로 사용할 숫자의 보관 테이블을 별도로 생성해서 엔티티를 생성할 때마다 값을 갱신하며 사용합니다.<br>
@TableGenerator 어노테이션으로 테이블 정보를 설정합니다.<br>


# @Column
엔티티 클래스의 필드는 자동으로 테이블 칼럼으로 매핑됩니다.<br>
그래서 별다른 설정을 하지 않을 예정이라면 이 어노테이션을 명시하지 않아도 괜찮습니다.<br>
@Column 어노테이션은 필드에 몇가지 설정을 더할 떄 사용합니다.<br>
name: 데이터베이스의 칼럼명을 설정하는 속성입니다. 명시하지 않으면 필드명으로 지정됩니다.<br>
nullable: 레코드를 생성할 때 칼럼 값에 null 처리가 가능한지를 명시하는 속성입니다.<br>
length: 데이터베이스에 저장하는 데이터의 최대 길이를 설정합니다.<br>
unique: 해당 칼럼을 유니크로 설정합니다.<br>

# @Transient
엔티티 클래스에는 선언돼 있는 필드지만 데이터베이스에서는 필요 없을 경우 이 어노테이션을 사용해 데이터베이스에서 이용하지 않게 할 수 있습니다.<br>

```java
package com.springboot.jpa.data.entity;

import jakarta.persistence.*;
import lombok.*;

import java.time.LocalDateTime;

@Entity
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
@ToString(exclude = "name")
@Table(name="product")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    private LocalDateTime createAt;
    private LocalDateTime updateAt;

}

```

---

### 8. 리포지토리 인터페이스 설계
JpaRepository를 상속하는 인터페이스를 생성하면 다양한 메서드를 손쉽게 활용할 수 있다.

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

### 리포지토리 메서드의 생성 규칙
리포지토리에서는 몇가지 명명규칙에 따라 커스텀 메서드도 생성할 수 있습니다.<br>
일반적으로 CRUD에서 따로 생성해서 사용하는 메서드는 대부분 Read 부분에 해당하는 Select 쿼리밖에 없습니다.<br>
엔티티를 저장하거나 갱신 또는 삭제할 때는 별도의 규칙이 필요하지 않기 떄문입니다.<br>
다만 리포지토리에서 기본적으로 제공하는 조회 메서드는 기본값으로 단일 조회하거나 전체 엔티티를 조회하는 것만
지원하고 있기 떄문에 필요에 따라 다른 조회 메서드가 필요합니다.<br>
FindBy: SQL문의 where 절 역할을 수행하는 구문입니다.<br>
예)findByName(String name)<br>
findBy 뒤에 엔티티의 필드값을 입력해서 사용합니다.<br>
AND, OR: 조건을 여러 개 설정하기 위해 사용합니다.<br>
예)findByNameAndEmail(String name,String email)<br>
Like/NotLike:SQL문의 like와 동일한 기능을 수행하며, 특정 문자를 포함하는지 여부를 조건으로 추가합니다.<br>
비슷한 키워드로 Containing, Contains, isContaining이 있습니다.<br>
StartsWith/StartingWith: 특정 키워드로 시작하는 문자열 조건을 설정합니다.<br>
EndsWith/EndingWith: 특정 키워드로 끝나는 문자열 조건을 설정합니다.<br>
isNull/IsNotNull: 레코드 값이 Null이거나 Null이 아닌 값을 검색합니다.<br>
True/False: Boolean 타입의 레코드를 검색할 때 사용합니다.<br>
Before/After: 시간을 기준으로 값을 검색합니다.<br>
LessThan/GreaterThan: 특정 값(숫자)을 기준으로 대소 비교를 할 때 사용합니다.<br>
Between: 두 값(숫자) 사이의 데이터를 조회합니다.<br>
OrderBy:SQL 문에서 order by와 동일한 기능을 수행합니다. <br>
예를 들어, 가격순으로 이름 조회를 수행한다면 List<Product> findByNameOrderByPriceAsc(String name);와 같이 작성합니다.<br>
countBy:SQL문의 count와 동일한 기능을 수행하며, 결과값의 개수(count)를 추출합니다.<br>

---

### 9. DAO 설계
DAO (Data Access Object)는 데이터베이스에 접근하기 위한 로직을 관리하는 객체이다.<br>
서비스 레이어와 리포지토리의 중간 계층을 구성한다.<br>
스프링 데이터 JPA에서 DAO의 개념은 리포지토리가 대체하고 있습니다.<br>
규모가 작은 서비스에서는 DAO를 별도로 설계하지 않고 바로 서비스 레이어에서 데이터베이스에 접근해서 구현하기도 하지만,<br>
DAO를 서비스 레이어와 리포지토리의 중간 계층을 구성하는 역할로 사용할 예정입니다.<br>
실제로 업무에 필요한 비즈니스 로직을 개발하다 보면 데이터를 다루는 중간 계층을 두는 것이 유지보수 측면에서 
용이한 경우가 많습니다.<br>
물론 서비스 레이어에서 리포지토리의 메서드를 호출하고 그 결과에 대해 처리할 수 있지만<br>
비즈니스 로직을 수행하는 과정에서 데이터베이스에 관한 작업을 처리하는 것은 기능을 분리하고 관리하기에 좋은 코드라고
보기 어렵습니다.<br>
객체지향적인 설계에서는 서비스와 비즈니스 레이어를 분리해서 서비스 레이어에서는 서비스 로직을 수행하고 비즈니스 레이어에서는 비즈니스 로직을 수행해야 한다는 의견도 많습니다.<br>
이런 관점만 간단하게 다루고 서비스 객체가 비즈니스 로직까지 포함하는 방향으로 진행하겠습니다.<br>
도메인(엔티티) 객체를 중심으로 다뤄지는 로직은 비즈니스 로직으로 볼 수 있습니다.<br>

```java
package com.springboot.jpa.data.dao;

import com.springboot.jpa.data.entity.Product;

import java.util.Optional;


public interface ProductDAO {

    Product insertProduct(Product product);
    Optional<Product> selectProduct(Long number);
    Product updateProductName(Long number, String namem) throws Exception;
    void deleteProduct(Long number) throws Exception;
}
```

일반적으로 데이터베이스에 접근하는 메서드는 리턴값으로 데이터 객체를 전달합니다.<br>
이때 데이터 객체를 엔티티 객체로 전달할지, DTO 객체로 전달할지에 대해서는 개발자마다 의견이 분분합니다.<br>
일반적인 설계 원칙에서 엔티티 객체는 데이터베이스에 접근하는 계층에서만 사용하도록 정의합니다.<br>
다른 계층으로 데이터를 전달할 때는 DTO 객체를 사용합니다.<br>
그러나 이부분은 회사나 부서마다 견해 차이가 있으므로 각자 정해진 원칙에 따라 진행하는 것이 좋습니다.<br>

```java
package com.springboot.jpa.data.dao.impl;

import com.springboot.jpa.data.dao.ProductDAO;
import com.springboot.jpa.data.entity.Product;
import com.springboot.jpa.data.repository.ProductRepository;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.Optional;

@Repository
public class ProductDAOImpl implements ProductDAO {

    private final ProductRepository productRepository;

    // 생성자 주입 시 @Autowired 생략 가능
    public ProductDAOImpl(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Override
    public Product insertProduct(Product product) {
        Product savedProduct = productRepository.save(product);
        return savedProduct;
    }

    @Override
    public Optional<Product> selectProduct(Long number) {
        Optional<Product> selectedProduct = productRepository.findById(number);
        return selectedProduct;
    }

    @Override
    public Product updateProductName(Long number, String name) throws Exception {
        Optional<Product> selectedProduct = productRepository.findById(number);

        Product updatedProduct;

        if (selectedProduct.isPresent()) {
            Product product = selectedProduct.get();

            product.setName(name);
            product.setUpdateAt(LocalDateTime.now());

            updatedProduct = productRepository.save(product);
        } else {
            throw new Exception();
        }
        return updatedProduct;
    }

    @Override
    public void deleteProduct(Long number) throws Exception {
        Optional<Product> selectedProduct = productRepository.findById(number);

        if (selectedProduct.isPresent()) {
            Product product = selectedProduct.get();

            productRepository.delete(product);
        } else {
            throw new Exception();
        }
    }
}
```
ProductDAOImpl 클래스를 스프링이 관리하는 빈으로 등록하려면 @Component 또는 @Service 어노테이션을 지정해야 합니다.<br>
빈으로 등록된 객체는 다른 클래스가 인터페이스를 가지고 의존성을 주입받을 때 이 구현체를 찾아 주입하게 됩니다.<br>
마찬가지로 DAO 객체에서도 데이터베이스에 접근하기 위해 리포지토리 인터페이스를 사용해 의존성 주입을 받아야합니다.<br>


---

### 10. DAO 연동을 위한 컨트롤러와 서비스 설계
#### 서비스 클래스

앞에서 설계한 구성 요소들을 클라이언트의 요청과 연결하려면 컨트롤러와 서비스를 생성해야 합니다.<Br>
이를 위해 먼저 DAO의 메서드를 호출하고 그 외 비즈니스 로직을 수행하는 서비스 레이어를 생성한 후 컨트롤러를 생성하겠습니다.<Br>

## 서비스 클래스 만들기
서비스 레이어에서는 도메인 모델을 활용해 애플리케이션에서 제공하는 핵심 기능을 제공합니다.<Br>
여기서 말하는 핵심 기능을 구현하려면 세부 기능을 정의해야 합니다.<Br>
세부 기능이 모여 핵심 기능을 구현하기 때문입니다.<Br>
이러한 모든 로직을 서비스 레이어에서 포함하기란 쉽지 않은 일입니다.<Br>
이 같은 아키텍처의 한계를 극복하기 위해 아키텍처를 서비스 로직과 비즈니스 로직으로 분리하기도 합니다.<Br>
도메인을 활용한 세부 기능들을 비즈니스 레이어의 로직에서 구현하고, 서비스 레이어에서는 기능들을
종합해서 핵심 기능을 전달하도록 구성하는 경우가 대표적입니다.<Br>
이 책의 목적은 과도한 기능 구현보다는 어떻게 프로젝트를 구성하고 스프링 부트의 기능을 온전히 사용할 수 있는지를
고민하는 것이므로 서비스 레이어에서 비즈니스 로직을 처리하는 아키텍처로 진행합니다.<Br>
서비스 객체는 DAO와 마찬가지로 추상화해서 구성합니다.<Br>
service 패키지와 클래스, 인터페이스를 구성.<Br>

# Dto 클래스 
### ProductDto

```java
package com.springboot.jpa.data.dto;

import lombok.*;

@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@ToString
@EqualsAndHashCode
@Builder
public class ProductDto {
    private String name;
    private int price;
    private int stock;
}
```

### ProductResponseDto

```java
package com.springboot.jpa.data.dto;

import lombok.*;

@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
@EqualsAndHashCode
@Builder
@ToString
public class ProductResponseDto {

    private Long number;
    private String name;
    private int price;
    private int stock;
}
```

### ChangeProductNameDto

```java
package com.springboot.jpa.data.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

@AllArgsConstructor
@Getter
@Setter
public class ChangeProductNameDto {
    private Long number;
    private String name;
}
```

서비스 인터페이스 작성. 기본적인 CRUD의 기능을 호출하기 위해 간단하게 메서드를 정의하겠습니다.<br>
다음 인터페이스는 DAO에서 구현한 기능을 서비스 인터페이스에서 호출해 결과값을 가져오는 작업을 수행하도록 설계했습니다.<br>
서비스에서는 클라이언트가 요청한 데이터를 적절하게 가공해서 컨트롤러에게 넘기는 역할을 합니다.<br>
이 과정에서 여러 메서드를 사용하는데, 지금은 간단하게 CRUD만 구현하기 때문에 코드가 단순해 보일수 있습니다.<br>
다음 예제를 보면 리턴 타입이 DTO 객체인 것을 볼 수 있습니다.<br>
DAO 객체에서 엔티티 타입을 사용하는 것을 고려하면 서비스 레이어에서 DTO 객체와 엔티티 객체를 각 레이어에 변환해서
전달하는 역할도 수행한다고 볼 수 있습니다.<br>
다만 이부분은 실무 환경에서 내부적으로 어떻게 정의하느냐에 따라 달라 질 수 있습니다.<br>
정리해보면 데이터베이스와 밀접한 관련이 있는 데이터 엑세스 레이어까지는 엔티티 객체를 사용하고,<br>
클라이언트와 가까워지는 다른 레이어에서는 데이터를 교환하는데 DTO 객체를 사용하는 것이 일반적입니다.<br>
이 책에서 구현하는 스프링 부트 애플리케이션 구조는 다음과 같습니다.<br>
클라이언트 (DTO) <-> (DTO) 컨트롤러 (DTO) <-> (DTO) 서비스 (Entity) <-> (Entity) DAO(리포지토리) (Entity) <-> (Entity) 데이터베이스<br>

서비스와 DAO의 사이에서 엔티티로 데이터를 전달하는 것으로 표현했지만 회사나 개발 그룹 내 규정에 따라 DTO를 사용하기도 합니다.<br>
위 구조는 각 레이어 사이의 큰 데이터의 전달을 표현한 것이고, 단일 데이터나 소량의 데이터를 전다하는 경우 DTO나 엔티티를 사용하지 않기도 합니다.<br>

```java
package com.springboot.jpa.service;

import com.springboot.jpa.data.dto.ProductDto;
import com.springboot.jpa.data.dto.ProductResponseDto;


public interface ProductService {

    ProductResponseDto getProduct(Long number);
    ProductResponseDto saveProduct(ProductDto productDto);
    ProductResponseDto changeProductName(Long number, String name) throws Exception;
    void deleteProduct(Long number) throws Exception;
}
```

```java
package com.springboot.jpa.service.impl;

import com.springboot.jpa.data.dao.ProductDAO;
import com.springboot.jpa.data.dto.ProductDto;
import com.springboot.jpa.data.dto.ProductResponseDto;
import com.springboot.jpa.data.entity.Product;
import com.springboot.jpa.service.ProductService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.Optional;

@Service
public class ProductServiceImpl implements ProductService {

    private final Logger LOGGER = LoggerFactory.getLogger(ProductServiceImpl.class);
    private final ProductDAO productDAO;

    public ProductServiceImpl(ProductDAO productDAO) {
        this.productDAO = productDAO;
    }

    @Override
    public ProductResponseDto getProduct(Long number) {
        LOGGER.info("[getProduct] input number : {}", number);
        Optional<Product> product = productDAO.selectProduct(number);

        LOGGER.info("[getProduct] product number : {}, name : {}", product.get().getNumber(), product.get().getName());
        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(product.get().getNumber());
        productResponseDto.setName(product.get().getName());
        productResponseDto.setPrice(product.get().getPrice());
        productResponseDto.setStock(product.get().getStock());

        return productResponseDto;
    }

    @Override
    public ProductResponseDto saveProduct(ProductDto productDto) {
        LOGGER.info("[saveProduct] productDto : {}", productDto.toString());

        Product product = new Product();
        product.setName(productDto.getName());
        product.setPrice(productDto.getPrice());
        product.setStock(productDto.getStock());
        product.setCreateAt(LocalDateTime.now());
        product.setUpdateAt(LocalDateTime.now());

        Product savedProduct = productDAO.insertProduct(product);
        LOGGER.info("[saveProduct] savedProduct : {}", savedProduct);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(savedProduct.getNumber());
        productResponseDto.setName(savedProduct.getName());
        productResponseDto.setPrice(savedProduct.getPrice());
        productResponseDto.setStock(savedProduct.getStock());

        return productResponseDto;
    }

    @Override
    public ProductResponseDto changeProductName(Long number, String name) throws Exception {
        Product product = productDAO.updateProductName(number, name);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(product.getNumber());
        productResponseDto.setName(product.getName());
        productResponseDto.setPrice(product.getPrice());
        productResponseDto.setStock(product.getStock());

        return productResponseDto;
    }

    @Override
    public void deleteProduct(Long number) throws Exception {
        productDAO.deleteProduct(number);
    }
}
```

현재 서비스 레이어에는 DTO 객체와 엔티티 객체가 공존하도록 설계돼 있어 변환 작업이 필요합니다.
DTO 객체를 생성하고 값을 넣어 초기화하는 작업을 수행하는데, 이런 부분은 빌더 패턴을 활용하거나 엔티티 객체나 DTO 객체 내부에 변환하는 메서드를 추가해서 간단하게 전환할 수 있습니다.


#### 컨트롤러
### Swagger API를 통한 동작 확인
컨트롤러는 클라이언트로부터 요청을 받고 해당 요청에 대해 서비스 레이어에 구현된 적절한 메서드를 호출해서 결과값을 받습니다.
이처럼 컨트롤러는 요청과 응답을 전달하는 역할만 맡는 것이 좋습니다.

```java
package com.springboot.jpa.data.controller;

import com.springboot.jpa.data.dto.ChangeProductNameDto;
import com.springboot.jpa.data.dto.ProductDto;
import com.springboot.jpa.data.dto.ProductResponseDto;
import com.springboot.jpa.service.ProductService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/product")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @Operation(summary = "Get product by number", description = "상품 번호로 상품을 조회합니다.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "정상적으로 조회되었습니다."),
            @ApiResponse(responseCode = "404", description = "상품을 찾을 수 없습니다.")
    })
    @GetMapping
    public ResponseEntity<ProductResponseDto> getProduct(@RequestParam Long number) {
        ProductResponseDto productResponseDto = productService.getProduct(number);
        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }

    @Operation(summary = "Create a new product", description = "새로운 상품을 생성합니다.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "정상적으로 생성되었습니다."),
            @ApiResponse(responseCode = "400", description = "잘못된 요청입니다.")
    })
    @PostMapping
    public ResponseEntity<ProductResponseDto> createProduct(@RequestBody ProductDto productDto) {
        ProductResponseDto productResponseDto = productService.saveProduct(productDto);
        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }

    @Operation(summary = "Change product name", description = "상품의 이름을 변경합니다.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "정상적으로 변경되었습니다."),
            @ApiResponse(responseCode = "404", description = "상품을 찾을 수 없습니다.")
    })
    @PutMapping
    public ResponseEntity<ProductResponseDto> changeProductName(@RequestBody ChangeProductNameDto changeProductNameDto) throws Exception {
        ProductResponseDto productResponseDto = productService.changeProductName(changeProductNameDto.getNumber(), changeProductNameDto.getName());
        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }

    @Operation(summary = "Delete product by number", description = "상품 번호로 상품을 삭제합니다.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "정상적으로 삭제되었습니다."),
            @ApiResponse(responseCode = "404", description = "상품을 찾을 수 없습니다.")
    })
    @DeleteMapping
    public ResponseEntity<String> deleteProduct(@RequestParam Long number) throws Exception {
        productService.deleteProduct(number);
        return ResponseEntity.status(HttpStatus.OK).body("정상적으로 삭제되었습니다.");
    }

}
```

### Swagger config 설정

```java
package com.springboot.jpa.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    @Bean
    public GroupedOpenApi dividendApi() {
        return GroupedOpenApi.builder()
                .group("jpa-api")
                .pathsToMatch("/product/**")
                .build();
    }

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("스프링부트 JPA 학습 프로젝트")
                        .description("스프링부트 핵심가이드를 통해 스프링 API를 공부하는 프로젝트입니다.")
                        .version("1.0"));
    }
}
```

지금까지 구현한 코드에는 상품정보를 조회, 저장, 삭제할 수 있는 기능을 비롯해 상품정보 중 상품의
이름을 수정하는 기능이 포홤돼 있습니다.
 각 기능에 대한 요청은 '컨트롤러 - 서비스 - DAO - 리포지토리'의 계층을 따라 이동하고, 그것의 역순으로 응답을 전달하는 구조입니다.
 그럼 Swagger API를 통해 애플리케이션의 클라이언트 입장에서 기능을 요청해보고 어떻게 결과가 나타나는지 살펴보겠습니다.

애플리케이션을 실행하고 웹 브라우저를 통해 Swagger 페이지로

 접속한다: [http://localhost:8080/swagger-ui/index.html#/](http://localhost:8080/swagger-ui/index.html#/)
