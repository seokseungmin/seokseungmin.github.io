---
title: "스프링 핵심가이드 5주차"
excerpt: "[제로베이스] 스프링 핵심가이드"

categories:
  - Spring
tags:
  - [Spring]

permalink: /categories/Spring5Week/ # url

toc: true
toc_sticky: true

date: 2024-08-02
last_modified_at: 2024-08-02
---

[스프링부트 핵심가이드 프로젝트 따라하면서 짠 코드](https://github.com/seokseungmin/relationship)<br>

본 게시글은 '스프링 부트 핵심 가이드' 책의 내용을 정리한 것입니다.<br>
저자 : 장정우<br>
출판사 : 위키북스<br>

# 연관관계 매핑

REDBMS를 사용할 때는 테이블 하나만 사용해서 애플리케이션의 모든 기능을 구현하기란 
불가능합니다.
대체로 설계가 복잡해지면 각 도메인에 맞는 테이블을 설계하고 연관관계를 조인(Join) 등의 기능을 활용합니다.
JPA를 사용하는 애플리케이션에서도 테이블의 연관관계를 엔티티 간의 연관관계로 표현할 수 있습니다.
다만 객체와 테이블의 성질이 달라 정확한 연관관계를 표현할 수는 없습니다.
이번 장에서는 JPA에서 이러한 제약을 보완하면서 연관관계를 매핑하고 사용하는 방법을 알아보겠습니다.

# 연관관계 매핑 종류와 방향
연관관계를 맺는 두 엔티티 간에 생성할 수 있는 연관관계의 종류는 다음과 같습니다.

- One To One : 일대일(1:1)
- One To Many : 일대다(1:N)
- Many To One : 다대일(N:1)
- Many TO Many : 다대다(N:M)

연관관계를 이해하기 위해 한 가게가 재고관리시스템을 통해 상품을 관리하고 있다고 해봅시다.
재고로 등록돼 있는 상품 엔티티에는 가게로 상품을 공급하는 공급업체의 정보 엔티티가 매핑돼 있습니다.
공급업체 입장에서 보면 한 가게에 납품하는 상품이 여러 개 있을 수 있으므로 상품 엔티티와는 일대다 관계가 되며,
상품 입장에서 보면 하나의 공급업체에 속하게 되므로 다대일 관계가 됩니다.
즉, 어떤 엔티티를 중심으로 연관 엔티티를 보느냐에 따라 연관관계의 상태가 달라집니다,

데이터베이스에서는 두 테이블의 연관관계를 설정하면 외래키를 통해 서로 조인해서 참조하는 구조로
생성되지만 JPA를 사용하는 객체지향 모델링에서는 엔티티 간 참조 방향을 설정할 수 있습니다.
데이터베이스와 관계를 일치시키기 위해 양방향으로 설정해도 무관하지만
비즈니스 로직의 관점에서 봤을 때는 단방향 관계만 설정해도 해결되는 경우가 많습니다.
이러한 단방향과 양방향 관계에 대해 간단하게 정리하면 다음과 같습니다.

- 단방향 : 두 엔티티의 관계에서 한쪽의 엔티티만 참조하는 형식입니다.
- 양방향 : 두 엔티티의 관계에서 각 엔티티가 서로의 엔티티를 참조하는 형식입니다.

연관관계가 설정되면 한 테이블에서 다른 테이블의 기본값을 외래키로 갖게 됩니다.
이런 관계에서는 주인(Owner)이라는 개념이 사용됩니다.
일반적으로 외래키를 가진 테이블이 그 관계의 주인이 되며, 주인은 외래키를 사용할 수 있으나
상대 엔티티는 읽는 작업만 수행할 수 있습니다.

#### 프로젝트 생성
groupId: com.springboot  
artifactId: relationship

#### 의존성 선택
Lombok, Spring Configuration Processor, Spring Web, Spring Data JPA, MariaDB Driver

그리고 8장에서 사용한 소스코드를 그대로 가져와 사용합니다.
이에 따라 queryDSL의 의존성과 플러그인 설정을 그대로 추가해야 합니다.

# 일대일 매핑

먼저 두 엔티티 간에 일대일 매핑을 만들어 보겠습니다.
우선 지금까지 사용해온 Product 엔티티를 대상으로 일대일로 매핑될 상품정보 테이블을 생성합니다.
상품 테이블 <-> 상품 정보 테이블 (일대일 관계)

### 일대일 단방향 매핑
프로젝트의 entity 패키지 안에 다음과 같이 상품정보 엔티티를 작성합니다.
상품정보에 대한 도메인은 ProductDetail로 설정해서 진행하겠습니다.

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "product_detail")
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class ProductDetail extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String description;

    @OneToOne
    @JoinColumn(name ="product_number")
    private Product product;
}
```

엔티티를 작성했던 방법 그대로 상품정보 엔티티를 작성합니다.
그리고 상품 번호에 매핑하기 위해  

```
@OneToOne
@JoinColumn(name ="product_number")
private Product product;
```

다음과 같이 작성합니다.
@OneToOne 어노테이션은 다른 엔티티 객체를 필드로 정의했을 때 일대일 연관관계로 매핑하기 위해 사용됩니다.
뒤이어 @JoinColumn 어노테이션을 사용해 매핑할 외래키를 설정합니다.
@JoinColumn 어노테이션은 기본값이 설정돼 있어 자동으로 이름을 매핑하지만 의도한 이름이 들어가지 않기 때문에
name 속성을 사용해 원하는 컬럼명을 지정하는 것이 좋습니다. 만약 @JoinColumn을 선언하지 않으면
엔티티를 매핑하는 중간 테이블이 생기면서 관리 포인트가 늘어나 좋지 않습니다.
간단하게 @JoinColumn 어노테이션에서 사용할수 있는 속성을 설명하면 다음과 같습니다.

- name : 매핑할 외래키의 이름을 설정합니다.
- referencedColumName : 외래키가 참조할 상대 테이블의 칼럼명을 지정합니다.
- foreignKey : 외래키를 생성하면서 지정할 제약조건을 설정합니다.(unique, nullable, insertable, updatable 등).

이렇게 엔티티 클래스를 생성하면 단방향 관계의 일대일 관계 매핑이 완성됩니다.
hibernate.ddl-auto의 값을 create로 설정한 후 애플리케이션을 실행하면 하이버네이트에서 자동으로 테이블을 생성합니다.

생성된 상품정보 엔티티 객체들을 사용하기 위해 리포지토리 인터페이스를 생성합니다.
기존에 작성했던 ProductDetailRepository와 동일한 형식으로 작성합니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.ProductDetail;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductDetailRepository extends JpaRepository<ProductDetail, Long> {
}
```

그럼 연관관계를 활용한 데이터 생성 및 조회 기능을 테스트 코드로 간략하게 작성해보겠습니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.ProductDetail;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ProductDetailRepositoryTest {

    @Autowired
    ProductDetailRepository productDetailRepository;

    @Autowired
    ProductRepository productRepository;

    @Test
    void saveAndReadTest1() {
        Product product = new Product();
        product.setName("스프링 부트 JPA");
        product.setPrice(5000);
        product.setStock(500);

        productRepository.save(product);

        ProductDetail productDetail = new ProductDetail();
        productDetail.setProduct(product);
        productDetail.setDescription("스프링 부트와 JPA를 함께 볼 수 있는 책");

        productDetailRepository.save(productDetail);

        // 생성한 데이터 조회
        System.out.println("savedProduct : " + productDetailRepository.findById(productDetail.getId()).get().getProduct());

        System.out.println("savedProductDetail : " + productDetailRepository.findById(productDetail.getId()).get());
    }
}
```

위와 같은 테스트 코드를 실행하기 위해서는 상품과 상품정보에 매핑된 리포지토리에 대해
의존성 주입을 받아야 합니다.
그리고 이 테스트에서 조회할 엔티티 객체를 저장합니다.
ProductDetail 객체에서 Product 객체를 일대일 단방향 연관관계를 설정했기 때문에
ProductDetailRepository에서 ProductDetail 객체를 조회한 후 연관 매핑된 Product 객체를 조회할 수 있습니다.

생성한 데이터 조회 줄에서 조회하는 쿼리는 다음과 같이 표현됩니다.

```
Hibernate: 
    select
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at,
        pd1_0.updated_at 
    from
        product_detail pd1_0 
    left join
        product p1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        pd1_0.id=?
```

select 구문을 보면 ProductDetail 객체와 Product 객체가 함께 조회되는 것을 볼 수 있습니다.
이처럼 엔티티를 조회할 때 연관된 엔티티도 함께 조회하는 것을 '즉시 로딩'이라고 합니다.
그리고 left join이 수행되는 것을 볼 수 있습니다.
여기서 left join이 수행되는 이유는 @OneToOne 어노테이션 때문입니다.

# @OneToOne 어노테이션 인터페이스

```
public @interface OneToOne {
    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};

    FetchType fetch() default FetchType.EAGER;

    boolean optional() default true;

    String mappedBy() default "";

    boolean orphanRemoval() default false;
}
```

이후에 더 자세히 살펴볼 예정이므로 여기서는 fetch() 요소와 optional() 요소만 보겠습니다.
@OneToOne 어노테이션은 기본 fetch 전략으로 EAGER, 즉 즉시 로딩 전략이 채택된 것을 볼 수 있습니다.
그리고 optional() 메서드는 기본값으로 true가 설정돼 있습니다.
기본값이 true인 상태는 매핑되는 값이 nullable이라는 것을 의미합니다.
반드시 값이 있어야 한다면 ProductDetail 엔티티에서 속성값에 다음과 같이 설정할 수 있습니다.

### Product 객체가 반드시 있어야 하는 ProductDetail 엔티티 클래스

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "product_detail")
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class ProductDetail extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String description;

    @OneToOne(optional = false)
    @JoinColumn(name ="product_number")
    private Product product;
}
```

위와 같이 @OneToOne 어노테이션 'optional = false' 속성을 설정하면 product가 null인 값을
허용하지 않게 됩니다.
위와 같이 설정하고 애플리케이션을 실행하면 다음과 같이 테이블을 생성하는 쿼리에서
not null이 설정되는 것을 확인할 수 있습니다.

```
Hibernate: 
    create table product_detail (
        created_at datetime(6),
        id bigint not null auto_increment,
        product_number bigint not null,
        updated_at datetime(6),
        description varchar(255),
        primary key (id)
    ) engine=InnoDB
```

그리고 앞에서 작성한 예제를 실행하면 다음과 같이 쿼리문이 바뀌에 실행됩니다.

```
Hibernate: 
    select
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.product_number,
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at,
        pd1_0.updated_at 
    from
        product_detail pd1_0 
    join
        product p1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        pd1_0.id=?
```

즉, @OneToOne 어노테이션에 'optional = false' 속성을 지정한 경우에는 left join이 join으로 바뀌어
실행됩니다. 이처럼 객체에 대한 설정에 따라 JPA는 최적의 쿼리를 생성해서 실행합니다.
이후 내용을 진행하기 위해 @OneToOne에 적용했던 'optional = false' 속성은 제거하겠습니다.

# 일대일 양방향 매핑
이번에는 앞에서 생성한 일대일 단방향 설정을 양방향 설정으로 변경해보겠습니다.
사실 객체에서의 양방향 개념은 양쪽에서 단방향으로 서로를 매핑하는 것을 의미합니다.
일대일 양방향 매핑을 위해서는 다음과 같이 엔티티를 추가합니다.

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    @OneToOne(mappedBy = "product")
    @ToString.Exclude
    private ProductDetail productDetail;

    @ManyToOne
    @JoinColumn(name = "provider_id")
    @ToString.Exclude
    private Provider provider;

}
```

이렇게 설정하고 애플리케이션을 실행하면 product 테이블에도 칼럼이 생성되는 것을 볼 수 있습니다.

```
Hibernate: 
    select
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at,
        pd1_0.updated_at 
    from
        product_detail pd1_0 
    left join
        product p1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        pd1_0.id=?
```

여러 테이블끼리 연관관계가 설정돼 있어 여러 left join 이 설정되는 것은 괜찮으나 위와 같이 양쪽에서
외래키를 가지고 left join이 두번이나 수행되는 경우는 효율성이 떨어집니다.
실제 데이터베이스에서도 테이블 간 연관관계를 맺으면 한쪽 테이블이 외래키를 가지는 구조로 이뤄집니다.
바로 앞에서 언급한 '주인' 개념입니다.

JPA에서도 실제 데이터베이스의 연관관계를 반영해서 한쪽의 테이블에서만 외래키를 바꿀 수 있도록 정하는 것이 좋습니다.
이 경우 엔티티는 양방향으로 매핑하되 한쪽에게만 외래키를 줘야하는데, 이때 사용되는 속성 값이 mappedBy입니다.
mappedBy는 어떤 객체가 주인인지 표시하는 속성이라고 볼 수 있습니다.
다음과 같이 Product 객체에 mappedBy 속성을 추가해 보겠습니다.

mappedBy 속성을 추가한 Product 클래스

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    @OneToOne(mappedBy = "product")
    @ToString.Exclude
    private ProductDetail productDetail;

    @ManyToOne
    @JoinColumn(name = "provider_id")
    @ToString.Exclude
    private Provider provider;

}
```
@#OneToOne 어노테이션에 mappedBy 속성값을 사용했습니다.
mapppedBy에 들어가는 값은 연관관계를 갖고 있는 상대 엔티티에 있는 연관관계 필드의 이름이 됩니다.
이 설정을 마치면 ProductDetail 엔티티가 Product 엔티티의 주인이 되는 것입니다.
애플리케이션을 실행하고 데이터베이스의 테이블을 보면 외래키가 사라진 것을 볼 수 있습니다.

그리고 다시 테스트 코드를 실행하면 toString을 실행하는 시점에서 StackOverflowError가 발생하는 것을 볼 수 있습니다.
양방향으로 연관관계가 설정되면 ToString을 사용할 때 순환참조가 발생하기 때문입니다.
그렇기 때문에 필요한 경우가 아니라면 대체로 단방향으로 연관관계를 설정하거나 양방향 설정이 필요할 경우에는
순환참조 제거를 위해 다음과 같이 exclude를 사용해 ToString에서 제외 설정을 하는 것이 좋습니다.

```
 @OneToOne(mappedBy = "product")
    @ToString.Exclude
    private ProductDetail productDetail;
```

# 다대일, 일대다 매핑
상품 테이블과 공급업체 테이블은 다음과 같이 상품 테이블의 입장에서 볼 경우에는 다대일,
공급업체 테이블의 입장에서 볼 경우에는 일대다 관계로 볼 수 있습니다.
이런 관계는 어떻게 구현해야 할지 직접 매핑하면서 알아보겠습니다.

# 다대일 단방향 매핑
먼저 공급업체 테이블에 매핑되는 엔티티 클래스를 만들겠습니다.
다음과 같이 엔티티 클래스를 생성할 수 있습니다.

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "provider", fetch = FetchType.EAGER, cascade = CascadeType.PERSIST, orphanRemoval = true)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();
}
```

공급업체는 provider라는 도메인을 사용해서 정의했습니다.
공급업체의 정보를 담는다면 더 많은 칼럼이 필요하겠지만 간단한 실습을 위해 필드로는
id와 name만 작성합니다. 여기에 BaseEntity를 통해 생성일자와 변경일자를 상속받습니다.
상품 엔티티에서는 공급업체의 번호를 받기 위해 다음과 같이 엔티티 필드의 구성을 추가해야 합니다.
다음과 같이 상품 엔티티에 필드를 추가하겠습니다.

상품 엔티티와 공급업체 엔티티의 다대일 연관관계 설정

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    @OneToOne(mappedBy = "product")
    @ToString.Exclude
    private ProductDetail productDetail;

    @ManyToOne
    @JoinColumn(name = "provider_id")
    @ToString.Exclude
    private Provider provider;

}
```

다음은 공급업체 엔티티에 대한 다대일 연관관계를 설정합니다.

```
  @ManyToOne
    @JoinColumn(name = "provider_id")
    @ToString.Exclude
    private Provider provider;
```

일반적으로 외래키를 갖는 쪽이 주인의 역할을 수행하기 때문에 이 경우 상품 엔티티가 공급업체
엔티티의 주인입니다.
위와 같이 설정한 후 애플리케이션을 가동하면 Product 테이블과 Provider 테이블이 생성됩니다.

당장은 사용하지 않지만 이후 공급업체 엔티티를 활용할 수 있게 리포지토리를 생성합니다.
기존에 리포지토리를 생성했던 방식과 동일하게 다음과 같이 코드를 작성했습니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Provider;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProviderRepository extends JpaRepository<Provider, Long> {
}
```

이제 작성된 코드를 기반으로 테스트를 진행하겠습니다.
지금 다루고 있는 두 엔티티에서 주인은 Product 엔티티이기 때문에 productRepository를 활용해 테스트합니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.Provider;
import org.assertj.core.util.Lists;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class ProviderRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    ProviderRepository providerRepository;

    @Test
    void relationshipTest1() {
        // 테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 물산");

        providerRepository.save(provider);

        Product product = new Product();
        product.setName("가위");
        product.setPrice(5000);
        product.setStock(500);
        product.setProvider(provider);

        productRepository.save(product);

        System.out.println("product : " + productRepository.findById(1L).orElseThrow(RuntimeException::new));

        System.out.println("provider : " + productRepository.findById(1L).orElseThrow(RuntimeException::new).getProvider());
    }

    @Test
    void relationshipTest() {
        //테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 상사");

        providerRepository.save(provider);

        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(2000);
        product1.setStock(100);
        product1.setProvider(provider);

        Product product2 = new Product();
        product2.setName("가방");
        product2.setPrice(20000);
        product2.setStock(200);
        product2.setProvider(provider);

        Product product3 = new Product();
        product3.setName("노트");
        product3.setPrice(3000);
        product3.setStock(100);
        product3.setProvider(provider);

        productRepository.save(product1);
        productRepository.save(product2);
        productRepository.save(product3);

        List<Product> productList = providerRepository.findById(provider.getId()).get().getProductList();

        for (Product product : productList) {
            System.out.println(product);
        }
    }

    @Test
    void cascadeTest() {
        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.save(provider);
    }

    private Provider saveProvider(String name) {
        Provider provider = new Provider();
        provider.setName(name);
        return providerRepository.save(provider);
    }

    private Product saveProduct(String name, Integer price, Integer stock) {
        Product product = new Product();
        product.setName(name);
        product.setPrice(price);
        product.setStock(stock);
        return productRepository.save(product);
    }

   @Test
    void orphanRemovalTest() {

        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.saveAndFlush(provider);

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);

        Provider foundProvider = providerRepository.findById(1L).get();

        // 리스트가 비어 있지 않은지 확인
        if (!foundProvider.getProductList().isEmpty()) {
            foundProvider.getProductList().remove(0);
        }

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);
    }
```

이제 각 엔티티의 연관관계를 테스트하기 위해 테스트 데이터를 생성해야 합니다.
그렇기 때문에 두 리포지토리에 대해 의존성 주입을 받았습니다.
그 뒤 각 리포지토리를 통해 다음과 같이 테스트 데이터를 생성합니다.
위에서 생성한 provider 객체를 product에 추가해서 데이터베이스에 저장하는 코드입니다.
애플리케이션을 실행했을때 하이버네이트로 생성된 쿼리를 보면 다음과 같습니다.

```
Hibernate: 
    insert 
    into
        product
        (created_at, name, price, provider_id, stock, updated_at) 
    values
        (?, ?, ?, ?, ?, ?) 
    returning number
```

쿼리로 데이터를 저장할 때는 provider_id 값만 들어가는 것을 볼 수 있습니다.
이렇게 product 테이블에는 @JoinColumn에 설정한 이름을 기반으로 자동으로 값을 선정해서 추가하게 됩니다.

Product 엔티티에서 단방향으로 Provider 엔티티 연관관계를 맺고 있기 때문에 ProductRepository 객체도 조회가 가능합니다.

```java
System.out.println("product : " + productRepository.findById(1L).orElseThrow(RuntimeException::new));
System.out.println("provider : " + productRepository.findById(1L).orElseThrow(RuntimeException::new).getProvider());
```

위 두 줄에서 생성하는 쿼리는 하이버네이트에서 동일하게 실행됩니다.

```
Hibernate: 
    select
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.updated_at,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at 
    from
        product p1_0 
    left join
        product_detail pd1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        p1_0.number=?
```

이 쿼리가 수행되고 출력된 결과는 다음과 같습니다.

```
product : Product(super=BaseEntity(createdAt=2024-08-02T09:52:02.602954, updatedAt=2024-08-02T09:52:02.602954), number=1, name=가위, price=5000, stock=500)
provider : Provider(super=BaseEntity(createdAt=2024-08-02T09:52:02.434550, updatedAt=2024-08-02T09:52:02.434550), id=1, name=oo 물산)
```

# 다대일 양방향 매핑
앞에서 상품 엔티티와 공급업체 엔티티 사이에 다대일 단방향 연관관계를 설정했습니다.
이제 반대로 공급업체를 통해 등록된 상품을 조회하기 위한 일대다 연관관계를 설정해보겠습니다.
JPA에서는 이처럼 양쪽에서 단방향으로 매핑하는 것이 양방향 매핑 방식입니다.
이번에는 다음과 같이 공급업체 엔티티에서만 연관관게를 설정합니다. 

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "provider", fetch = FetchType.EAGER, cascade = CascadeType.PERSIST, orphanRemoval = true)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();
}
```

일대다 연관 관계의 경우 여러 상품 엔티티가 포함될 수 있어 컬렉션(Collection, List, Map)
형식으로 필드를 생성합니다. 이렇게 @OneToMany가 붙은 쪽에서 @JoinColumn 어노테이션을 사용하면 상대 엔티티에 외래키가 설정됩니다.
또한 롬복의 ToString에 의해 순환참조가 발생할 수 있어  @ToString.Exclude으로 ToString에서 제외 처리를 하는 것이 좋습니다.
fetch = FetchType.EAGER로 설정한 것은 @OneToMany의 기본 fetch 전략이 Lazy이기 때문에 즉시 로딩으로 조정한 것입니다.
앞으로 진행할 테스트에서 지연 로딩 방식을 사용하면 'no Session'으로 에러가 발생하기 때문에 조정했습니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.Provider;
import org.assertj.core.util.Lists;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class ProviderRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    ProviderRepository providerRepository;

    @Test
    void relationshipTest1() {
        // 테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 물산");

        providerRepository.save(provider);

        Product product = new Product();
        product.setName("가위");
        product.setPrice(5000);
        product.setStock(500);
        product.setProvider(provider);

        productRepository.save(product);

        System.out.println("product : " + productRepository.findById(1L).orElseThrow(RuntimeException::new));

        System.out.println("provider : " + productRepository.findById(1L).orElseThrow(RuntimeException::new).getProvider());
    }

    @Test
    void relationshipTest() {
        //테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 상사");

        providerRepository.save(provider);

        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(2000);
        product1.setStock(100);
        product1.setProvider(provider);

        Product product2 = new Product();
        product2.setName("가방");
        product2.setPrice(20000);
        product2.setStock(200);
        product2.setProvider(provider);

        Product product3 = new Product();
        product3.setName("노트");
        product3.setPrice(3000);
        product3.setStock(100);
        product3.setProvider(provider);

        productRepository.save(product1);
        productRepository.save(product2);
        productRepository.save(product3);

        List<Product> productList = providerRepository.findById(provider.getId()).get().getProductList();

        for (Product product : productList) {
            System.out.println(product);
        }
    }

    @Test
    void cascadeTest() {
        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.save(provider);
    }

    private Provider saveProvider(String name) {
        Provider provider = new Provider();
        provider.setName(name);
        return providerRepository.save(provider);
    }

    private Product saveProduct(String name, Integer price, Integer stock) {
        Product product = new Product();
        product.setName(name);
        product.setPrice(price);
        product.setStock(stock);
        return productRepository.save(product);
    }

    @Test
    void orphanRemovalTest() {

        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.saveAndFlush(provider);

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);

        Provider foundProvider = providerRepository.findById(1L).get();

        // 리스트가 비어 있지 않은지 확인
        if (!foundProvider.getProductList().isEmpty()) {
            foundProvider.getProductList().remove(0);
        }

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);
    }
}
```

위와 같이 Provider 엔티티 클래스를 수정해도 애플리케이션을 가동해보면 칼럼은 변경되지 않습니다.
mappedBy로 설정된 필드는 칼럼에 적용되지 않습니다.
즉, 양쪽에서 연관관계를 설정하고 있을 때 RDBMS의 형식처럼 사용하기 위해 mappedBy를 통해 한쪽으로 외래키 관리를 위임한 것입니다.

### 지연로딩과 즉시로딩
JPA에서 지연로딩(lazy loading)과 즉시로딩(eager loading)은 중요한 개념입니다.
엔티티라는 객체의 개념으로 데이터베이스를 구현했기 때문에 연관관계를 가진 각 엔티티 클래스에는 연관관계가 있는 객체들이 필드에 존재하게 됩니다.
연관관계와 상관없이 즉각 해당 엔티티의 값만 조회하고 싶거나 연관관계를 가진 테이블의 값도 조회하고 싶은 경우 등 여러 조건들을 만족하기 위해 등장한 개념이 지연로딩과 즉시로딩입니다.

그럼 수정된 공급업체 엔티티를 가지고 연관된 엔티티의 값을 가져올 수 있는지 테스트하겠습니다.
다음과 같이 테스트 코드를 작성해서 테스트를 진행합니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.Provider;
import org.assertj.core.util.Lists;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class ProviderRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    ProviderRepository providerRepository;

    @Test
    void relationshipTest1() {
        // 테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 물산");

        providerRepository.save(provider);

        Product product = new Product();
        product.setName("가위");
        product.setPrice(5000);
        product.setStock(500);
        product.setProvider(provider);

        productRepository.save(product);

        System.out.println("product : " + productRepository.findById(1L).orElseThrow(RuntimeException::new));

        System.out.println("provider : " + productRepository.findById(1L).orElseThrow(RuntimeException::new).getProvider());
    }

    @Test
    void relationshipTest() {
        //테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 상사");

        providerRepository.save(provider);

        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(2000);
        product1.setStock(100);
        product1.setProvider(provider);

        Product product2 = new Product();
        product2.setName("가방");
        product2.setPrice(20000);
        product2.setStock(200);
        product2.setProvider(provider);

        Product product3 = new Product();
        product3.setName("노트");
        product3.setPrice(3000);
        product3.setStock(100);
        product3.setProvider(provider);

        productRepository.save(product1);
        productRepository.save(product2);
        productRepository.save(product3);

        List<Product> productList = providerRepository.findById(provider.getId()).get().getProductList();

        for (Product product : productList) {
            System.out.println(product);
        }
    }

    @Test
    void cascadeTest() {
        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.save(provider);
    }

    private Provider saveProvider(String name) {
        Provider provider = new Provider();
        provider.setName(name);
        return providerRepository.save(provider);
    }

    private Product saveProduct(String name, Integer price, Integer stock) {
        Product product = new Product();
        product.setName(name);
        product.setPrice(price);
        product.setStock(stock);
        return productRepository.save(product);
    }

    @Test
    void orphanRemovalTest() {

        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.saveAndFlush(provider);

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);

        Provider foundProvider = providerRepository.findById(1L).get();

        // 리스트가 비어 있지 않은지 확인
        if (!foundProvider.getProductList().isEmpty()) {
            foundProvider.getProductList().remove(0);
        }

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);
    }
}
```

Provider 엔티티 클래스는 Product 엔티티와의 연관관계에서 주인이 아니기 때문에 외래키를 관리할 수 없습니다.
그렇기 때문에 테스트 데이터를 생성하는

```
productRepository.save(product1);
productRepository.save(product2);
productRepository.save(product3);
```
줄과 같이 Provider를 등록한 후 각 Product에 객체를 설정하는 작업을 통해 데이터베이스에 저장합니다.
만약 앞의 예제에서 테스트 데이터를 생성하는 방식이 아니라 Provider 엔티티에 정의한 productList 필드에 
다음과 같이

```
provider.getProductList().add(product1); //무시
provider.getProductList().add(product2); //무시
provider.getProductList().add(product3); //무시
```

Product 엔티티를 추가하는 방식으로 데이터베이스에 레코드를 저장하게 되면 Provider 엔티티 클래스는 연관관계의 주인이 아니기 때문에
해당 데이터는 데이터베이스에 반영되지 않습니다.

List<Product> productList = providerRepository.findById(provider.getId()).get().getProductList();
위에 코드는 ProviderRepository를 통해 연관관계가 매핑된 Product 리스트를 가져와 출력합니다.
이때 생성되는 select 쿼리는 다음과 같습니다.

```
Hibernate: 
    select
        p1_0.id,
        p1_0.created_at,
        p1_0.name,
        p1_0.updated_at,
        pl1_0.provider_id,
        pl1_0.number,
        pl1_0.created_at,
        pl1_0.name,
        pl1_0.price,
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.updated_at,
        pl1_0.stock,
        pl1_0.updated_at 
    from
        provider p1_0 
    left join
        product pl1_0 
            on p1_0.id=pl1_0.provider_id 
    left join
        product_detail pd1_0 
            on pl1_0.number=pd1_0.product_number 
    where
        p1_0.id=?
```

# 일대다 단방향 매핑
앞에서 다대일 연관관계에서의 단방향과 양방향 매핑을 살펴봤습니다.
이번에는 일대다 단방향 매핑 방법을 알아보겠습니다.
참고로 일대다 양방향 매핑은 다루지 않을 예정입니다.
그 이유는 @OneToMany를 사용하는 입장에서는 어느 엔티티 클래스도 연관관계의 주인이 될 수 없기 때문입니다.
@OneToMany 관계에서 주인이 될 수 없는 이유는 이번 절에서 함께 다루겠습니다.
먼저 실습을 위해 새로운 엔티티를 생성하겠습니다. 상품분류 테이블을 생성합니다.

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString
@EqualsAndHashCode
@Table(name = "category")
public class Category {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    private String code;

    private String name;

    @OneToMany(fetch = FetchType.EAGER)
    @JoinColumn(name = "category_id")
    private List<Product> products = new ArrayList<>();
}
```

위와 같이 상품 분류 엔티티 클래스를 생성하고 애플리케이션을 실행하면 상품분류 테아블이 생성되고
상품 테이블에 외래키가 추가되는 것을 확인 할수 있습니다.

상품 분류 엔티티에서 @OneTOMany와 @JoinColumn을 사용하면 상품 엔티티에서 별도의 설정을 하지 않아도
일대다 단방향 연관관계가 매핑됩니다. 
앞에서 언급한 것처럼 @JoinColumn 어노테이션은 필수 사항은 아닙니다.
이 어노테이션을 사용하지 않으면 중간 테이블로 Join테이블이 생성되는 전략이 채택됩니다.
지금 같은 일대다 단방향 관계의 단점은 매핑의 주체가 아닌 반대 테이블에 외래키가 추가되다는 점입니다.
이 방식은 다대일 구조와 다르게 외래키를 설정하기 위해 다른 테이블에 대한 update쿼리를 발생시킵니다.
테스트를 통해 확인해 보겠습니다.

 CategoryRepository를 활용한 테스트

 ```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Category;
import com.springboot.relationship.data.entity.Product;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class CategoryRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    CategoryRepository categoryRepository;

    @Test
    void relationshipTest() {
        // 테스트 데이터 생성
        Product product = new Product();
        product.setName("펜");
        product.setPrice(2000);
        product.setStock(100);

        productRepository.save(product);

        Category category = new Category();
        category.setCode("S1");
        category.setName("도서");
        category.getProducts().add(product);

        categoryRepository.save(category);

        // 테스트 코드
        List<Product> products = categoryRepository.findById(1L).get().getProducts();

        for (Product foundProduct : products) {
            System.out.println(foundProduct);
        }
    }

}
```

테스트 데이터 생성을 위해 ProductRepository의 의존성도 함께 주입받겠습니다.
Product 객체를 Category에서 생성한 리스트 객체에 추가해서 연관관계를 설정합니다.

우선  

``` 
  Product product = new Product();
        product.setName("펜");
        product.setPrice(2000);
        product.setStock(100);

        productRepository.save(product);

        Category category = new Category();
        category.setCode("S1");
        category.setName("도서");
        category.getProducts().add(product);

        categoryRepository.save(category);
```

다음 코드에서의 데이터 생성 쿼리는 다음과 같이 생성됩니다.

```
Hibernate: 
    insert 
    into
        product
        (created_at, name, price, provider_id, stock, updated_at) 
    values
        (?, ?, ?, ?, ?, ?) 
    returning number
Hibernate: 
    insert 
    into
        category
        (code, name) 
    values
        (?, ?) 
    returning id
Hibernate: 
    update
        product 
    set
        category_id=? 
    where
        number=?
```

일대다 연관관게에서는 위와 같이 연관관계 설정을 위한 update 쿼리가 발생합니다.
이 같은 문제를 해결하기 위해서는 일대다 양방향 연관관계를 사용하기보다는 다대일 연관관계를 사용하는 것이 좋습니다.
이렇게 테스트 데이터를 생성한 뒤에 CategoryRepository를 활용해 상품정보를 가져오는 테스트 코드를 실행하면 다음과 같은 쿼리가 생성됩니다.

```
Hibernate: 
    select
        c1_0.id,
        c1_0.code,
        c1_0.name,
        p1_0.category_id,
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.updated_at,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at 
    from
        category c1_0 
    left join
        product p1_0 
            on c1_0.id=p1_0.category_id 
    left join
        product_detail pd1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        c1_0.id=?
```

일대다 연관관계에서는 이처럼 category와 product의 조인이 발생해서 상품 데이터를 정상적으로 가져옵니다.

# 다대다 매핑
다대다(M:N) 연관관계는 실무에서 거의 사용되지 않는 구성입니다.
다대다 연관관계를 상품과 생산업체의 예로 들자면 한 종류의 상품이 여러 생산업체를 통해 생산될 수 있고,
생산업체 한 곳이 여러 상품을 생산할 수도 있습니다.
다대다 연관관계에서는 각 엔티티에서 서로를 리스트로 가지는 구조가 만들어집니다.
이런 경우에는 교차 엔티티라고 부르는 중간 테이블을 생성해서 다대다 관계를 일대다 또는 다대일 관게로 해소합니다.

# 다대다 단방향 매핑
생산업체에 매핑되는 도메인은 Producer라고 가정하고 작성합니다.

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "producer")
public class Producer extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String code;

    private String name;

    @ManyToMany(fetch = FetchType.EAGER)
    @ToString.Exclude
    private List<Product> products = new ArrayList<>();

    public void addProduct(Product product) {
        products.add(product);
    }
}
```

상품 엔티티에 적용한 것과 같이 다대다 연관관계는 @ManyToMany 어노테이션으로 설정합니다.
리스트로 필드를 가지는 객체에서는 외래키를 가지지 않기 때문에 별도의 @JoinColumn은 설정하지 않아도 됩니다.
이렇게 엔티티를 생성하고 애플리케이션을 실행하면 생산업체 테이블이 생성됩니다.

생산업체 테이블에는 별도의 외래키가 추가되지 않은 것을 볼 수 있습니다.
그리고 데이터베이스에 추가로 중간 테이블이 생성돼 있습니다.

별도의 설정을 하지 않았다면 테이블은 producer_products라는 이름으로 설정되며, 만약 테이블의 이름을 관리하고 싶다면 @ManyToMany 어노테이션 아래에 @JoinTable(name = "이름")의 형식으로 어노테이션을 정의하면 됩니다.
producer_products 테이블의 경우 상품 테이블과 생산업체 테이블에서 id값을 가져와 두 개의 외래키가 설정되는 것을 볼 수 있습니다.
그럼 이러한 연관관계를 테스트하기 위해 다음과 같이 생산업체 엔티티에 대한 리포지토리를 생성해보겠습니다.


```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Producer;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProducerRepository extends JpaRepository<Producer, Long> {
}
```

이 같은 리포지토리를 생성하면 생산업체에 대한 기본적인 데이터베이스 조작이 가능합니다.
그럼 다음과 같이 테스트코드를 작성해서 앞에서 설정한 연관관계가 정상적으로 동작하는지 확인하겠습니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Producer;
import com.springboot.relationship.data.entity.Product;
import org.assertj.core.util.Lists;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ProducerRepositoryTest {

    @Autowired
    ProducerRepository producerRepository;

    @Autowired
    ProductRepository productRepository;

    @Test
    void relationshipTest() {

        Product product1 = saveProduct("동글펜", 500, 1000);
        Product product2 = saveProduct("네모 공책", 152, 1234);
        Product product3 = saveProduct("지우개", 152, 1234);

        Producer producer1 = saveProducer("flature");
        Producer producer2 = saveProducer("wikibooks");

        producer1.addProduct(product1);
        producer1.addProduct(product2);

        producer2.addProduct(product2);
        producer2.addProduct(product3);

        producerRepository.saveAll(Lists.newArrayList(producer1, producer2));

        System.out.println(producerRepository.findById(1L).get().getProducts());
    }

    private Product saveProduct(String name, Integer price, Integer stock) {
        Product product = new Product();
        product.setName(name);
        product.setPrice(price);
        product.setStock(stock);

        return productRepository.save(product);
    }

    private Producer saveProducer(String name) {
        Producer producer = new Producer();
        producer.setName(name);
        return producerRepository.save(producer);
    }
}
```

위 예제에서는 가독성을 위해 리포지토리를 통해 테스트 데이터를 생성하는 부분을 별도 메소드로 구현했습니다.
이 경우 리포지토리를 사용하게 되면 매번 트랜잭션이 끊어져 생산업체 엔티티에서 상품 리스트를 가져오는 작업이 불가능합니다.
이 같은 문제를 해소하기 위해 테스트 메서드에 @Transactional 어노테이션을 지정해 트랜잭션이 유지되도록 구성해서 테스트를 진행합니다.

System.out.println(producerRepository.findById(1L).get().getProducts());
다음 코드 줄의 생산업체 엔티티와 연관관계가 설정된 상품 데이터 리스트를 출력하면 다음과 같은 내용이 출력됩니다.

```
[Product(super=BaseEntity(createdAt=2024-08-02T11:05:13.494866, updatedAt=2024-08-02T11:05:13.494866), number=1, name=동글펜, price=500, stock=1000), Product(super=BaseEntity(createdAt=2024-08-02T11:05:13.578875, updatedAt=2024-08-02T11:05:13.578875), number=2, name=네모 공책, price=152, stock=1234)]
```


앞서

```
producer1.addProduct(product1);
producer1.addProduct(product2);

producer2.addProduct(product2);
producer2.addProduct(product3);
```

연관관계를 설정했기 때문에 정상적으로 생산업체 엔티티에서 상품 리스트를 가져오는 것을 볼 수 있습니다.
앞의 테스트를 통해 테스트 데이터를 생성하면 product 테이블과 producer 테이블에 레코드가 추가되지만 보여지는
내용만으로는 연관관계 설정 여부를 확인하기 어렵습니다.
그 이유는 다대다 연관관계 설정을 통해 생성된 중간 테이블에 연관관계 매핑이 돼 있기 때문입니다.
중간 테이블에 생성된 레코드를 확인하면 다음과 같습니다.

producer_products라는 이름의 중간 테이블에는

```
producer1.addProduct(product1);
producer1.addProduct(product2);

producer2.addProduct(product2);
producer2.addProduct(product3);
```
에서 설정한 연관관계에 맞춰 양 테이블의 기본키를 매핑한 레코드가 생성된 것을 볼 수 있습니다.

# 다대다 양방향 매핑
다대다 단방향 매핑의 개념을 이해했다면 양방향 매핑을 하는 방법은 간단합니다.
상품 엔티티에서 다음과 같이 작성합니다.

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@NoArgsConstructor
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    @OneToOne(mappedBy = "product")
    @ToString.Exclude
    private ProductDetail productDetail;

    @ManyToOne
    @JoinColumn(name = "provider_id")
    @ToString.Exclude
    private Provider provider;


    @ManyToMany
    @ToString.Exclude
    private List<Producer> producers = new ArrayList<>();

    public void addProducer(Producer producer) {
        this.producers.add(producer);
    }
}
```

이 예제에서 

```
@ManyToMany
@ToString.Exclude
private List<Producer> producers = new ArrayList<>();
```
다음은 생산업체에 대한 다대다 연관관계를 설정합니다.
필요에 따라 mappedBy 속성을 사용해 두 엔티티 간 연관관계의 주인을 설정할 수도 있습니다.
이렇게 설정한 후 애플리케이션을 실행하면 데이터베이스의 테이블 구조는 변경되지 않습니다.
중간 테이블이 연관관게를 설정하고 있기 때문입니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Producer;
import com.springboot.relationship.data.entity.Product;
import org.assertj.core.util.Lists;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ProducerRepositoryTest {

    @Autowired
    ProducerRepository producerRepository;

    @Autowired
    ProductRepository productRepository;

    @Test
    void relationshipTest() {

        Product product1 = saveProduct("동글펜", 500, 1000);
        Product product2 = saveProduct("네모 공책", 152, 1234);
        Product product3 = saveProduct("지우개", 152, 1234);

        Producer producer1 = saveProducer("flature");
        Producer producer2 = saveProducer("wikibooks");

        producer1.addProduct(product1);
        producer1.addProduct(product2);

        producer2.addProduct(product2);
        producer2.addProduct(product3);

        producerRepository.saveAll(Lists.newArrayList(producer1, producer2));

        System.out.println(producerRepository.findById(1L).get().getProducts());
    }

    @Test
    @Transactional
    void relationshipTest2() {
        Product product1 = saveProduct("동글펜", 500, 1000);
        Product product2 = saveProduct("네모 공책", 152, 1234);
        Product product3 = saveProduct("지우개", 152, 1234);

        Producer producer1 = saveProducer("flature");
        Producer producer2 = saveProducer("wikibooks");

        producer1.addProduct(product1);
        producer1.addProduct(product2);
        producer2.addProduct(product2);
        producer2.addProduct(product3);

        product1.addProducer(producer1);
        product2.addProducer(producer1);
        product2.addProducer(producer2);
        product3.addProducer(producer2);

        producerRepository.saveAll(Lists.newArrayList(producer1, producer2));
        productRepository.saveAll(Lists.newArrayList(product1, product2, product3));

        System.out.println("products : " + producerRepository.findById(1L).get().getProducts());
        System.out.println("produccers : " + productRepository.findById(1L).get().getProducers());
    }

    private Product saveProduct(String name, Integer price, Integer stock) {
        Product product = new Product();
        product.setName(name);
        product.setPrice(price);
        product.setStock(stock);

        return productRepository.save(product);
    }

    private Producer saveProducer(String name) {
        Producer producer = new Producer();
        producer.setName(name);
        return producerRepository.save(producer);
    }
}
```

여기에 양방향 연관관계 설정을 위해 

```
product1.addProducer(producer1);
product2.addProducer(producer1);
product2.addProducer(producer2);
product3.addProducer(producer2);
```

연관관계 설정 코드를 추가했습니다.
연관관계를 설정하고 아래의 줄처럼

```
System.out.println("products : " + producerRepository.findById(1L).get().getProducts());
System.out.println("produccers : " + productRepository.findById(1L).get().getProducers());
```

각 엔티티에 연관된 엔티티를 출력하면 다음과 같이 정상적으로 출력되는 것을 볼 수 있습니다.

```
products : [Product(super=BaseEntity(createdAt=2024-08-02T17:26:51.403504400, updatedAt=2024-08-02T17:26:51.403504400), number=1, name=동글펜, price=500, stock=1000), Product(super=BaseEntity(createdAt=2024-08-02T17:26:51.465854, updatedAt=2024-08-02T17:26:51.465854), number=2, name=네모 공책, price=152, stock=1234)]
producers : [Producer(super=BaseEntity(createdAt=2024-08-02T17:26:51.469858600, updatedAt=2024-08-02T17:26:51.469858600), id=1, code=null, name=flature)]
```

이렇게 다대다 연관관계를 설정하면 중간 테이블을 통해 연관된 엔티티의 값을 가져올 수 있습니다.
다만 다대다 연관관계에서는 중간 테이블이 생성되기 때문에 예기치 못한 쿼리가 생길 수 있습니다.
즉, 관리하기 힘든 포인트가 발생한다는 문제가 있습니다.
그렇기 때문에 이러한 다대다 연관관계의 한계를 극복하기 위해서는 중간 테이블을 생성하는 대신 일대다 다대일로
연관관계를 맺을 수 있는 중간 엔티티로 승격시켜 JPA에서 관리할 수 있게 생성하는 것이 좋습니다.

# 영속성 전이
영속성 전이 (cascade)란 특정 엔티티의 영속성 상태를 변경할 때 그 엔티티와 연관된 엔티티의 영속성에도 영향을 미쳐 영속성 상태를 변경하는 것을 의미합니다.
예를 들어 @OneTOMany 어노테이션의 인터페이스를 살펴보면 다음과 같습니다.

```java
public @interface OneToMany {
    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};

    FetchType fetch() default FetchType.LAZY;

    String mappedBy() default "";

    boolean orphanRemoval() default false;
}
```

연관관계와 관련된 어노테이션을 보면 위와 같이 cascade()라는 요소를 볼 수 있습니다.
이 어노테이션은 영속성 전이를 설정하는데 활용됩니다.
cascase() 요소와 함께 사용하는 영속성 전이 타입은 다음과 같습니다.

영속성 전이 타입의 종류

| 종류     | 설명                                                                                         |
|----------|----------------------------------------------------------------------------------------------|
| ALL      | 모든 영속 상태 변경에 대해 영속성 전이를 적용                                                 |
| PERSIST  | 엔티티가 영속화할 때 연관된 엔티티도 함께 영속화                                              |
| MERGE    | 엔티티를 영속성 컨텍스트에 병합할 때 연관된 엔티티도 병합                                     |
| REMOVE   | 엔티티를 제거할 때 연관된 엔티티도 제거                                                       |
| REFRESH  | 엔티티를 새로고침할 때 연관된 엔티티도 새로고침                                               |
| DETACH   | 엔티티를 영속성 컨텍스트에서 제외하면 연관된 엔티티도 제외                                    |

여기서 알 수 있듯이 영속성 전이에 사용되는 타입은 엔티티 생명주기와 연관이 있습니다.
한 엔티티가 표와 같이 casecade 요소의 값으로 주어진 영속 상태의 변경이 일어나면 매핑으로 연관된
엔티티에도 동일한 동작이 일어나도록 전이를 발생시키는 것입니다.
위의 코드를 보면 casecade() 요소의 리턴 타입은 배열 형식인 것을 볼 수 있습니다.
이 말은 개발자가 사용하고자 하는  cascade 타입을 골라 각 상황에 적용할 수 있다는 것입니다.

# 영속성 전이 적용
이제 영속성 전이를 적용해 보겠습니다.
여기서 사용할 엔티티는 상품 엔티티와 공급업체 엔티티입니다.
예를 들어, 한 가게가 새로운 공급업체와 계약하며 몇 가지 새 상품을 입고시키는 상황에 어떻게 영속성 전이가 적용되는지 살펴보겠습니다.
우선 엔티티를 데이터베이스에 추가하는 경우로 영속성 전이 타입으로 PERSIST를 지정하겠습니다.
먼저 공급업체 엔티티에 다음과 같이 영속성전이 타입을 설정합니다.

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "provider", fetch = FetchType.EAGER, cascade = CascadeType.PERSIST, orphanRemoval = true)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();
}
```

영속성 전이 타입을 설정하기 위해서는 @OneTOMany 어노테이션 속성을 활용합니다.
이렇게 설정한 후에 다음과 같이 코드를 작성합니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.Provider;
import org.assertj.core.util.Lists;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class ProviderRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    ProviderRepository providerRepository;

    @Test
    void relationshipTest1() {
        // 테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 물산");

        providerRepository.save(provider);

        Product product = new Product();
        product.setName("가위");
        product.setPrice(5000);
        product.setStock(500);
        product.setProvider(provider);

        productRepository.save(product);

        System.out.println("product : " + productRepository.findById(1L).orElseThrow(RuntimeException::new));

        System.out.println("provider : " + productRepository.findById(1L).orElseThrow(RuntimeException::new).getProvider());
    }

    @Test
    void relationshipTest() {
        //테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 상사");

        providerRepository.save(provider);

        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(2000);
        product1.setStock(100);
        product1.setProvider(provider);

        Product product2 = new Product();
        product2.setName("가방");
        product2.setPrice(20000);
        product2.setStock(200);
        product2.setProvider(provider);

        Product product3 = new Product();
        product3.setName("노트");
        product3.setPrice(3000);
        product3.setStock(100);
        product3.setProvider(provider);

        productRepository.save(product1);
        productRepository.save(product2);
        productRepository.save(product3);

        List<Product> productList = providerRepository.findById(provider.getId()).get().getProductList();

        for (Product product : productList) {
            System.out.println(product);
        }
    }

    @Test
    void cascadeTest() {
        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.save(provider);
    }

    private Provider saveProvider(String name) {
        Provider provider = new Provider();
        provider.setName(name);
        return providerRepository.save(provider);
    }

    private Product saveProduct(String name, Integer price, Integer stock) {
        Product product = new Product();
        product.setName(name);
        product.setPrice(price);
        product.setStock(stock);
        return productRepository.save(product);
    }

    @Test
    void orphanRemovalTest() {

        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.saveAndFlush(provider);

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);

        Provider foundProvider = providerRepository.findById(1L).get();

        // 리스트가 비어 있지 않은지 확인
        if (!foundProvider.getProductList().isEmpty()) {
            foundProvider.getProductList().remove(0);
        }

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);
    }
}
```


테스트를 수행하기 위해 다음과 같이 공급업체 하나와 상품 객체를 3개 생성합니다.
영속성 전이를 테스트하기 위해 객체에는 영속화 작업을 수행하지 않고

```
product1.setProvider(provider);
product2.setProvider(provider);
product3.setProvider(provider);

provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));
```
다음과 같이 연관관계만 설정합니다.
영속성 전이가 수행되는 부분은 제일 마지막줄 코드입니다.
마지막줄 코드에서 데이터베이스에 저장하는 쿼리를 보면 다음과 같습니다.

```
Hibernate: 
    insert 
    into
        provider
        (created_at, name, updated_at) 
    values
        (?, ?, ?) 
    returning id
Hibernate: 
    insert 
    into
        product
        (created_at, name, price, provider_id, stock, updated_at) 
    values
        (?, ?, ?, ?, ?, ?) 
    returning number
Hibernate: 
    insert 
    into
        product
        (created_at, name, price, provider_id, stock, updated_at) 
    values
        (?, ?, ?, ?, ?, ?) 
    returning number
Hibernate: 
    insert 
    into
        product
        (created_at, name, price, provider_id, stock, updated_at) 
    values
        (?, ?, ?, ?, ?, ?) 
    returning number
```

지금까지는 엔티티를 데이터베이스에 저장하기 위해 각 엔티티를 저장하는 코드를 작성해야 했습니다.
하지만 영속성 전이를 사용하면 부모 엔티티티가 되는 Provider 엔티티만 저장하면 코드에 작성돼 있는
Cascade.PERSIST에 맞춰 상품 엔티티도 함께 저장할 수 있습니다.

이처럼 특정 상황에 맞춰 영속성 전이 타입을 설정하면 영속 상태의 변화에 따라 연관된 엔티티들의 동작도 함께 수행할 수 있어 개발의 생산성이 높아집니다.
다만 자동 설정으로 동작하는 코드들이 정확히 어떤 영향을 미치는지 파악할 필요가 있습니다.
예를 들어, REMOVE와 REMOVE를 포함하는 ALL 같은 타입을 무분별하게 사용하면 연관된 엔티티가 의도치 않게
모두 삭제될 수 있기 때문에 다른 타입보다 더욱 사이드 이펙트(side effect)를 고려해서 사용해야 합니다.

# 고아 객체
JPA에서 고아(Orphan)란 부모 엔티티와 연관관계가 끊어진 엔티티를 의미합니다.
JPA에는 이러한 고아 객체를 자동으로 제거하는 기능이 있습니다.
물론 자식 엔티티가 다른 엔티티와 연관관계를 가지고 있다면 이 기능은 사용하지 않는 것이 좋습니다.
현재 예제에서 사용되는 상품 엔티티는 다른 엔티티와 연관관계가 많이 설정돼 있지만 그 부분은 예외로 두고 테스트를 진행하겠습니다.

고아 객체를 제거하는 기능을 사용하기 위해서는 공급업체 엔티티를 다음과 같이 작성합니다.

```java
package com.springboot.relationship.data.entity;

import jakarta.persistence.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "provider", fetch = FetchType.EAGER, cascade = CascadeType.PERSIST, orphanRemoval = true)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();
}
```

orphanRemoval = true 속성은 고아 객체를 제거하는 기능입니다.
이 설정을 추가하고 정상적으로 동작하는지 확인하기 위해 다음과 같이 테스트 코드를 작성합니다.


```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.Provider;
import org.assertj.core.util.Lists;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class ProviderRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    ProviderRepository providerRepository;

    @Test
    void relationshipTest1() {
        // 테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 물산");

        providerRepository.save(provider);

        Product product = new Product();
        product.setName("가위");
        product.setPrice(5000);
        product.setStock(500);
        product.setProvider(provider);

        productRepository.save(product);

        System.out.println("product : " + productRepository.findById(1L).orElseThrow(RuntimeException::new));

        System.out.println("provider : " + productRepository.findById(1L).orElseThrow(RuntimeException::new).getProvider());
    }

    @Test
    void relationshipTest() {
        //테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("oo 상사");

        providerRepository.save(provider);

        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(2000);
        product1.setStock(100);
        product1.setProvider(provider);

        Product product2 = new Product();
        product2.setName("가방");
        product2.setPrice(20000);
        product2.setStock(200);
        product2.setProvider(provider);

        Product product3 = new Product();
        product3.setName("노트");
        product3.setPrice(3000);
        product3.setStock(100);
        product3.setProvider(provider);

        productRepository.save(product1);
        productRepository.save(product2);
        productRepository.save(product3);

        List<Product> productList = providerRepository.findById(provider.getId()).get().getProductList();

        for (Product product : productList) {
            System.out.println(product);
        }
    }

    @Test
    void cascadeTest() {
        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.save(provider);
    }

    private Provider saveProvider(String name) {
        Provider provider = new Provider();
        provider.setName(name);
        return providerRepository.save(provider);
    }

    private Product saveProduct(String name, Integer price, Integer stock) {
        Product product = new Product();
        product.setName(name);
        product.setPrice(price);
        product.setStock(stock);
        return productRepository.save(product);
    }

    @Test
    @Transactional
    void orphanRemovalTest() {

        Provider provider = saveProvider("새로운 공급업체");

        Product product1 = saveProduct("상품1", 1000, 1000);
        Product product2 = saveProduct("상품2", 500, 1500);
        Product product3 = saveProduct("상품3", 750, 500);

        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.saveAndFlush(provider);

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);

        Provider foundProvider = providerRepository.findById(1L).get();

        // 리스트가 비어 있지 않은지 확인
        if (!foundProvider.getProductList().isEmpty()) {
            foundProvider.getProductList().remove(0);
        }

        providerRepository.findAll().forEach(System.out::println);
        productRepository.findAll().forEach(System.out::println);
    }
}
```

먼저 테스트 데이터를 생성하고

```
product1.setProvider(provider);
product2.setProvider(provider);
product3.setProvider(provider);

provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));
```

다음과 같이 연관관계 매핑을 수행합니다.
연관관계가 매핑된 각 엔티티들을 저장한후 

```
providerRepository.findAll().forEach(System.out::println);
productRepository.findAll().forEach(System.out::println);
```

다음과 같이 각 엔티티를 출력하면 다음과 같이 생성한 공급업체 엔티티 1개, 상품 엔티티 3개가 출력되는 것을 볼 수 있습니다. 

```
Hibernate: 
    select
        p1_0.id,
        p1_0.created_at,
        p1_0.name,
        p1_0.updated_at,
        pl1_0.provider_id,
        pl1_0.number,
        pl1_0.created_at,
        pl1_0.name,
        pl1_0.price,
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.updated_at,
        pl1_0.stock,
        pl1_0.updated_at 
    from
        provider p1_0 
    left join
        product pl1_0 
            on p1_0.id=pl1_0.provider_id 
    left join
        product_detail pd1_0 
            on pl1_0.number=pd1_0.product_number 
    where
        p1_0.id=?
Hibernate: 
    select
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.updated_at,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at 
    from
        product p1_0 
    left join
        product_detail pd1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        p1_0.number=?
Hibernate: 
    select
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.updated_at,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at 
    from
        product p1_0 
    left join
        product_detail pd1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        p1_0.number=?
Hibernate: 
    select
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.updated_at,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at 
    from
        product p1_0 
    left join
        product_detail pd1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        p1_0.number=?
Hibernate: 
    select
        p1_0.id,
        p1_0.created_at,
        p1_0.name,
        p1_0.updated_at 
    from
        provider p1_0
Hibernate: 
    select
        pl1_0.provider_id,
        pl1_0.number,
        pl1_0.created_at,
        pl1_0.name,
        pl1_0.price,
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        pd1_0.updated_at,
        pl1_0.stock,
        pl1_0.updated_at 
    from
        product pl1_0 
    left join
        product_detail pd1_0 
            on pl1_0.number=pd1_0.product_number 
    where
        pl1_0.provider_id=?
Provider(super=BaseEntity(createdAt=2024-08-02T17:59:41.849487, updatedAt=2024-08-02T17:59:41.849487), id=1, name=새로운 공급업체)
Hibernate: 
    select
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p1_0.provider_id,
        p1_0.stock,
        p1_0.updated_at 
    from
        product p1_0
Hibernate: 
    select
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at,
        pd1_0.updated_at 
    from
        product_detail pd1_0 
    left join
        product p1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        pd1_0.product_number=?
Hibernate: 
    select
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at,
        pd1_0.updated_at 
    from
        product_detail pd1_0 
    left join
        product p1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        pd1_0.product_number=?
Hibernate: 
    select
        pd1_0.id,
        pd1_0.created_at,
        pd1_0.description,
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p2_0.id,
        p2_0.created_at,
        p2_0.name,
        p2_0.updated_at,
        p1_0.stock,
        p1_0.updated_at,
        pd1_0.updated_at 
    from
        product_detail pd1_0 
    left join
        product p1_0 
            on p1_0.number=pd1_0.product_number 
    left join
        provider p2_0 
            on p2_0.id=p1_0.provider_id 
    where
        pd1_0.product_number=?
Product(super=BaseEntity(createdAt=2024-08-02T17:59:41.930800, updatedAt=2024-08-02T17:59:41.930800), number=1, name=상품1, price=1000, stock=1000)
Product(super=BaseEntity(createdAt=2024-08-02T17:59:41.942322, updatedAt=2024-08-02T17:59:41.942322), number=2, name=상품2, price=500, stock=1500)
Product(super=BaseEntity(createdAt=2024-08-02T17:59:41.945321, updatedAt=2024-08-02T17:59:41.945321), number=3, name=상품3, price=750, stock=500)
```

그리고 고아 객체를 생성하기 위해 생성한 공급업체 엔티티를 가져온 후 첫 번째로 매핑돼 있는 상품 엔티티의 연관관계를 제거했습니다.

```
Provider foundProvider = providerRepository.findById(1L).get();

// 리스트가 비어 있지 않은지 확인
if (!foundProvider.getProductList().isEmpty()) {
  foundProvider.getProductList().remove(0);
}
```

그리고 전체 코드를 수행하면 다음과 같이 연관관계가 끊긴 상품의 엔티티가 제거된 것을 확인할 수 있습니다.

```
Hibernate: 
    select
        p1_0.id,
        p1_0.created_at,
        p1_0.name,
        p1_0.updated_at 
    from
        provider p1_0
Provider(super=BaseEntity(createdAt=2024-08-02T18:06:21.814993, updatedAt=2024-08-02T18:06:21.814993), id=1, name=새로운 공급업체)
Hibernate: 
    select
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p1_0.provider_id,
        p1_0.stock,
        p1_0.updated_at 
    from
        product p1_0
Product(super=BaseEntity(createdAt=2024-08-02T18:06:21.932027400, updatedAt=2024-08-02T18:06:21.959213500), number=1, name=상품1, price=1000, stock=1000)
Product(super=BaseEntity(createdAt=2024-08-02T18:06:21.940686, updatedAt=2024-08-02T18:06:21.963727700), number=2, name=상품2, price=500, stock=1500)
Product(super=BaseEntity(createdAt=2024-08-02T18:06:21.942692200, updatedAt=2024-08-02T18:06:21.963727700), number=3, name=상품3, price=750, stock=500)
Hibernate: 
    select
        p1_0.id,
        p1_0.created_at,
        p1_0.name,
        p1_0.updated_at 
    from
        provider p1_0
Provider(super=BaseEntity(createdAt=2024-08-02T18:06:21.814993, updatedAt=2024-08-02T18:06:21.814993), id=1, name=새로운 공급업체)
Hibernate: 
    delete 
    from
        product 
    where
        number=?
Hibernate: 
    select
        p1_0.number,
        p1_0.created_at,
        p1_0.name,
        p1_0.price,
        p1_0.provider_id,
        p1_0.stock,
        p1_0.updated_at 
    from
        product p1_0
Product(super=BaseEntity(createdAt=2024-08-02T18:06:21.940686, updatedAt=2024-08-02T18:06:21.963727700), number=2, name=상품2, price=500, stock=1500)
Product(super=BaseEntity(createdAt=2024-08-02T18:06:21.942692200, updatedAt=2024-08-02T18:06:21.963727700), number=3, name=상품3, price=750, stock=500)
```

출력 결과에서 실제로 연관관계가 제거되면서 하이버네이트에서는 상태 감지를 통해 삭제하는 쿼리가 수행되는 것을 볼 수 있습니다.

# 정리
이번 장에서는 연관관계를 설정하는 방법과 영속성 전이라는 개념을 알아봤습니다.
JPA를 사용할 때 영속이라는 개념은 매우 중요합니다.
코드를 통해 편리하게 데이터베이스에 접근하기 위해서는 중간에서 엔티티의 상태를 조율하는 영속성 컨텍스트가 어떻게 동작하는지 이해해야 합니다.
이 과정에서 하이버네이트를 직접 사용하는 것과 Spring Data Jpa를 사용하는 데는 차이가 있습니다.
따라서 이 책에서는 다루지 않았지만 하이버네이트만 사용하는 JPA도 함께 공부하는 것을 권장합니다.
JPA 자체를 그대로 다뤄보는 경험을 해보면 DAO와 리포지토리의 차이에 대해서도 더 알 수 있게 되고 스프링 부트 JPA에 대해서도 좀 더 폭넓게 이해할 수 있게 됩니다.
