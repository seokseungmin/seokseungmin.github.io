---
title: "스프링 핵심가이드 8주차"
excerpt: "[제로베이스] 스프링 핵심가이드"

categories:
  - Spring
tags:
  - [Spring]

permalink: /categories/Spring8Week/ # url

toc: true
toc_sticky: true

date: 2024-08-05
last_modified_at: 2024-08-05
---

[스프링부트 핵심가이드 프로젝트 따라하면서 짠 코드](https://github.com/seokseungmin/actuator)<br>

본 게시글은 '스프링 부트 핵심 가이드' 책의 내용을 정리한 것입니다.<br>
저자 : 장정우<br>
출판사 : 위키북스<br>

# 13장 서비스의 인증과 권한 부여

애플리케이션을 개발하다 보면 인증과 인가 등의 보안 기능을 추가해야 할 때가 있습니다.
이번 장에서는 보안과 관련된 용어와 개념을 알아보고 스프링에 보안을 적용할 때 사용하는 스프링 시큐리티(Spring Security)에 대해 알아보겠습니다.
지금까지 실습한 애플리케이션은 화면이 없는 무상태 REST 애플리케이션이기 때문에 이번 장에서는 로그인을 통한 일반적인
인증과 인가 방식이 아닌 매 요청마다 토근값을 활용하는 보안 기법을 알아보겠습니다.

# 보안 용어 이해
스프링 시큐리티를 활용하려면 먼저 보안과 관련된 용어를 알아두는 것이 중요합니다.
그러므로 스프링 시큐리티를 배우기에 앞서 보안과 관련된 용어를 간단하게 설명하겠습니다.

### 인증
인증(authentication)은 사용자가 누구인지 확인하는 단계를 의미합니다.
인증의 대표적인 예로 '로그인'이 있습니다. 로그인은 데이터베이스에 등록된 아이디와 패스워드를 사용자가 입력한 아이디와
비밀번호와 비교해서 일치 여부를 확인하는 과정입니다.
로그인에 성공하면 애플리케이션 서버는 응답으로 사용자에게 토큰(token)을 전달합니다.
로그인에 실패한 사용자는 토큰을 전달받지 못해 원하는 리소스에 접근 할 수 없게 됩니다.

### 인가
인가(authorization)는앞에서 설명한 인증을 통해 검증된 사용자가 애플리케이션 내부의 리소스에 접근할 때
사용자가 해당 리소스에 접근할 권리가 있는지를 확인하는 과정을 의미합니다.
예를 들어, 로그인한 사용자가 특정 게시판에 접근해서 글을 보려고 하는 경우 게시판 접근 등급을 확인해 접근을 허가하거나 거부하는 것이
대표적인 인가의 사례입니다.

일반적으로 사용자가 인증 단계에서 발급받은 토근은 인가 내용을 포함하고 있으며, 사용자가 리소스에 접근하면서
토근을 함께 전달하면 애플리케이션 서버는 토근을 통해 권한 유무등을 확인해 인가를 수행합니다.

# 접근 주체
접근 주체(principal)는 말 그대로 애플리케이션의 기능을 사용하는 주체를 의미합니다.
접근 주체는 사용자가 될 수도 있고, 디바이스, 시스템 등이 될 수도 있습니다.
애플리케이션은 앞서 소개한 인증 과정을 통해 접근 주체가 신뢰할 수 있는지 확인하고, 인가 과정을 통해 접근 주체에게
부여된 권한을 확인하는 과정 등을 거칩니다.


# 스프링 시큐리티의 동작 구조
스프링 시큐리티는 서블릿 필터(Servlet Filter)를 기반으로 동작하며
DispatcherServlet 앞에 필터가 배치돼 있습니다.

![new repo](/assets/images/posts_img/Spring/ServletFilter.png)

필터체인(FilterChain)은 서블릿 컨테이너에서 관리하는 ApplicationFilterChain을 의미합니다.
클라이언트에서 애플리케이션으로 요청을 보내면 서블릿 컨테이너는 URI를 확인해서 필터와 서블릿을 매핑합니다.
스프링 시큐리티는 사용하고자 하는 필터체인을 서블릿 컨테이너의 필터 사이에서 동작시키기 위해 DelegatingFilterProxy를 사용합니다.

![new repo](/assets/images/posts_img/Spring/FilterChainProxy.png)

DelegatingFilterProxy는 서블릿 컨테이너의 생명주기와 스프링 애플리케이션 컨텍스트(Application Context) 사이에서
다리 역할을 수행하는 필터 구현체입니다.
표준 서블릿 필터를 구현하고 있으며, 역할을 위임할 필터체인 프록시(FilterChainProxy)를 내부에 가지고 있습니다.
필터체인 프록시는 스프링 부트의 자동 설정에 의해 자동 생성됩니다.

필터체인 프록시는 스프링 시큐리티에서 제공하는 필터로서 보안 필터체인(SecurityFilterChain)을 통해 많은
보안 필터(Security Filter)를 사용할 수 있습니다. 필터체인 프록시에서 사용할 수 있는 보안 필터체인은
List 형식으로 담을 수 있게 설정돼 있어 URI 패턴에 따라 특정 보안필터 체인을 선택해서 사용하게 됩니다.

보안필터 체인에서 사용하는 필터는 여러 종류가 있으며, 각 필터마다 실행되는 순서가 다릅니다.
공식 문서에서 소개하는 필터의 실행 순서는 다음과 같습니다.

```
ChannelProcessingFilter
WebAsyncManagerIntegrationFilter
SecurityContextPersistenceFilter
HeaderWriterFilter
CorsFilter
CsrfFilter
LogoutFilter
OAuth2AuthorizationRequestRedirectFilter
Saml2WebSsoAuthenticationRequestFilter
X509AuthenticationFilter
AbstractPreAuthenticatedProcessingFilter
CasAuthenticationFilter
OAuth2LoginAuthenticationFilter
Saml2WebSsoAuthenticationFilter
UsernamePasswordAuthenticationFilter
DefaultLoginPageGeneratingFilter
DefaultLogoutPageGeneratingFilter
ConcurrentSessionFilter
DigestAuthenticationFilter
BearerTokenAuthenticationFilter
BasicAuthenticationFilter
RequestCacheAwareFilter
SecurityContextHolderAwareRequestFilter
JaasApiIntegrationFilter
RememberMeAuthenticationFilter
AnonymousAuthenticationFilter
OAuth2AuthorizationCodeGrantFilter
SessionManagementFilter
ExceptionTranslationFilter
AuthorizationFilter
SwitchUserFilter
```

보안 필터체인은 WebSecurityConfigurerAdapter 클래스를 상속받아 설정할 수 있습니다.
앞에서 이야기한 것처럼 필터체인 프록시는 여러 보안 필터체인을 가질 수 있는데,
여러 보안 필터체인을 만들기 위해서는 WebSecurityConfigurerAdapter 클래스를 상속받는 클래스를 여러 개 생성하면 됩니다.
이때 WebSecurityConfigurerAdapter 클래스에는 #Order 어노테이션을 통해 우선순위가 지정돼 있는데,
2개 이상의 클래스를 생성했을 때 똑같은 설정으로 우선순위가 100이 설정돼 있으면 예외가 발생하기 때문에 상속받은 클래스에서
@Order 어노테이션을 지정해 순서를 정의하는 것이 중요합니다.
별도의 설정이 없다면 스프링 시큐리티에서는 다음과 같이 SecurityFilterChain에서 사용하는 필터중 UsernamePasswordAuthenticationFilter를 통해 인증을 처리합니다.

![new repo](/assets/images/posts_img/Spring/UsernamePasswordAuthenticationFilter.png)

위 그림의 인증 수행 과정을 설명하면 다음과 같습니다.
클라이언트로부터 요청을 받으면 서블릿 필터에서 SecurityFilterChain에서으로 작업이 위임되고
그중 UsernamePasswordAuthenticationFilter(위 그림에서 AuthenticationFilter에 해당)에서 인증을 처리합니다.
AuthenticationFilter는 요청 객체(HttpServletRequest)에서 username과 password를 추출해서 토큰을 생성합니다.
그러고 나서 AuthenticationManager에게 토큰을 전달합니다. AuthenticationManager는 인터페이스이며, 일반적으로 사용되는 구현체는 ProviderManager입니다.
ProviderManager는 인증을 위해 AuthenticationProvider로 토큰을 전달합니다.
AuthenticationProvider는 토근의 정보를 UserDetailService에 전달합니다.
UserDetailService는 전달받은 정보를 통해 데이터베이스에서 일치하는 사용자를 찾아 UserDetails 객체를 생성합니다.
생성된 UserDetails 객체는 AuthenticationProvider로 전달되며, 해당 Provider에서 인증을 수행하고 성공하게 되면
ProviderManager로 권한을 담은 토큰을 전달합니다.
ProviderManager는 검증된 토근을 AuthenticationFilter로 전달합니다.
AuthenticationFilter는 검증된 토큰을 SecurityContextHolder에 있는 SecurityContext에 저장합니다.
위 과정에서 사용된 UsernamePasswordAuthenticationFilter는 접근 권한을 확인하고 인증이 실패할 경우 로그인 폼이라는
화면을 보내는 역할을 수행합니다.
이 책에서 실습 중인 프로젝트는 화면이 없는 RESTful 애플리케이션이기 때문에 다른 필터에서 인증 및 인가 처리를 수행해야 합니다.
이 책에서는 JWT 토큰을 사용해 인증을 수행할 예정이라 JWT와 관련된 필터를 생성하고 UsernamePasswordAuthenticationFilter 앞에 배치해서 먼저 인증을 수행할수 있게 설정하겠습니다.
 
# JWT
JWT(JSON Web TOken)는 당사자 간에 정보를 JSON 형태로 안전하게 전송하기 위한 토큰입니다.
JWT는 URL로 이용할 수 있는 문자열로만 구성돼 있으며, 디지털 서명이 적용돼 있어 신회할 수 있습니다.
JWT는 주로 서버와의 통신에서 권한 인가를 위해 사용됩니다.
URL에서 사용할 수 있는 문자열로만 구성돼 있기 때문에 HTTP 구성요소 어디든 위치할 수 있습니다.

### JWT의 구조
JWT는 점('.')으로 구분된 아래의 세 부분으로 구성됩니다.

- 헤더(Header)
- 내용(Payload)
- 서명(Signature)

따라서 JWT는 일반적으로 다음과 같은 형식을 띠고 있습니다.

xxxxxx.yyyyy.zzzzz
헤더.내용.서명

### 헤더
JWT의 헤더는 검증과 관련된 내용을 담고 있습니다.
헤더는 다음과 같이 두 가지 정보를 포함하고 있는데, 바로 alg와 typ 속성입니다.

```
{
  "alg": "HS256"
  "typ": "JWT"
}
```

alg 속성에서는 해싱 알고리즘을 지정합니다.
해싱 알고리즘은 보통 SHA256 또는 RSA를 사용하며, 토큰을 검증할 때 사용되는 서명 부분에서 사용됩니다.
위 예제에 작성돼 있는 HS256은  'HMACSHA256' 알고리즘을 사용한다는 의미입니다.
그리고 typ 속성에는 토큰의 타입을 지정합니다.
이렇게 완성된 헤더는 Base64Url 형식으로 인코딩돼 사용됩니다.

# 내용
JWT의 내용에는 토큰에 담는 정보를 포함합니다. 이곳에 포함된 속성들은 클레임(Claim)이라 하며, 크게 세 가지로 분류됩니다.

- 등록된 클레임(Registered Claims)
- 공개 클레임(Public Claims)
- 비공개 클레임(Private Claims)
 
등록된 클레임은 필수는 아니지만 토큰에 대한 정보를 담기 위해 이미 이름이 정해져 있는 클레임을 뜻합니다.
등록된 클레임은 다음과 같이 정의돼 있습니다.

- iss : JWT의 발급자(Issuer) 주체를 나타냅니다. iss의 값은 문자열이나 URL를 포함하는 대소문자를 구분하는 문자열입니다.
- sub: JWT의 제목(Subject)입니다.
- aud: JWT의 수신인(Audience)입니다. JWT를 처리하려는 각 주체는 해당 값으로 자신을 식별해야 합니다.
요청을 처리하는 주체가 'aud' 값으로 자신을 식별하기 않으면 JWT는 거부됩니다.
- exp: JWT의 만료시간(Expiration)입니다. 시간은  NumericDate 형식으로 지정해야 합니다.
- nbf: 'Not Before'를 의미합니다.
- lat: JWT가 발급된 시간(Issued at)입니다.
- jti: JWT의 식별자(JWT ID)입니다. 주로 정복 처리를 방지하기 위해 사용됩니다.

공개 클레임은 키 값을 마음대로 정의할 수 있습니다. 다만 충돌이 발생하지 않을 이름으로 설정해야 합니다.
비공개 클레임은 통신 간에 상호 합의되고 등록된 클레임과 공개된 클레임이 아닌 클레임을 의미합니다.
내용의 예는 다음과 같습니다.

JWT 사용예시

```
{
  "sub" : "wikibooks payload"
  "exp" : "1602076408"
  "userId" : "wikibooks"
  "username" : "flature"
}
```

이렇게 완성된 내용은 Base64Url 형식으로 인코딩되어 사용됩니다.

# 서명
JWT의 서명 부분은 인코딩된 헤더, 인코딩된 내용, 비밀키, 헤더의 알고리즘 속성값을 가져와 생성됩니다.
예를 들어, HMAC SHA256 알고리즘을 사용해서 서명을 생성한다면 다음과 같은 방식으로 생성됩니다.

### 서명 생성 방식

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

서명은 토큰의 값들을 포함해서 암호화하기 때문에 메시지가 도중에 변경되지 않았는지 확인 할 때 사용됩니다.

# JWT 디버거 사용하기
JWT 공식 사이트에서는 더욱 쉽게 JWT를 생성해볼 수 있습니다.
웹 브라우저에서 다음 URL로 접속

https://jwt.io/#debugger-io

접속해서 화면을 보면 Encoded와 Decoded로 나눠져 있으며, 양측의 내용이 일치하는지 사이트에서 확인할 수도 있고
Decoded의 내용을 변경하면 Encoded의 콘텐츠가 자동으로 반영됩니다.

# 스프링 시큐리티와 JWT 적용
이제 애플리케이션에 스프링 시큐리티와 JWT를 적용해보겠습니다.
이전 장에서 사용했던 기본 프로젝트 틀을 가져가기 위해 7장에서 사용한 프로젝트 코드를 그대로 가져와 사용합니다.
인증과 인가 코드를 작성하기 위해 의존성을 추가합니다.

```
    // 시큐리티
    implementation 'org.springframework.boot:spring-boot-starter-security'

    // JWT      ------------------------------------------------------------
    implementation 'io.jsonwebtoken:jjwt-api:0.11.2'
    implementation 'io.jsonwebtoken:jjwt-impl:0.11.2'
    implementation 'io.jsonwebtoken:jjwt-gson:0.11.2'
```

스프링 시큐리티는 기본적으로 UsernamePasswordAuthenticationFilter를 통해 인증을 수행하도록 구성돼 있습니다.
참고로 이 필터에서는 인증이 실패하면 로그인 폼이 포함된 화면을 전달하게 되는데, 이 책의 실습 프로젝트에는 이러한 화면이 없습니다.
따라서 JWT를 사용하는 인증 필터를 구현하고 UsernamePasswordAuthenticationFilter 앞에 인증 필터를 배치해서
인증 주체를 변경하는 작업을 수행하는 방식으로 구성하겠습니다.

# UserDatails와 UserDetailsService 구현
먼저 다음과 같이 사용자 정보를 담는 엔티티를 생성합니다.

```java
package com.springboot.security.data.entity;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.persistence.*;
import lombok.*;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.stream.Collectors;

@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@Entity
public class User extends BaseEntity implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String uid;

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private String name;

    @ElementCollection(fetch = FetchType.EAGER)
    @Builder.Default
    private List<String> roles = new ArrayList<>();


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        // 계정이 가지고 있는 권한 목록 반환
        return this.roles
                .stream()
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());

    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public String getUsername() {
        // 계정의 이름(아이디)을 반환
        return this.uid;
    }


    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonExpired() {
        // 계정이 만료됐는지 반환
        return true; // 만료되지 않았음을 의미
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonLocked() {
        // 계정이 잠겨있는지 반환
        return true; // 잠겨있지 않음을 의미
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isCredentialsNonExpired() {
        // 비밀번호가 만료됐는지 반환
        return true; // 만료되지 않았음을 의미
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isEnabled() {
        // 계정이 활성화 되어 있는지 반환
        return true; // 활성화 상태임을 의미
    }
}
```

User 엔티티는 UserDetails 인터페이스를 구현하고 있습니다.
UserDetails는 UserDetails를 통해 입력된 로그인 정보를 가지고 데이터베이스에서 사용자 정보를 가져오는
역할을 수행합니다. UserDetails 인터페이스는 다음과 같은 메서드를 가지고 있습니다.

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.security.core.userdetails;

import java.io.Serializable;
import java.util.Collection;
import org.springframework.security.core.GrantedAuthority;

public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    default boolean isAccountNonExpired() {
        return true;
    }

    default boolean isAccountNonLocked() {
        return true;
    }

    default boolean isCredentialsNonExpired() {
        return true;
    }

    default boolean isEnabled() {
        return true;
    }
}
```

각 메서드의 용도를 정리하면 다음과 같습니다.
getAuthorities(): 계정이 가지고 있는 권한 목록을 리턴합니다.
getPassword(): 계정의 비밀번호를 리턴합니다.
getUsername(): 계정의 이름(아이디)을 반환.
isAccountNonExpired(): 계정이 만료됐는지 반환. true는 만료되지 않았다는 의미.
isAccountNonLocked(): 계정이 잠겨있는지 반환. true는 잠기지 않았다는 의미.
isCredentialsNonExpired(): 비밀번호가 만료됐는지 반환. true는 만료되지 않았음을 의미.
isEnabled(): 계정이 활성화 되어 있는지 반환. true는 활성화 상태임을 의미.

이번 예제에서는 계정의 상태 변경은 다루지 않을 게정이므로 true로 리턴합니다.
이 엔티티는 앞으로 토큰을 생성할 때 토큰의 정보로 사용될 정보와 권한 정보를 갖게 됩니다.
이번에는 앞에서 살펴본 엔티티를 조회하는 기능을 구현하기 위해 리포지토리와 서비스를 구현하겠습니다.
리포지토리 구현은 다음과 같습니다.

```java
package com.springboot.security.data.repository;

import com.springboot.security.data.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {

    User getByUid(String uid);

}
```

UserRepository를 작성하는 것은 기존에 리포지토리를 작성하던 방법과 동일합니다.
JpaRepository를 상속받고 User 엔티티에 대해 설정하면 됩니다. 그리고 현재 ID 값은 인덱스 값이기 때문에 id 값을 토큰 생성 정보로
사용하기 위해 getByUid() 메서드를 생성합니다.
다음과 같이  UserDetailsServiceImpl을 생성합니다.

```java
package com.springboot.security.service.impl;

import com.springboot.security.data.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Slf4j
@RequiredArgsConstructor
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        log.info("[loadByUsername] loadUserByUsername 수행 . username = {} ", username);
        return userRepository.getByUid(username);
    }

}
```

UserDetails는 스프링 시큐리티에서 제공하는 개념으로, UserDetails의 username은 각 사용자를 구분할 수 있는 ID를 의미합니다.
username을 가지고 UserDetails 객체를 리턴하게끔 정의돼 있는데, UserDetails의 구현체로 User 엔티티를 생성했기 때문에 user 객체를 리턴하게씀 구현한 것입니다.

### JwtTokenProvider 구현
이제 JWT 토큰을 생성하는데 필요한 정보를 UUserDetails에서 가져올 수 있기 때문에 JWT 토큰을 생성하는 TokenProvider를 생성합니다.

```java
package com.springboot.security.config.security;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;
import jakarta.servlet.http.HttpServletRequest;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.List;

@Slf4j
@Component
@RequiredArgsConstructor
public class JwtTokenProvider {

    private final UserDetailsService service;
    private final long tokenValidMillisecond = 1000L * 60 * 60;
    private Key key = Keys.secretKeyFor(SignatureAlgorithm.HS256);

    public String createToken(String userUid, List<String> roles) {
        log.info("[createToken] 토큰 생성 시작");

        Claims claims = Jwts.claims().setSubject(userUid);
        claims.put("roles", roles); // 토큰을 사용하는 사용자 권한 확인을 위해 입력
        Date now = new Date();

        // 토큰 생성
        String token = Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(now)
                .setExpiration(new Date(now.getTime() + tokenValidMillisecond))
                .signWith(key)
                .compact();

        log.info("[createToken] 토큰 생성 완료");
        return token;
    }


    // 필터에서 인증이 성공했을 때 SecurityContextHolder에 저장할 Authentication을 생성
    public Authentication getAuthentication(String token) {
        log.info("[getAuthentication] 토큰 인증 정보 조회 시작");

        UserDetails userDetails = service.loadUserByUsername(this.getUsername(token));

        log.info("[getAuthentication] 토큰 인증 정보 조회 완료, UserDetails username : {}", userDetails.getUsername());
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }


    public String getUsername(String token) {
        log.info("[getUsername] 토큰 기반 회원 구별 정보 추출");
        String info = Jwts
                .parserBuilder()
                .setSigningKey(key).build()
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
        log.info("[getUsername] 토큰 기반 회원 구별 정보 추출 완료, info : {}", info);
        return info;
    }

    public String resolveToken(HttpServletRequest request) {
        log.info("[resolveToken] HTTP 헤더에서 Token 값 추출");

        return request.getHeader("X-AUTH-TOKEN");
    }

    public boolean validateToken(String token) {
        log.info("[validateToken] 토큰 유효 체크 시작");

        try {
            Jws<Claims> claims = Jwts
                    .parserBuilder()
                    .setSigningKey(key).build()
                    .parseClaimsJws(token);

            return !claims.getBody().getExpiration().before(new Date());
        } catch (Exception e) {
            log.info("[validateToken] 토큰 유효 체크 예외 발생");

            return false;
        }
    }
}
```

토큰을 생성하기 위해서는 secretKey가 필요하므로 secretKey 값을 정의합니다.
@Value의 값은 application.properties 파일에서 다음과 같이 정의할 수 있습니다.

```
springboot.jwt.secret=flature!@#
```

```java
@PostConstruct
    protected void init() {
        LOGGER.info("[init] JwtTokenProvider 내 secretKey 초기화 시작");
        secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes(StandardCharsets.UTF_8));

        LOGGER.info("[info] JwtTokenProvider 내 secretKey 초기화 완료");
    }
```

여기서 사용한 @PostConstruct 어노테이션은 해당 객체가 빈 객체로 주입된 이후 수행되는 메서드를 가리킵니다.
JwtTokenProvider 클래스에는 @Component 어노테이션이 지정돼 있어 애플리케이션이 가동되면서 빈으로 가동 주입됩니다.
그때 @PostConstruct가 지정돼 있는 init() 메서드가 자동으로 실행됩니다.
init() 메서드에서는 secretKey를 Base64 형식으로 인코딩합니다. 인코딩 전후의 문자열을 확인하면 다음과 같습니다.

// Base64 인코딩 결과
ZmxhdHvyZSFAIW==

```java
public String createToken(String userUid, List<String> roles) {
        log.info("[createToken] 토큰 생성 시작");

        Claims claims = Jwts.claims().setSubject(userUid);
        claims.put("roles", roles); // 토큰을 사용하는 사용자 권한 확인을 위해 입력
        Date now = new Date();

        // 토큰 생성
        String token = Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(now)
                .setExpiration(new Date(now.getTime() + tokenValidMillisecond))
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();

        log.info("[createToken] 토큰 생성 완료");
        return token;
    }
```

JWT 토큰의 내용에 값을 넣기 위해 Claims 객체를 생성합니다.
setSubject() 메서드를 통해 sub 속성에 값을 추가하려면 User의 uid 값을 사용합니다.
해당 토큰을 사용하는 사용자의 권한을 확인할 수 있는 role 값을 별개로 추가했습니다.
Jwt.builer를 사용해 토큰을 생성합니다.

다음으로 볼 내용은 getAuthentication() 메서드입니다.

```java
  // 필터에서 인증이 성공했을 때 SecurityContextHolder에 저장할 Authentication을 생성
    public Authentication getAuthentication(String token) {
        log.info("[getAuthentication] 토큰 인증 정보 조회 시작");

        UserDetails userDetails = service.loadUserByUsername(this.getUsername(token));

        log.info("[getAuthentication] 토큰 인증 정보 조회 완료, UserDetails username : {}", userDetails.getUsername());
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }
```

이 메서드는 필터에서 인증이 성공했을 때 SecurityContextHolder에 저장할 Authenication을 생성하는 역할을 합니다.
Authenication을 구현하는 편한 방법은 UsernamePasswordAuthenticationToken을 사용하는 것입니다.
UsernamePasswordAuthenticationToken은 AbstractAuthenticationToken을 상속받고 있는데 AbstractAuthenticationToken은 Authentication 인터페이스의 구현체입니다.

이 토큰 클래스를 사용하려면 초기화를 위한 UserDetails가 필요합니다. 이 객체는 UserDetailsService를 통해 가져오게 됩니다.
이때 사용되는 Username값은 다음과 같이 구현합니다.


```java
public String getUsername(String token) {
        log.info("[getUsername] 토큰 기반 회원 구별 정보 추출");
        String info = Jwts
                .parserBuilder()
                .setSigningKey(secretKey).build()
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
        log.info("[getUsername] 토큰 기반 회원 구별 정보 추출 완료, info : {}", info);
        return info;
    }
```

Jwt.parser()를 통해 secretKey를 설정하고 클레임을 추출해서 토큰을 생성할 때 넣었던 sub 값을 추출합니다.
그다음으로 살펴볼 메서드는 resolveToken()입니다. resolveToken() 메서드는 다음과 같이 구현돼 있습니다.

```java
  public String resolveToken(HttpServletRequest request) {
        log.info("[resolveToken] HTTP 헤더에서 Token 값 추출");

        return request.getHeader("X-AUTH-TOKEN");
    }
```

이 메서드는 HttpServletRequest를 파라미터로 받아 헤더 값으로 전달된 'X-AUTH-TOKEN' 값을 가져와 리턴합니다.
클라이언트가 헤더를 통해 애플리케이션 서버로 JWT 토큰 값을 전달해야 정상적인 추출이 가능합니다.
헤더의 이름은 임의로 변경할 수 있습니다.

```java
public boolean validateToken(String token) {
        log.info("[validateToken] 토큰 유효 체크 시작");

        try {
            Jws<Claims> claims = Jwts
                    .parserBuilder()
                    .setSigningKey(secretKey).build()
                    .parseClaimsJws(token);

            return !claims.getBody().getExpiration().before(new Date());
        } catch (Exception e) {
            log.info("[validateToken] 토큰 유효 체크 예외 발생");

            return false;
        }
    }
```

이 메서드는 토큰을 전달받아 클레임의 유효기간을 체크하고 boolean 타입의 값을 리턴하는 역할을 합니다.

# JwtAuthenticationFilter 구현
JwtAuthenticationFilter는 JWT 토큰으로 인증하고 SecurityContextHolder에 추가하는 필터를 설정하는 클래스입니다.

```java
package com.springboot.security.config.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Slf4j
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtTokenProvider provider;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {

        String token = provider.resolveToken(request);
        log.info("[doFilterInternal] token 값 추출 완료. token : {}", token);

        log.info("[doFilterInternal] token 값 유효성 체크 시작");
        if (token != null && provider.validateToken(token)) {

            Authentication authentication = provider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(authentication);

            log.info("[doFilterInternal] token 값 유효성 체크 완료");
        }

        filterChain.doFilter(request, response);
    }
}
```

코드는 비교적 간단합니다.
먼저 살펴볼 부분은 1번 줄의 OncePerRequestFilter 입니다.
스프링 부트에서는 필터를 여러 방법으로 구현할 수 있는데, 가장 편한 구현 방법은 필터를 상속받아 사용하는 것 입니다.
대표적으로 많이 사용되는 상속 객체는 GenericFilterBean과 OncePerRequestFilter입니다.
GenericFilterBean을 상속받아 구현할수도 있습니다.

OncePerRequestFilter로 부터 오버라이딩한 doFilterInternal() 메서드가 있습니다.
doFilter() 메서드는 서블릿을 실행하는 메서드인데, doFilter() 메서드를 기준으로 앞에 작성한 코드는 서블릿이 실행되기 전에 실행되고,
뒤에 작성한 코드는 서블릿이 실행된 후에 실행됩니다.
메서드의 내부 로직을 보면 JwtTokenProvider를 통해 servletRequest에서 토큰을 추출하고, 토큰에 대한 유효성을 검사합니다.
토큰이 유효하다면 Authentication 객체를 생성해서 SecurityContextHolder에 추가하는 작업을 수행합니다.

```java
package com.springboot.security.config.security;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.security.web.AuthenticationEntryPoint;


@RequiredArgsConstructor
@Configuration
@EnableWebSecurity
public class SecurityConfiguration {

    private final JwtTokenProvider provider;
    private final AccessDeniedHandler accessDeniedHandler;
    private final AuthenticationEntryPoint authenticationEntryPoint;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .httpBasic(httpBasic -> httpBasic.disable())
                .csrf(csrf -> csrf.disable())
                .sessionManagement(sessionManagement -> sessionManagement
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                )
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers(
                                "/webjars/**",
                                "/v3/api-docs/**",
                                "/swagger-resources/**",
                                "/swagger-ui.html",
                                "/swagger-ui/**",
                                "/sign-api/sign-in",
                                "/sign-api/sign-up",
                                "/sign-api/exception",
                                "**exception**"
                        ).permitAll()
                        .anyRequest().hasRole("ADMIN")
                )
                .exceptionHandling(exceptionHandling -> exceptionHandling
                        .accessDeniedHandler(accessDeniedHandler)
                        .authenticationEntryPoint(authenticationEntryPoint)
                );

        // 추가된 필터 설정
        http.addFilterBefore(
                new JwtAuthenticationFilter(provider),
                UsernamePasswordAuthenticationFilter.class
        );

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

스프링 시큐리티의 설정은 대부분 HttpSecurity를 통해 진행합니다.
대표적인 기능은 다음과 같습니다.

- 리소스 접근 권한 설정
- 인증 실패 시 발생하는 예외 처리
- 인증 로직 커스터마이징
- csrf, cors등의 스프링 시큐리티 설정

각 메서드는 CustomAccessDeniedHandler와 CustomAuthenticationEntryPoint로 예외를 전달합니다.
스프링 시큐리티는 각각의 역할을 수행하는 필터들이 체인 형태로 구성돼 순서대로 동작합니다.
JWT로 인증하는 필터를 생성했으며, 이 필터의 등록은 HttpSecurity 설정에서 진행합니다.
addFilterBefore() 메서드를 사용해 어느 필터 앞에 추가할 것인지 설정할 수 있는데,
현재 구현돼 있는 설정은 스프링 시큐리티에서 인증을 처리하는 필터인 UsernamePasswordAuthenticationFilter 앞에 앞에서
생성한 JwtAuthenticationFilter는 자동으로 통과되기 때문에 위와 같은 구성을 선택했습니다.

WebSecurity를 사용하는 configure() 메서드입니다.
WebSecurity는 HttpSecurity 앞단에 적용되며, 전체적으로 스프링 시큐리티의 영향권 밖에 있습니다.
즉, 인증과 인가가 모두 적용되기 전에 동작하는 설정입니다.
그렇게 때문에 다양한 곳에서 사용되지 않고 인증과 인가가 적용되지 않는 리소스 접근에 대해서만 사용합니다.
Swagger에 적용되는 인증과 인가를 피하기 위해 ignoring()메서드를 사용해 Swagger와 관련된 경로에 대한 예외 처리를 수행합니다.
의미상 예외 처리라고 표현했지만 정확하게는 인증, 인가를 무시하는 경로를 설정한 것입니다.

# 커스텀 AccessDeniedHanlder, AuthenticationEntryPoint 구현
인증과 인가 과정의 예외 상황에서 CustomAccessDeniedHanlder와 CustomAuthenticationEntryPoint로 예외를 전달하고 있었습니다.
이번 절에서는 이러한 클래스를 작성하는 방법을 알아보겠습니다.
먼저 AccessDeniedHandler 인터페이스의 구현체 클래스를 생성하겠습니다.
다음과 같이 handle() 메서드를 오버라이딩해서 구현하게 됩니다.

```java
package com.springboot.security.config.security;

import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.web.access.AccessDeniedHandler;

import java.io.IOException;

public class CustomAccessDeniedHanlder implements AccessDeniedHandler {
    private final Logger LOGGER = LoggerFactory.getLogger(CustomAccessDeniedHanlder.class);

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, org.springframework.security.access.AccessDeniedException accessDeniedException) throws IOException, ServletException {
        LOGGER.info("[handle] 접근이 막혔을 경우 경로 리다이렉트");
        response.sendRedirect("/sign-api/exception");
    }
}
```

AccessDeniedHandler은 엑세스 권한이 없는 리소스에 접근할 경우 발생하는 예외입니다.
이 예외를 처리하기 위해 AccessDeniedHandler 인터페이스가 사용되며, SecurityCOnfiguration애도 exceptionHandling() 메서드를 통해 추가했습니다.
AccessDeniedHandler의  구현 클래스인 CustomAccessDeniedHandler 클래스는 handle() 메서드를 오버라이딩합니다.
이 메서드는 HttpServletRequest와 HttpServletResponse, AccessDeniedException을 파라미터로 가져옵니다.

이번 예제에서는 response에서 리다이렉트하는 sendRedirect() 메서드를 활용하는 방식으로 구현했습니다.
경로를 정의하면 다음과 같이 리다이렉트되어 정의한 예외 메서드가 호출되는 것을 볼수 있습니다.

로그에 출력된 스레드 번호를 보면 리다이렉트됐기 떄문에 다른 스레드에서 동작하는 것을 볼 수 있습니다.
다음은 인증이 실패한 상황을 처리하는 AuthenticationEntryPoint 인터페이스를 구현한 CustomAuthenticationEntryPoint 클래스입니다.

```java
package com.springboot.security.data.dto;

import com.fasterxml.jackson.databind.ObjectMapper;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Slf4j
@Component
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        ObjectMapper mapper = new ObjectMapper();
        log.info("[commence] 인증 실패로 response.sendError 발생");

        EntryPointErrorResponse entryPointErrorResponse = new EntryPointErrorResponse();
        entryPointErrorResponse.setMsg("인증이 실패하였습니다.");

        response.setStatus(401);
        response.setContentType("application/json");
        response.setCharacterEncoding("utf-8");
        response.getWriter().write(mapper.writeValueAsString(entryPointErrorResponse));
    }

}
```

```java
package com.springboot.security.data.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class EntryPointErrorResponse {

    private String msg;

}
```

클래스 구조는 앞에서 본 AccessDeniedHandler와 크게 다르지 않으며, commence() 메서드를 오버라이딩해서 코드를 구현합니다.
이 commence() 메서드는 HttpServletRequest, HttpServletResponse, AuthenticationException을 매개변수로 받는데,
이번 예제에서는 예외 처리를 위해 리다이렉트가 아니라 직접 Response를 생성해서 클라이언트에게 응답하는 방식으로 구현돼 있습니다.
컨트롤러에서는 응답을 위한 설정들이 자동으로 구현되기 때문에 별도의 작업이 필요하지 않았지만 여기서는 응답값을 설정할 필요가 있습니다.
메시지를 담기 위해 EntryPointErrorResponse 객체를 사용해 메시지를 설정하고, response에 상태 코드(status)와 콘텐츠 타입(Content-type) 등을 설정한 후 ObjectMapper를 사용해 EntryPointErrorResponse 객체를 바디 값으로 파싱합니다.

# 회원가입과 로그인 구현
인증에 사용되는 UserDetails 인터페이스의 구현체 클래스로 User 엔티티를 생성했습니다.
지금까지는 User 객체를 통해 인증하는 방법을 구현헀는데, 이번 절에서는 User 객체를 생성하기 위해 회원가입을 구현하고 User 객체로 인증을 시도하는 로그인을 구현하겠습니다.
회원가입과 로그인의 도메인은 Sign으로 통합해서 표현할 예정이며, 각각  Sign-up, Sign-in으로 구분해서 기능을 구현합니다.
먼저 서비스 레이어를 구현하겠습니다.
SignService 인터페이스에 정의된 메서드는 다음과 같습니다.

```java
package com.springboot.security.service;

import com.springboot.security.data.dto.SignInResultDto;
import com.springboot.security.data.dto.SignUpResultDto;

public interface SignService {

    SignUpResultDto signup(String id, String password, String name, String role);

    SignInResultDto signin(String id, String password) throws RuntimeException;
}
```

```java
package com.springboot.security.service.impl;

import com.springboot.security.config.security.JwtTokenProvider;
import com.springboot.security.data.dto.CommonResponse;
import com.springboot.security.data.dto.SignInResultDto;
import com.springboot.security.data.dto.SignUpResultDto;
import com.springboot.security.data.entity.User;
import com.springboot.security.data.repository.UserRepository;
import com.springboot.security.service.SignService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.Collections;

@Slf4j
@RequiredArgsConstructor
@Service
public class SignServiceImpl implements SignService {

    private final UserRepository userRepository;
    private final JwtTokenProvider jwtTokenProvider;
    private final PasswordEncoder passwordEncoder;

    @Override
    public SignUpResultDto signup(String id, String password, String name, String role) {
        log.info("[getSignUp Result] 회원가입 정보 전달 ");

        User user;
        if (role.equalsIgnoreCase("admin"))
            user = User.builder()
                    .uid(id)
                    .name(name)
                    .password(passwordEncoder.encode(password))
                    .roles(Collections.singletonList("ROLE_ADMIN"))
                    .build();
        else
            user = User.builder()
                    .uid(id)
                    .name(name)
                    .password(passwordEncoder.encode(password))
                    .roles(Collections.singletonList("ROLE_USER"))
                    .build();

        User savedUser = userRepository.save(user);
        SignUpResultDto signUpResultDto = new SignUpResultDto();

        log.info("[getSignUpResult] userEntity 값이 들어왔는지 확인 후 결과값 주입");
        if (!savedUser.getName().isEmpty()) {
            log.info("[getSignUpResult] 정상 처리 완료");
            setSuccessResult(signUpResultDto);
        } else {
            log.info("[getSignUpResult] 실패 처리 완료");
            setFailResult(signUpResultDto);
        }

        return signUpResultDto;
    }

    @Override
    public SignInResultDto signin(String id, String password) throws RuntimeException {
        log.info("[getSignInResult] signDataHandler 로 회원 정보 요청");

        User user = userRepository.getByUid(id);
        log.info("[getSignInResult] Id : {}", id);

        log.info("[getSignInResult] 패스워드 비교 수행");
        if (!passwordEncoder.matches(password, user.getPassword()))
            throw new RuntimeException();

        log.info("[getSignInResult] SignInResultDto 객체 생성");
        SignInResultDto signInResultDto = SignInResultDto.builder()
                .token(jwtTokenProvider.createToken(String.valueOf(user.getUid()), user.getRoles()))
                .build();

        log.info("[getSignInResult] SignInResultDto 객체에 값 주입");
        setSuccessResult(signInResultDto);

        return signInResultDto;
    }

    private void setSuccessResult(SignUpResultDto result) {
        result.setSuccess(true);
        result.setCode(CommonResponse.SUCCESS.getCode());
        result.setMsg(CommonResponse.SUCCESS.getMsg());
    }

    private void setFailResult(SignUpResultDto result) {
        result.setSuccess(false);
        result.setCode(CommonResponse.FAIL.getCode());
        result.setMsg(CommonResponse.FAIL.getMsg());
    }

}
```

회원가입과 로그인을 구현하기 위해 세 가지 객체에 대한 의존성 주입을 받습니다.
회원가입을 구현하고, 현재 애플리케이션에서는 ADMIN과 USER로 권한을 구분하고 있습니다.
SignUp()메서드는 그에 맞게 전달받은 role 객체를 확인해 User 엔티티의 roles 변수에 추가해서 엔티티를 생성합니다.
패스워드는 암호화해서 저장해야 하기 때문에 PasswordEncoder를 활용해 인코딩을 수행합니다.
PasswordEncoder는 다음과 같이 별도의 @Configuration 클래스를 생성하고 @Bean 객체로 등록하도록 구현했습니다.

```java
package com.springboot.security.config;

import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

public class PasswordEncoderConfiguration {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

위 예제는 빈 객체를 등록하기 위해서 생성된 클래스이기 때문에 SecurityConfiguration 클래스 같은 이미 생성된 @Configuration 클래스 내부에
passwordEncoder() 메서드를 정의해도 충분합니다.
이렇게 생성된 엔티티를 UserRepository를 통해 저장합니다.
실제 엔터프라이즈 환경에서는 회원가입을 위한 필드도 많고 코드도 복잡하겠지만 이 책에서는 부가적인 사항들은 모두 배제하고 회원가입 자체만 구현하겠습니다.
이제 회원으로 가입한 사용자의 아이디와 패스어드를 가지고 로그인을 수행할 수 있습니다.
로그인 메서드는 구현돼 있습니다.
로그인은 미리 저장돼 있는 계정 정보와 요청을 통해 전달된 계정 정보와 일치하는지 확인하는 작업입니다.
signIn()메서드는 아이디와 패스워드를 입력받아 처리하게 됩니다.
내루 로직을 좀 더 자세리 살펴보겠습니다.

id를 기반으로 UserRepository에서 User 엔티티를 가져옵니다
PasswordEncoder를 시용해 데이터베이스에 저장돼 있던 패스워드와 입력받은 패스워드가 일치하는지 확인하는 작업을 수행합니다.
이번 예제에서는 패스워드가 일치하지 않아 예외를 발생시키는데 RuntimeException을 사용했지만 별도의 커스텀 예외를 만들어서 사용하기도 합니다.
패스워드가 일치해서 인증을 통과하면 JwtTokenProvider를 통해 id와 role값을 전달해서 생성한 후 Response에 담아 전달합니다.
결과 데이터를 설정하는 메서드입니다.
회원가입과 로그인 메서드에서 사용할 수 있게 설정돼 있으며, 각 메서드는 DTO를 전달받아 값을 설정합니다.
이떄 사용된 CommomResponse 클래스는 다음과 같이 작성돼 있습니다.

```java
package com.springboot.security.data.dto;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum CommonResponse {

    SUCCESS(0, "Success"), FAIL(-1, "Fail");

    private final int code;
    private final String msg;
}
```

이제 회원 가입과 로그인을 API로 노출하는 컨트롤러를 생성해야 하는데 사실상 서비스 레이어로 요청을 전달하고 응답하는 역할만
수행하기 때문에 코드만 소개하겠습니다.

```java
package com.springboot.security.data.controller;

import com.springboot.security.data.dto.SignInResultDto;
import com.springboot.security.data.dto.SignUpResultDto;
import com.springboot.security.service.SignService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@Slf4j
@RequiredArgsConstructor
@RestController
@RequestMapping("/sign-api")
public class SignController {

    private final SignService service;

    @PostMapping("/sign-in")
    public SignInResultDto signIn(
            @RequestParam String id,
            @RequestParam String password
    ) throws RuntimeException {
        log.info("[signIn] 로그인을 시도하고 있습니다. id : {} , pw : ****", id);
        SignInResultDto dto = service.signin(id, password);

        if (dto.getCode() == 0)
            log.info("[signIn] 정상적으로 로그인 되었습니다. id : {} , token : {}", id, dto.getToken());

        return dto;
    }

    @PostMapping("/sign-up")
    public SignUpResultDto signUp(
            @RequestParam String id,
            @RequestParam String password,
            @RequestParam String name,
            @RequestParam String role
    ) {
        log.info(
                "[signUp] 회원가입을 수행합니다. id: {} , password: **** , name : {} , role: {}",
                id, name, role);

        SignUpResultDto dto = service.signup(id, password, name, role);

        log.info("[signUp] 회원가입을 완료했습니다. id : {}", id);

        return dto;
    }

    @GetMapping("/exception")
    public void exceptionTest() throws RuntimeException {
        throw new RuntimeException("접근이 금지되었습니다.");
    }

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<Map<String, String>> ExceptionHandler(RuntimeException e) {
        HttpHeaders responseHeaders = new HttpHeaders();

        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;

        log.error("ExceptionHandler 호출, {}, {}", e.getCause(), e.getMessage());

        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", "에러 발생");

        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }
}
```

클라이언트는 위와 같이 계정을 생성하고 로그인 과정을 거쳐 토큰값을 전달받음으로써 이 애플리케이션에서 제공하는 API 서비스를 사용할 준비를 마칩니다.
Response로 전달되는 SignUpResultDto와 SignInResultDto 클래스는 다음가 같습니다.

```java
package com.springboot.security.data.dto;

import lombok.*;

@Data
@EqualsAndHashCode(callSuper = true)
@NoArgsConstructor
@AllArgsConstructor
@ToString(callSuper = true)
public class SignInResultDto extends SignUpResultDto{

    private String token;

    @Builder
    public SignInResultDto(boolean success, int code, String msg, String token) {
        super(success, code, msg);
        this.token = token;
    }
}
```

```java
package com.springboot.security.data.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class SignUpResultDto {

    private boolean success;
    private int code;
    private String msg;

}
```

여기까지 구현이 완료되면 정상적으로 스프링 시큐리티가 동작하는 애플리케이션 환경이 완성된 것입니다.

