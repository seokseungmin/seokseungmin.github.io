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

---

### 6. 데이터베이스 연동
#### 프로젝트 생성
groupId: com.springboot  
artifactId: jpa

#### 의존성 선택
Swagger 의존성 추가:
- springfox-boot-starter
- swagger2
- swagger-ui

Swagger 설정파일, logback 설정파일 추가:
- config/SwaggerConfiguration.java
- logback-spring.xml

#### JPA 설정 추가
```yaml
spring:
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://localhost:3306/springboot
    username: root
    password: 1234
  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

mariaDB에서 `springboot` 데이터베이스를 생성해야 한다.
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

엔티티 클래스 예시:
```java
@Getter
@Setter
@Entity
@Table(name = "product")
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

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;
}
```

---

### 8. 리포지토리 인터페이스 설계
JpaRepository를 상속하는 인터페이스를 생성하면 다양한 메서드를 손쉽게 활용할 수 있다.
```java
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

---

### 9. DAO 설계
DAO (Data Access Object)는 데이터베이스에 접근하기 위한 로직을 관리하는 객체이다. 서비스 레이어와 리포지토리의 중간 계층을 구성한다.

```java
public interface ProductDAO {
    Product insertProduct(Product product);
    Product selectProduct(Long number);
    Product updateProductName(Long number, String name);
    void deleteProduct(Long number);
}
```

```java
@RequiredArgsConstructor
@Component
public class ProductDAOImpl implements ProductDAO {
    private final ProductRepository productRepository;

    @Override
    public Product insertProduct(Product product) {
        return productRepository.save(product);
    }

    @Override
    public Product selectProduct(Long number) {
        return productRepository.findById(number)
            .orElseThrow(() -> new RuntimeException("조회 오류"));
    }

    @Override
    public Product updateProductName(Long number, String name) {
        Product product = productRepository.findById(number)
            .orElseThrow(() -> new RuntimeException("조회 오류"));
        product.setName(name);
        product.setUpdatedAt(LocalDateTime.now());
        return productRepository.save(product);
    }

    @Override
    public void deleteProduct(Long number) {
        Product product = productRepository.findById(number)
            .orElseThrow(() -> new RuntimeException("조회 오류"));
        productRepository.delete(product);
    }
}
```

---

### 10. DAO 연동을 위한 컨트롤러와 서비스 설계
#### 서비스 클래스
```java
public interface ProductService {
    ProductResponseDto getProduct(Long number);
    ProductResponseDto saveProduct(ProductDto productDto);
    ProductResponseDto changeProductName(Long number, String name) throws Exception;
    void deleteProduct(Long number) throws Exception;
}

@RequiredArgsConstructor
@Service
public class ProductServiceImpl implements ProductService {
    private final ProductDAO productDAO;

    @Override
    public ProductResponseDto getProduct(Long number) {
        Product product = productDAO.selectProduct(number);
        return new ProductResponseDto(product);
    }

    @Override
    public ProductResponseDto saveProduct(ProductDto productDto) {
        Product product = new Product(productDto);
        product.setCreatedAt(LocalDateTime.now());
        product.setUpdatedAt(LocalDateTime.now());
        return new ProductResponseDto(productDAO.insertProduct(product));
    }

    @Override
    public ProductResponseDto changeProductName(Long number, String name) throws Exception {
        return new ProductResponseDto(productDAO.updateProductName(number, name));
    }

    @Override
    public void deleteProduct(Long number) throws Exception {
        productDAO.deleteProduct(number);
    }
}
```

#### 컨트롤러
```java
@RequiredArgsConstructor
@RestController
@RequestMapping("/product")
public class ProductController {
    private final ProductService productService;

    @GetMapping
    public ResponseEntity<ProductResponseDto> getProduct(Long number) {
        return ResponseEntity.ok(productService.getProduct(number));
    }

    @PostMapping
    public ResponseEntity<ProductResponseDto> createProductName(@RequestBody ProductDto productDto) {
        return ResponseEntity.ok(productService.saveProduct(productDto));
    }

    @PutMapping
    public ResponseEntity<ProductResponseDto> changeProductName(@RequestBody ChangeProductNameDto changeProductNameDto) throws Exception {
        return ResponseEntity.ok(productService.changeProductName(changeProductNameDto.getNumber(), changeProductNameDto.getName()));
    }

    @DeleteMapping
    public ResponseEntity<String> deleteProduct(Long number) throws Exception {
        productService.deleteProduct(number);
        return ResponseEntity.ok("정상적으로 삭제되었습니다.");
    }
}
```

### Swagger API를 통한 동작 확인
```java
@RequiredArgsConstructor
@RestController
@RequestMapping("/product")
public class ProductController {
    private final ProductService productService;

    @ApiOperation(value = "GET 메서드 예제")
    @GetMapping
    public ResponseEntity<ProductResponseDto> getProduct(Long number) {
        return ResponseEntity.ok(productService.getProduct(number));
    }

    @ApiOperation(value = "POST 메서드 예제")
    @PostMapping
    public ResponseEntity<ProductResponseDto> createProductName(@RequestBody ProductDto productDto) {
        return ResponseEntity.ok(productService.saveProduct(productDto));
    }

    @ApiOperation(value = "PUT 메서드 예제")
    @PutMapping
    public ResponseEntity<ProductResponseDto> changeProductName(@RequestBody ChangeProductNameDto changeProductNameDto) throws Exception {
        return ResponseEntity.ok(productService.changeProductName(changeProductNameDto.getNumber(), changeProductNameDto.getName()));
    }

    @ApiOperation(value = "DELETE 메서드 예제")
    @DeleteMapping
    public ResponseEntity<String> deleteProduct(Long number) throws Exception {
        productService.deleteProduct(number);
        return ResponseEntity.ok("정상적으로 삭제되었습니다.");
    }
}
```

애플리케이션을 실행하고 웹 브라우저를 통해 Swagger 페이지로

 접속한다: [http://localhost:8080/swagger-ui/index.html#/](http://localhost:8080/swagger-ui/index.html#/)
